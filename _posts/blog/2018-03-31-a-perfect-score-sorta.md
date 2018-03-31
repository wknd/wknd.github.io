---
layout: post
title:  "a perfect score, sorta"
date:   2018-03-31 19:00:00 +0100
categories: blog site
image: 
  name:   "hacker-room.jpg"
  global: true
---
I added some functionality to my image minify [script](https://github.com/wknd/site-resources/blob/master/minifyimages.sh) so that it now also generates resized images that are cropped to fit in titles or index pages. This means the image selected will be more likely to match correctly. 

Additionally, which I forgot to mention last post, I have implemented the late loading of css.
{% highlight html %}
<noscript id="deferred-styles">
 <link rel="stylesheet" type="text/css" href="{% raw %}{{ sheet }}{% endraw %}"/>
</noscript>
<script>
 var loadDeferredStyles = function() {
   var addStylesNode = document.getElementById("deferred-styles");
   var replacement = document.createElement("div");
   replacement.innerHTML = addStylesNode.textContent;
   document.body.appendChild(replacement)
   addStylesNode.parentElement.removeChild(addStylesNode);
 };
 var raf = window.requestAnimationFrame || window.mozRequestAnimationFrame ||
     window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;
 if (raf) raf(function() { window.setTimeout(loadDeferredStyles, 0); });
 else window.addEventListener('load', loadDeferredStyles);
</script>
{% endhighlight %}

I've also implemented the same responsive image strategy for images inside posts. It isn't totally automatic, for images that have a height set you should also pass along a width for best results.

Together these changes give me (almost) a perfect score on [google pagespeed](https://developers.google.com/speed/pagespeed/insights/?url=secur.ity-pro.be).  
The only thing it complains about is further compressing one image ```Compressing https://secur.ity-pro.be/…shed-basic-layout/cloudflare-ttl-480.png could save 4.2KiB (14% reduction).```. But that image at that size is somewhat of an anomaly. It is actually bigger than some images at a larger size and no matter what I do I can't seem to fix that.

Perhaps I should make the script start converting the largest sizes and if the next size down is bigger in filesize, just use the previous size image for it instead. Or maybe I should just accept that sometimes images are weird and I can live with 1 point less on a single page. (though noted, if I didn't specify width on a bunch of images on that page, I would have only gotten a score of 24 even though those images were smaller than not making them responsive at all.)

I think the next problem to tackle is giving proper image attribution. You may notice how the last 3 posts all have the same image. I'm finding images I want to use, but they require to mention the creators and I don't have a system in place to do that yet. (for the background image, I've gotten word from its creator that he allows me to use it, so yay for that)
