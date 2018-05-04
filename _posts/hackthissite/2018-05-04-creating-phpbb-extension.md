---
layout: post
title:  "creating phpbb extension"
date:   2018-05-04 14:00:00 +0200
last-modified: 2018-05-04 15:00:00 +0200
categories: hackthissite phpbb sso
featured: false
head-image: 
  name: false
  file:   "phpbbcode.png"
  description: "phpBB code"
  global: local
  creator: weekend
  
---
As you may know I'm an administrator of [hackthissite.org](https://www.hackthissite.org) (also referred to as HTS). The current ongoing project at HTS is moving the main site over to a new server. The current code base is pretty ancient and hacked together so this is also a chance to improve on that and make it easier to update in the future.

The major effort to change HTS is splitting the project into separate components so they can be reworked individually. And the first step in that process is creating a Single Sign On (SSO) system so users can authenticate in one place and be logged in on all components.  
This post is about updating and revamping the forum SSO.

Like the rest of the code, the current forums are also old and will need updating for it to work on a new system. To do that I'll be starting from a fresh [phpBB](https://www.phpbb.com) installation and creating a new authentication extension for it.  
This will be in place of the old hacky method of providing SSO for the forums.

## Setting up development environment

Our dev environment will be a vagrant box where we'll install all the things. Since phpbb doesn't seem to be very happy of living in a crappy misconfigured dev environment I created a [vagrant config](https://github.com/wknd/vagrant-php) that gets it to a minimal working state.  
In your main project folder will be a directory holding the phpbb code, one for our extension, and one for the vagrant files.

#### Vagrant
Tweak the ```config.yaml``` file as followed:
{%- highlight yaml -%}{% raw %}
---
ip: "192.168.7.7"
memory: 1024
cpus: 2

networking:
    - public: true

folders:
    - map: ../phpbb
      to: /var/www/html
      owner: www-data
      group: www-data
    - map: ../phpBB3-Authenticator-Ostiary
      to: /var/www/html/ext/hackthissite/ostiary
      owner: www-data
      group: www-data

nginx: "phpbb"
{% endraw %}{%- endhighlight -%}
and run the following from inside the vagrant folder:
```
vagrant up
```
This should create a vm with php7.2, mysql and nginx installed and configured for phpbb. It'll set up shared folders to place our previously created directories in the proper place.  
Browse to the ip specified in the config and setup phpbb.

## Plan of attack
Our initial goal is simple; see if we can get a proof of concept phpbb authentication extension working. We're not working on the HTS part of it so to we are going to use a totally insecure and horrible method to see if it works: we're going to take a cookie and pretend that if it is set to a username that user is in fact logged in.

The plugin contains two parts, a listener and an authentication service.

#### Authentication service
The [documentation](https://wiki.phpbb.com/Authentication_plugins) on this is pretty sparse, as is any other documentation on phpBB. But we get enough to [get started](https://area51.phpbb.com/docs/dev/extensions/tutorial_authentication.html).  
We will implement the ```login```, ```autologin``` and ```validate_session``` methods (and logout soon too). 

We're actually not very interested in login, since we want users to automatically login if they are logged in on our SSO system without having to enter a username or password.  
Of course phpBB isn't really built with that in mind even though most people who want to create an authentication plugin seem to have this goal in mind (before they give up). But we'll get around that limitation later.  
The login form also gets used to verify your identity again if you go to the admin page. Currently it totally ignores what you input in there and simple uses the SSO to see who you are, but I plan to make it ask for a 2 factor authentication token on admin in the future. phpBB doesn't allow you to change the login form though, only add more data to it, so the 2FA form should probably use some fancy javascript to rewrite the form.

First important bit is the autologin method, this gets run whenever a user autologins. So here we will verify that the user is signed in on our SSO platform, and return the relevant row for that user in the db. If the user isn't yet created on the forum, we first create it and then return the database row.

One problem with this is that despite the name and description (it gets run if a user creates a new session), it's values are ignored unless there is another cookie already set which enables autologin. This cookie would normally be set when a user logs in and checks the "remember me" checkbox. So if a user is logged in to our SSO we set that cookie before returning with:
{% highlight php %}{% raw %}
$this->user->set_login_key($row['user_id'], false, false);
{% endraw %}{% endhighlight %}
This won't log in the user, but at least will set things up to log him in the next time he creates a new session (which we do with a listener as soon as he goes to the login page).

The ```validate_session``` method is used to see if a user is still logged in. As a great superhero's [dead uncle](https://en.wikipedia.org/wiki/Uncle_Ben) once said "with single sign-on, also comes single sign-**off**". Or at least I think it went like that.  
So we again verify the user is still logged on with us and if it isn't we use the normal php methods (check if its an anon user or a bot and if it isn't we invalidate the session).

#### Listener 
So far the user still isn't logged in, which was what we set out to do in the first place. But because previously we have set the autologin cookie we will not ensure the user actually logs in as soon as he clicks the login button. We do this by adding a [listener](https://area51.phpbb.com/docs/dev/extensions/tutorial_events.html) that gets triggered whenever the [login box gets generated](https://wiki.phpbb.com/Event_List). The code we run on that event is pretty simple:
{% highlight php %}{% raw %}
if (!$event['admin'])
{ // its not the admin page

  $row = $this->ostiary->autologin();
  
  if (sizeof($row) && !empty($row['user_id']) && is_numeric($row['user_id']) && $row['user_id'] != 0)
  { //the user was totally legit and what not
    $this->user->session_create($row['user_id'], false, true, true);
    redirect($phpbb_root_path); 
  } else {
    redirect("https://google.com", false, true);
  }
}
{% endraw %}{% endhighlight %}
If the generated login box wasn't for the admin page, we run our ```autologin``` function again, which allows us to verify the user is signed in on SSO, and if it is we create a new [session](https://github.com/phpbb/phpbb/blob/master/phpBB/phpbb/session.php) for it (with ```autologin = true```) and redirect him to the main forum page. By creating a new session, ```autologin``` will get run again and the user will be logged in this time. So when the user reaches the main page it'll be in a logged in state.  
If the user wasn't logged in with us at the time he clicked the login button, we will redirect him to our SSO page (or for now, to google).

This part obviously could use some improving, because we are effectively running ```autologin``` twice.  
We could probably run it just once and only create the session but then we wouldn't know in advance if we should be redirecting the user to our SSO page.  
As I'm writing this I figure maybe we can just create the session in all cases and then listen for an event at the end of session creation (and thus after autologin) and redirect at the end of that (taking internal phpBB status of logged in or not to determine where to redirect to).

#### Summary

*    If a user isn't signed on SSO and goes to the forum, he'll get a new forum session. When he clicks login the user will get redirected to our SSO page.
*    If a user is signed on SSO and goes to the forum, he'll get a new forum session. When he clicks login the user will get logged in (and if he doesn't exist yet on the forum, his account will get created).
*    When a user signs off on the forum, he currently only signs off on the forum and we don't feed this back to the SSO system (maybe we optionally will?)
*    When an admin goes to the admin panel, he will get a login screen to verify his identiy. We ignore whatever he inputs there and log him in based on SSO (future may ask for 2FA?).

## Improvements
There are three main points that need improving:

1.    We are polling our backend very frequently, effectively every page load (sometimes even twice!). I think this should go down, though Kage likes it this way. The backend will communicate through a locally replicated redis instance and can probably handle it for our needs even if we put each service in a totally different datacenter. But I still don't like communicating more frequently than required. 

2.    The actual login form currently does nothing, it'll just ask the backend if the user is valid. I want to make it ask for 2FA data for admins (and maybe optionally for users who want 2FA for each individual service?).

3.    There is no backend. Right now it just looks for the contents of a certain cookie and assumes its the username. It doesn't actually ask the backend yet (but it will soon).
