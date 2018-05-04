---
layout: post
title:  "blog updates"
date:   2018-05-04 19:00:00 +0200
categories: blog site

---
The blog got some more updates. I got to the point where I had enough posts that I needed [pagination](https://jekyllrb.com/docs/pagination/) on the main index. And a serious bug got fixed in the image minify script making images seem much more clear.

## Pagination
This part was very straight forward. I added the proper items in the sites config, renamed my ```index.md``` to ```index.html``` and copied over the ```home.html``` layout to ```pagination.html```.  
Then all I had to do was change references from ```site.posts``` to ```paginator.posts``` and add some links to the different pages.

I also had to add some css to make it look nice, that part took a few more attempts and will probably get even more tweaks in the future.

## Minimize bug
I didn't notice it before but my minimize script was poorly cropping the images. They were in some cases cropped to the wrong size. This meant that the browser would fetch an image that was actually too small and blow it up to fit on the page.

I noticed it when I took a picture from [this reddit post](https://www.reddit.com/r/urbanexploration/comments/8f2wu7/so_much_waste_in_such_little_space/). It had a nice high resolution and was very clear, so it was perfect for resizing and cropping. But the end result was still fuzzy.  
Before I realized what was happening I thought github or cloudflare were messing up my images, but that wasn't the case (I looked at the wrong image when comparing when I came to that wrong conclusion).

When I found the cause I set about fixing it and redid how my image decides to resize by using proper math instead of a very naive estimation.  
While implementing that I also found another weird bug. For some reason when an image was previously symlinked to another it would take the size of the symlink (just a few bytes) instead of the newly generated image to compare if it needed more resizing. This made it also symlink all smaller images (since no new size of any image could beat those few bytes). 

It was some weird race condition where ```stat -c%s "$FILE"``` would still be looking at the symlink, even though that should have been overwritten by a real file. The solution was to first rm the symlink and then write the file (just a sync wasn't good enough).

Special thanks is in order to reddit user [DetroitEXP](https://www.reddit.com/user/DetroitEXP), without his picture it would have been a lot longer before I found this bug. I sent him a message to thank him and made sure the image points I used now points to the desired place. You can check out his instagram [here](https://www.instagram.com/tonybologna_exp/) for more urban exploration images.

# Future
I've found that writing down the things I do help me wrap my head around it. And if I write the post with plenty of references that means I can close a million tabs in my browser when I'm done instead of waiting until the whole project is finished. So I think I'll keep doing this.

Current projects I'm writing about are flashing android for nethunter (just updated my lineage OS to see how it reacts to the current setup and my HID patch got [merged upstream](https://github.com/pelya/android-keyboard-gadget/commit/43a1bdffc4054fe050bdc4c0a1f0244b4001ecdc)), creating a shared USB switch (when is [KiCAD](http://kicad-pcb.org/) 5 finally getting here?), the blog itself and maybe also some publically available stuff for [HackThisSite.org](https://www.hackthissite.org).
