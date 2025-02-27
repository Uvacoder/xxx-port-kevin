---
categories:
- coding
- frontend
date: 2013-08-15T00:00:00Z
title: px vs em

aliases:
  - /px-vs-em/
---

Recently [Tim Pietrusky](http://timpietrusky.com) wrote and article about [px vs em](http://timpietrusky.com/i-love-r-emmmmmm-because-px-suck) that contains a few links on resources that explain why you should use em instead of px as measure unit on websites.

I was using pixel like forever and I couldn't think of any reason to switch to a new unit until Tim explained why em is better: *em is more responsive and it is scalable*.
In fact when you switch from pixel to em you'll feel like working on a completely new Level. The fact you can scale all elements of your website by simply passing a new value to `:root {font-size:1em;}` or `body {font-size:1em;}`.

My `_responsive.scss` works this way. Because I think it's good to have large texts on smartphones I applied `font-size:120%` to the `.main-content` class that holds all elements.

One of the main reasons to use em instead of pixel is that this measurement unit doesn't depend on the screen size in Pixel while the pixel unit does. An button that has the markup you see below will look the same on any device at any scale - it is fully responsive in other words.

```css
.button {
	padding:.5em .7em;
}
```

Another important point is that em Media Queries apply even if the user zooms the page. Go ahead an try to zoom this blog, at some point the responsive layout (breakpoint 55em) will apply - if you use pixel in your breakpoint (for example `@media all and (max-width:800px)`) it will not apply the responsive layout when someone zooms in.

### Final Words
em is a lot better than pixel even if it takes some time to change your mind and work flow. It is *more flexible*, *more scalable* and - which is the most important, *completely responsive*.
So if you create a new responsive website, and you should always do so, do your self a favor and use em instead of px.

### Further Reading

More articles on this topic:

* [Sizing (Web) components](https://medium.com/front-end-development/8f433689736f) by [Simurai](http://simurai.com/)
* [Font-size with rem ](http://snook.ca/archives/html_and_css/font-size-with-rem) by [Jonathan Snook](http://snook.ca)
* [The ems have it: Proportional Media Queries FTW](http://blog.cloudfour.com/the-ems-have-it-proportional-media-queries-ftw/) by [Lyza Gardner](http://blog.cloudfour.com/author/lyza-gardner/)
* [em vs px](http://codepen.io/LukyVj/blog/em-vs-px) by [Lucas Bonomi](http://lucasbonomi.com/)
