---
layout: post
title: "cloudflare dns"
date:   2018-04-02 19:00:00 +0100
last-modified: 2018-04-02 19:00:00 +0100
categories: networking
image: 
  name:   "networkswitch.jpg"
  global: "local"
---
Yesterday [cloudflare](https://www.cloudflare.com/) [announced](https://blog.cloudflare.com/announcing-1111/) their new privacy-first consumer [DNS service](https://1.1.1.1/). Announcing it on April fools day was a bit suspicious but it made me re-evaluate my own home dns situation. 

Before today I used [dnscrypt-proxy](https://github.com/jedisct1/dnscrypt-proxy) (the previous version) along with the [Unbound](https://www.unbound.net/) DNS resolver. Both running in a [FreeBSD](https://www.freebsd.org/) jail on my [FreeNAS](http://www.freenas.org/) based [NAS](https://en.wikipedia.org/wiki/Network-attached_storage). This meant that all computers on my own network would get their dns servers from my router, which would point to my local DNS setup, which in turn encrypted all dns queries.
This allowed me to keep my dns queries private from my ISP and allowed me to easily ignore their court ordered [censorship](https://en.wikipedia.org/wiki/Countries_blocking_access_to_The_Pirate_Bay).

However since I was using the older version of [dnscrypt](https://dnscrypt.info/) it seemed the amount of available proxies to use was dwindling and they were getting very very slow (5+ seconds sometimes!). So obviously it was time for a new solution. 

Since I initially set this up, [dnscrypt-proxy](https://github.com/jedisct1/dnscrypt-proxy) has released a version 2. Which not only supports the dnscrypt protocol, but also [DNS-Over-HTTPs (DoH)](https://en.wikipedia.org/wiki/DNS_over_HTTPS). And since the [Cloudflare DNS service](https://1.1.1.1/) supports DoH it seemed an ideal combination to get supper fast DNS resolution, stay uncensored AND keep my privacy.  
An alternative client would be [facebooks doh-proxy](https://facebookexperimental.github.io/doh-proxy/) tools. But although they release some nice opensource tools, I'd rather not use anything made by facebook if I can help it. 
We'll find out in the future if cloudflare is as trustworthy as the volunteer dnscrypt proxies I used before, but they are definitely going to be a lot faster.

### My setup

As suggested above, I'm going to stick with [dnscrypt-proxy](https://github.com/jedisct1/dnscrypt-proxy), just upgrade to the newer version and use cloudflare as the target server. I'll also keep [Unbound](https://www.unbound.net/) in front of it for caching and changing some dns results to local ip's (though the new dnscrypt-proxy could probably handle this part too).

The slight problem with this is that the new dnscrypt-proxy doesn't have a freebsd package or port. Though they do provide binaries for FreeBSD, its install command doesn't work ([due to an upstream package](https://github.com/kardianos/service/issues/98)) and has to be done manually (looking a bit at their [pfsense installation instructions](https://github.com/jedisct1/dnscrypt-proxy/wiki/Installation-pfsense) for inspiration). 

First I installed dnscrypt using the following series of commands: 
{% highlight shell %}
cd /tmp
fetch -m https://github.com/jedisct1/dnscrypt-proxy/releases/download/2.0.8/dnscrypt-proxy-freebsd_amd64-2.0.8.tar.gz
mkdir dnscrypt-proxy-freebsd_amd64-2.0.8
tar -zxf dnscrypt-proxy-freebsd_amd64-2.0.8.tar.gz -C dnscrypt-proxy-freebsd_amd64-2.0.8
mv dnscrypt-proxy-freebsd_amd64-2.0.8/freebsd-amd64/dnscrypt-proxy /usr/local/bin/dnscrypt-proxy
chown root:wheel /usr/local/bin/dnscrypt-proxy
chmod +x /usr/local/bin/dnscrypt-proxy
mkdir /usr/local/etc/dnscrypt-proxy/
cp dnscrypt-proxy-freebsd_amd64-2.0.8/freebsd-amd64/example-dnscrypt-proxy.toml /usr/local/etc/dnscrypt-proxy/dnscrypt-proxy.toml
{% endhighlight %}

I edited ```dnscrypt-proxy.toml``` to use cloudflare, and set it to listen on port 6000 (Unbound is already listening on the normal dns port).

Next up, to make it actually auto start and be a properly managed service, added the following line to my ```/etc/rc.conf```: ```dnscrypt_proxy_enable="YES"``` (actually, in my case this was already there from the previous installed version which did have a FreeBSD package)

And lastly it needed a proper rc.d script, so I created the file ```/usr/local/etc/rc.d/dnscryptproxy``` with the following content:
{% highlight shell %}
{% raw %}
#!/bin/sh
#
# PROVIDE: dnscryptproxy
# REQUIRE: NETWORKING

. /etc/rc.subr

name="dnscryptproxy"
rcvar="dnscrypt_proxy_enable"
dnscrypt_command="/usr/local/bin/dnscrypt-proxy"
dnscrypt_config="/usr/local/etc/dnscrypt-proxy/dnscrypt-proxy.toml"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f ${dnscrypt_command} -config ${dnscrypt_config}"

load_rc_config $name
: ${dnscrypt_proxy_enable:=no}

run_rc_command "$1"
{% endraw %}
{% endhighlight %}

Then all that was left to do was service dnscryptproxy start
