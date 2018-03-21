---
layout: post
title:  "finished basic layout"
date:   2018-03-21 21:00:00 +0100
categories: blog site
image: 
  name:   "hacker-room.jpg"
  global: true
---
I've finished the basic layout of the blog and it is now ready for accepting posts. I set out a bunch of goals for it:
1. Cyberpunk vibe
2. Scale across devices
3. Fast loading
4. Cheap/Free hosting
5. https
Each of these had some challenges which I'll get into in the following sections.

## look and feel
Since the blog is meant for technical projects and writeups I wanted to go with a nice clear cyberpunk vibe. 

What makes a layout clearly a cyberpunk layout? Some even argue that you can't make a cyberpunk website without all sorts of VR support, but I think thats going a bit too far. Even in any imaginary cyberpunk world there will need to be simple text displaying devices, and what is better for that than just a flat screen with some text? 

But then what makes the site layout cyberpunk and not just another dark theme, or neon theme. I tried finding examples of existing cyberpunk themed websites and came up short.
The best (and only clearly cyberpunk) design I found was on [neondystopia](https://www.neondystopia.com/).

It made me conclude that the difference is to not just use some dark theme with some flashy highlights, but to incorporate images into the website that are cyberpunk themed themselves. 
So my first decision was to make sure my layout played nice with images, and force myself to use an image for every single post.

Second I needed to decide on the colours I'd use. I'm definitely no graphic designer, my last few personal sites were made out of ascii art. As inspiration I thought I'd look at the colorschemes used for syntax highlighting. Most of those are picked to provide enough contrast between the colours and usually come in a light and a dark variant. I picked the [base 16](https://github.com/chriskempson/base16) [summerfruit](https://github.com/cscorley/base16-summerfruit-scheme) theme as my starting point. In the end I only used 1 of the bright colours for extra flash while using its foreground and background colours for everything else.

For the actual layout I took some more inspiration from [neondystopia](https://www.neondystopia.com/). I really liked the idea of having a grid layout, with the newest post being extra large, and a large featured post. Since [css grid](https://www.mozilla.org/en-US/developer/css-grid/) is now available for all major browsers, I decided to use that which made the whole process pretty easy.

## scaling
{% include articleimage.html name="main-page-large.png" global=false %}
The use of [css grid](https://www.mozilla.org/en-US/developer/css-grid/) and some [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) made it easy to have everything scale. I start off with a layout designed for small screens, and then work my way up to tablet size, and then laptop size. 

{% include articleimage.html name="main-page-smallest.png" global=false float="right" width="40%" %}
{% highlight css %}
.post-list {
    list-style: none;
    display: grid;

    grid-template-columns: 1fr;
    grid-gap: ($spacing-unit / 2) ($spacing-unit / 2);
    @include on-tablet {
        grid-template-columns: 1fr 1fr;
        grid-gap: $spacing-unit $spacing-unit;

    }
    @include on-laptop {
        grid-template-columns: 1fr 1fr 1fr;
        grid-gap: $spacing-unit $spacing-unit;

    }
    grid-auto-flow: row;
    justify-items: stretch;
    justify-content: stretch;
}
{% endhighlight %}



Some sub title
=========
Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit, sed quia non numquam eius modi tempora incidunt ut labore et dolore magnam aliquam quaerat voluptatem. Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur?

another smaller title
-------------
more text  
lots and lots of text

and paragraphs

# different title
## even more different title
### how far can we go
#### I already know
##### but just to be clear
###### I had to go so far

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing

different style



> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
id sem consectetuer libero luctus adipiscing.

and nested
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level


> ## This is a header.
> 
> 1.   This is the first list item.
> 2.   This is the second list item.
> 
> Here's some example code:
> 
>     return shell_exec("echo $input | $markdown_script");


{% highlight php %}
return shell_exec("echo $input | $markdown_script");
{% endhighlight %}

*   Red
*   Green
*   Blue

1. red
2. green

![image test](/assets/images/global/cross-street_francesco.lorenzetti.jpg)
this is an image on top of a paragraph  
the image will be within the paragraph part and is using standard markdown style


here the image is included using articleimage.html
{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true %}

same here but we specify a height
{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true height="400px" %}


maybe width is better?
{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true width="400px" %}


this works nicer, wonder if we can make images float nicely so they don't take up so much space and what not

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.


{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true height="20em" float="right" %}

floaty image  
lets include lots of text  
texty text text text
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

so much of the text
so long text


moar text




{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true float="right" %}

floaty image  
lets include lots of text  
texty text text text
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

so much of the text
looooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo ooooooooooooooooooooooooooooooooooooooooooooooooooooooooo ooooooooooooooooong text
so long text

{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true width="440px" float="right" %}

floaty image  
lets include lots of text  
texty text text text
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

looooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo ooooooooooooooooooooooooooooooooooooooooooooooooooooooooo ooooooooooooooooong text

{% include articleimage.html name="nfHwYkh.jpg" float="left" %}
stuffs
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true float="left" width="200px" %}

stuffs with width set
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true float="left" height="200px" %}
stuffs with height set
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true float="left" width="30%" %}
stuffs with width as percentage
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{% include articleimage.html name="cross-street_francesco.lorenzetti.jpg" global=true float="left" width="30%" height="600px" %}
stuffs with width as percentage and a height set
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.


---
