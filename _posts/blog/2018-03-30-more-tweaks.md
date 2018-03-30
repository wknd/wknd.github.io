---
layout: post
title:  "more tweaks"
date:   2018-03-30 21:00:00 +0100
categories: blog site
image: 
  name:   "hacker-room.jpg"
  global: true
---
I made several tweaks to the site. Some improvements and some made everything worse.

### images
My image minify [shell script](https://github.com/wknd/site-resources/blob/master/minifyimages.sh) got some improvements. It now not only minimizes images, but also generates images at different resolutions from the default based on its target width(320 480 800 1600).

I replace the last ```.``` in the filename with ```-width.``` to be able to use them in the markup. The [liquid](https://shopify.github.io/liquid/) template engine that jekyll uses has a [replace](https://shopify.github.io/liquid/filters/replace/) and a  [replace_first](https://shopify.github.io/liquid/filters/replace_first/) filter. But to ensure it only replaced the last ```.``` I needed a ```replace_last``` filter. There's also a [asset_img_url](https://help.shopify.com/themes/liquid/filters/url-filters#asset_img_url) that can take a width as input, but that probably relies on some other internal mechanism shopify uses to actually pre-process the images and didn't work with jekyll.

To get around this I used the following method:
{% highlight liquid %}{% raw %}
{%- assign imgReversed = imagePath | split: "" | reverse | join: "" -%}
{%- assign size320 = "-320." | split: "" | reverse | join: "" -%}
{%- assign size480 = "-480." | split: "" | reverse | join: "" -%}
{%- assign size800 = "-800." | split: "" | reverse | join: "" -%}
{%- assign size1600 = "-1600." | split: "" | reverse | join: "" -%}
{%- assign image320 = imgReversed | replace_first: '.', size320 | split: "" | reverse | join: "" | prepend: '/assets/images/' | prepend: baseimgPath -%}
{%- assign image480 = imgReversed | replace_first: '.', size480 | split: "" | reverse | join: "" | prepend: '/assets/images/' | prepend: baseimgPath -%}
{%- assign image800 = imgReversed | replace_first: '.', size800 | split: "" | reverse | join: "" | prepend: '/assets/images/' | prepend: baseimgPath -%}
{%- assign image1600 = imgReversed | replace_first: '.', size1600 | split: "" | reverse | join: "" | prepend: '/assets/images/' | prepend: baseimgPath -%}}}
{% endraw %}{% endhighlight %}
So I first reverse the image location, then replace the first ```.``` reverse it back, and add some path stuff.
There isn't a reverse filter for strings though, only for arrays, so I split the string, reverse it and then join it back together.

To then make use of these images I use the ```srcset``` and ```sizes``` attribute of an ```img``` tag.
{% highlight html %}
<img src="/assets/images/global/hacker-room.jpg" srcset="
       /assets/images/global/hacker-room-320.jpg 320w,
       /assets/images/global/hacker-room-480.jpg 480w,
       /assets/images/global/hacker-room-800.jpg 800w,
       /assets/images/global/hacker-room-1600.jpg 1600w"
       sizes="
       (min-width: 1380px) 33vw,
       (min-width: 900px) 50vw,
       100vw" itemprop="image" alt="">
{% endhighlight %}
By using ```srcset``` the browser will fetch a different image based on how wide the screen is. However this actually made my [google pagespeed](https://developers.google.com/speed/pagespeed/insights/?url=secur.ity-pro.be) score lower because it realized that on a full size laptop, it was fetching some images that were only being displayed on 1/3rd of the screen.  
That is fixed by also specifying the ```sizes``` so it knows the image will be a certain width when picking which image to download.

This improved my pagespeed score and made downloads even smaller. However because I crop my images in to fit a nice grid, it is grabbing images that are the correct width and then cropping and stretching them, making them look blurry and plain ugly.

I'll have to rethink my approach and will probably end up pre-cropping the images in certain ways. 

### schema
I also added a bunch of [schema.org](http://schema.org) information and tested it using google's [provided tool](https://search.google.com/structured-data/testing-tool#).

Google doesn't obey the schema.org set specifications at all times(for instance publisher can't be a person, but has to be an organization), but its not too bad to work around. 
Google also wants you to create your indexpages in [certain ways](https://developers.google.com/search/docs/guides/mark-up-listings) (no BlogPosting elements on the index, only for the actual post).

### logo
To have a correct schema, I needed a pubslisher. Publischers have to be organizations and in turn organizations need a logo. 
So to make google truely happy, I created a logo. The logo follows their [requirements](https://developers.google.com/search/docs/data-types/article#logo-guidelines).
