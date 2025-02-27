---
categories:
  - coding
  - thoughts
  - frontend
date: 2014-01-02T00:00:00Z
title: Web Development in 2014
tags:
  - css
aliases:
  - /web-development-in-2014/
---

As soon as you start to really dig into web development it feels like things would change daily - at least I feel that way. But there's one thing that I'd like everyone to do in 2014: At least **basic responsive web development** to ensure your site - at least - looks good and is usable on whatever screen it will be displayed. There's nothing I hate more than scrolling websites around on my phone when I want to read an article (to be honest, I mostly leave the site and never come back). However, I'll give you a short look into basic responsive design/development patterns.

### Planning
First of all make sure what your site is about and what your content is like. **Never hide content!** Hiding content is the last resort and I prefer to avoid this. Make sure your site displays all the information you need the user to have in a order that makes sense: Do I want to scroll through your header, sidebar and a small advertisement area before I can find the article I'm searching for? No, I don't. Good responsive, scalable design starts with mark up and there are a few tricks to archive a good looking, usable website. Before we start to look at the code: I'll focus on blog-like websites here and will cover more complex things later in another post.

### Drop the m.domain
Some people may tell you that `m.domain.tld` is a great thing but it is not. Actually, it's the first and worst mistake you can make. There are a few reasons for this:

- you split your whole page into 2 domains (m.domain.tld and domain.tld)
- you need doubled content
- users on desktop don't get re-directed to the domain.tld when clicking on a m-dot link
- when the windows gets resized your page doesn't fit the new viewport

There are even more reasons to drop m-dot domains but I think those should make it clear. Another downside to "smartphone"-only optimization is the last point on the list: The resizing. I really like to re-size a page to half it's size on desktop so that I can, for example, follow a guide and do stuff inside my editor without switching between the windows all the time. This screenshot of [spiegel.de](http://spiegel.de) shows the downside of "mobile-only" responsive design.
![Screenshot of Spiegel Online, left the desktop version half it's size, right the mobile version](https://i.kevingimbel.me/sc/screenshot-16-46.png "Screenshot of Spiegel Online, left the desktop version half it's size, right the mobile version")

So, the key of a good, flexible design is the CSS and the breakpoints. I prefer to create them in em instead of pixel or - even worse - exact screen sizes to target different devices. If you keep your head out of the iOS world you'll find hundreds of different screen sizes, would you target them all with exact pixels? I guess you wouldn't want to and wouldn't do it. So what else can we do? Support just a few devices, let's say a min-width of 600px? No. We'll use em instead and build a responsive site with 3 to 4 breakpoints like so:

```css 
@media all and (min-width: 80em) {
    /* large screens */
}

@media all and (max-width: 80em) {
    /* medium up to large */
}

@media all and (max-width: 50em) {
    /* small screens as well as sites pinned to one-half of the screen */
}

@media all and (max-width: 30em) {
    /* small devices like smartphones */
}
```

I once created a page that only [shows the current breakpoint](http://dev.kevingimbel.me/breakpoint/) and based on this I found that the above breakpoints are safe to use. However, just knowing the breakpoints isn't the goal. Next it comes to markup and grids while I'll use a early beta of Bullgrid, a Grid System I'm working on with [Tim Pietrusky](http://twitter.com/timpietrusky) at the moment. It's based on [inuit CSS Grids](http://inuitcss.com/2012/12/building-grid-systems-with-inuit-css/) but moved into one file and re-written to use [em instead of px](/em-vs-px).

### The Markup
The first Demo Markup for this post is a simple blog with a large header, a content area and a sidebar as well as a footer.

<p data-height="400" data-theme-id="647" data-slug-hash="imvDF" data-user="kevingimbel" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/kevingimbel/pen/imvDF'>imvDF</a> by Kevin (<a href='http://codepen.io/kevingimbel'>@kevingimbel</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async "//codepen.io/assets/embed/ei.js"></script>

When you open the pen in [fullscreen view](http://codepen.io/kevingimbel/full/imvDF) and resize it you can see that the sidebar moves underneath the article itself so that it is not in the way but visible to everyone who finish the article - in my opinion a good way to hide a sidebar on small screens because it doesn't require an extra click to open or see the sidebar and it is still there.

This all works with a Grid I'll talk about next and the way the markup is written.

```html 
<!-- Wrapping the Grid inside the gw class -->
<div class="wrapper gw">
    <header class="g one-whole  small-one-whole">
        <!-- the header always stays at 100% of the available width -->
    </header>

    <main class="g two-thirds  small-one-whole">
        <!--
        The content takes 2/3 of the space
        -->
    </main>

    <aside class="g one-third  small-one-whole">
        <!--
        The Sidebar takes the remaining 1/3 of the space and because it's
        below the content in the DOM order it moves below it when the page
        get's resized. That's basically the "Markup magic"
        -->
    </aside>

    <footer class="g one-whole">
        <!-- Again, one-whole = all available width -->
    </footer>
</div>
```

You may wonder what this `small-one-whole` thing does? I'll cover that next!

### The Grid

As mentioned above I'm using an early beta of Bullgrid. However, the best thing about this grid is that it is re-usable, easy to understand and you can see the behavior of elements inside the markup. Here's an example.

```html 
<article class="g one-third  medium-one-half  small-one-whole">
    <!-- Article 1 -->
</article>

<article class="g one-third  medium-one-half  small-one-whole">
    <!-- Article 2 -->
</article>

<article class="g one-third  medium-one-half  small-one-whole">
    <!-- Article 3 -->
</article>
```

So the above results in something similar to this.

<p data-height="500" data-theme-id="647" data-slug-hash="aifLb" data-user="kevingimbel" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/kevingimbel/pen/aifLb'>aifLb</a> by Kevin (<a href='http://codepen.io/kevingimbel'>@kevingimbel</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async "//codepen.io/assets/embed/ei.js"></script>

As you can see the article's width is always adjusted to the width of the screen and it is completly readable inside the code.

The hearth of this grid is the breakpoint mixin you can see below.

```css 
@mixin breakpoint($point) {
  @if $point == "large" {
    @media all and (min-width: 80em) {
      @content;
    }
  }

  @if $point == "medium" {
    @media all and (max-width: 80em) {
      @content;
    }
  }

  @if $point == "small" {
    @media all and (max-width: 30em) {
      @content;
    }
  }
}
```

This mixin can be used like

```css 
    .my-class {
        width: 50%;

        @include breakpoint(small) {
            width:100%;
        }
    }
```

So when the breakpoint small is triggered `my-class` will have a width of 100% instead of 50%. Quite a lot to write in case of the grid right? [Harry Roberts](http://csswizardry.com/) has a handy `grid-setup()` SCSS function that we re-used for bullgrid. It looks as followed.

```css 
@mixin grid-setup($namespace: "") {

  /*
   * Hidden
   */
  .#{$namespace}hidden {
    display: none;
  }


  /**
   * Whole
   */
  .#{$namespace}one-whole {
    width: 100%;
  }


  /**
   * Halves
   */
  .#{$namespace}one-half {
    width: 50%;
  }


  /**
   * Thirds
   */
  .#{$namespace}one-third {
    width: 33.333%;
  }
  .#{$namespace}two-thirds {
    width: 66.666%;
  }


  /**
   * Quarters
   */
  .#{$namespace}one-quarter {
    width: 25%;
  }
  .#{$namespace}two-quarters {
    @extend .#{$namespace}one-half;
  }
  .#{$namespace}three-quarters {
    width: 75%;
  }

/*
    and so on....

*/

  .#{$namespace}eleven-twelfths {
    width: 91.666%;
  }
}
```

To set up the `small-`, `medium-` and `large-` grid we use `grid-setup` combined with `breakpoint`.

```css 
@include breakpoint(large){
  @include grid-setup("large-");
}

@include breakpoint(medium){
  @include grid-setup("medium-");
}

@include breakpoint(small){
  @include grid-setup("small-");
}
```

Once the grid is setup we can use all the different combinations and prefix them with `large-`,`meduim-` or `small-`. Basically that's it what I want to say about responsive development to this point. I think it is important to stop thinking in different categories like "mobile", "desktop", "TV" or whatever - it's more important to see a bunch of screens and devices that have a specific width you need to target with your breakpoints. Also I think every website you create this year should be responsive. There's no excuse anymore to do a non-responsive website. It should be a standard. So as a developer in a design team you should talk to your designers to keep responsive design in mind when making screen-designs. The Internet has become a canvas with a lot of sizes and we, as the artist, have to re-think the way we paint on it.

I appreciate your opinion and critic [@kevingimbel](http://twitter.com/kevingimbel) or via [eMail](/imprint).
