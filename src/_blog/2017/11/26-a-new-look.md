---
title: "CSS Custom Properties and a new look"
date: 2017-11-26T10:56:13+01:00
categories: 
  - coding
  - frontend
tags:
  - redesign
  - css
  - personal
  - modernization
  - accessibility
aliases:
  - /css-custom-properties-and-a-new-look/
---

You may have noticed that some things changed on this website. I completly re-wrote the Front-End and created a new theme with a focus on accessibility and well-structured content. On the web, Accessibility is enabled by default; All you need is a good HTML structure and your website is almost ready to go! A second important part of accessibility is color and contrast as well as font sizing. I decided to let users choose their own color scheme, font-size, and dark or light mode - all done with CSS Custom Properties ({%abbr "aka", "Also known as" %} CSS variables).

CSS variables are a [native CSS feature](https://www.w3.org/TR/css-variables-1/ "Read the CSS variable specs") which enables us as developers to re-use colors, font-sizes, and other properties throught our stylesheets. You may think _"But wait! Sass, Less, and Stylus had variables for years!"_ and you're right - the pre processors, which generate a CSS file, have had variables for years. What they did not have, however, is the abbility to modify and change these variables on the fly after the CSS had been generated. With CSS Custom Properties and some JavaScript we can modify the variables at runtime and the browser will re-render all pieces of the page which are using the variable - and that's exactly what I am doing with the settings on this website to change the accent colors and font sizing.

Below I will explain step by step how CSS variables look, work, and how we can use them to alter the look of a website - even persistent without any backend code!

## CSS variables

### Browser Support

Let's jump directly into browser support, which is looking pretty good in my opinion!

{% caniuse "css-variables" %}

`77.90%` (as of November 26. 2017) is not perfect but certainly good enough for me and my private website. I'd probably not rely on CSS Custom Properties in client projects yet tho.

### Syntax 

A CSS variable is a word preceded by two dashes (`--`), which looks like the following examples.

```css
--color: #ddd;
--base-font-size: 12px;
--breakpoint-large: 1200px;
```

Just like any CSS property they "cascade down". A variable defined at the top of the document on the `:root` or `html` selector will be defined everywhere in the document. To use a CSS variable we need to get it somehow. It's not enough to reference it, for example this does not work.

```css
// Does not work!
.my-selector {
  color: --color;
}
```

With CSS variables we need to retrieve the value by calling a `var` function. This function also takes a fallback parameter in case the variable is not set.

```css
// This does work
.my-selector {
  color: var(--color, #333)
}
```

`.my-selector` will have a color value equal to whatever is stored inside the `--color` variable or `#333` if it is not set.

A variable can later be changed to be "locally scoped". For this example we define a custom property named `--color` and set its value to `red`. We say all h1 elements should use the `--color` variable for their font-color. Then we create a CSS class named `local-scope` and inside we change the `--color` to `blue`. A `h1` element inside the `local-scope` will use the re-defined color value.

{% codepen "78d261a36ecded2b75d5260cb7056fce" "title:CSS Variables - Cascading & local scope" %}

We do not need to change the `h1` selector because `h1` will always have a font-color equal to `--color` - we only need to change the variable inside the local scope. The third `h1` element is outside the local scope and so it takes the  original `--color` value (red).

This type of inheritance is a powerful tool CSS has given us ("us" being developers). At runtime, when the website is loaded and all CSS is parsed and the website has been painted to the screen, we can still change these properties on-the-fly with JavaScript and the browser will simply re-render the parts that need changing. Of course, you can already change CSS on the go with JavaScript. You could select for example a bunch of elements by class name and then change their color to be `blue` instead of red. With CSS Custom Properties you don't need to do this! All we need to do is change the variable at a higher level, for example on the `body` element as the following example illustrates.

{% codepen "5245628703a6a223215cf5a30cf8294d" "title:CSS Variables - Changing with JS" %}

Click on the `Change color` button above and all the non-scoped `h1` elements will turn green. All we need for this to work is set an inline style on the `body` element.

```javascript
document.body.style = "--color: green;";
```

I use this technique to change the secondary and primary colors of my website from the settings menu. [These two lines](https://github.com/kevingimbel/kevingimbel.com/blob/9f11b96f428f01b1ae14f8673c2e4f48e8ee3b21/themes/next/static/js/a11y.settings.js#L83-L84 "View source code on GitHub.com") set the CSS variables `--color-accent-primary` and `--color-accent-secondary` which by default are dark blue and yellow.

### Recap

So let's recap this real quick:

- CSS variables (also called CSS Custom Properties) are a native CSS feature
- They cascade "down" in the CSS, just like any other property (`font-size`, `color`, etc.)
- They can be locally scoped, that is changed for a certain element and it's children
- They can be changed with JavaScript at runtime and the browser will re-render every element which uses them

At this point it becomes clear why they are an advantage over pre-processor variables. We have more control of changing them and they present us with a powerful new way to implement multiple layouts for our websites.

## Implementing persistent settings

As I mentioned before for my website (this very blog you read right now), the Settings are persistent. If you change the font size or colors and navigate to a new page you'll still have your custom styles - not my default styles. I do not use any backend software like PHP, Go, or Ruby; Instead this website is a static website, which means all HTML pages are rendered before and then deployed to [Netlify](https://www.netlify.com/). So there is no backend which can save your settings and then send them back to your browser once you navigate to a new site.

The saving part is done on the client side, inside your browser. A browser feature called [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage). Local Storage is a text-based, key-value storage which developers can use to store (small) text based values inside the browser. This allows us to have some sort of persitent storage on the client side without the need for a backend. For my use case (storing settings) this is perfect.

When you open the settings menu and click "Save" I grab all the values with JavaScript ([See the code on GitHub](https://github.com/kevingimbel/kevingimbel.com/blob/9f11b96f428f01b1ae14f8673c2e4f48e8ee3b21/themes/next/static/js/a11y.settings.js#L123-L138))

```javascript
settingsForm.addEventListener('submit', function(e) {
  e.preventDefault();
  var fd = new FormData(settingsForm).entries();

  for (var cssRule of fd) {
    lsSettings[cssRule[0]] = cssRule[1];
  }

  setStylesAndCreateForgroundColors(lsSettings);
  populateSettingsFromArray(lsSettings);

  localStorage.setItem('a11y_settings', JSON.stringify(lsSettings));
  document.body.classList.remove('page-settings--open');
  document.body.setAttribute('tabindex', "0");
  return false;
});
```

There's quite a lot going on. The important bit for now is `localStorage.setItem('a11y_settings', JSON.stringify(lsSettings));`. This line sets a new item in the local storage named `a11y_settings`, scoped to my website (kevingimbel.com). This JSON object holds all relevant information for your custom settings and looks like this:

```json
{
  "--body-invert": 0,
  "--color-accent-primary": "#3e934b",
  "--color-accent-secondary": "#70cfff"
}
```

The key of the JSON object is always the CSS variable name, the value is the CSS variable value. With these settings we get the following result.

{%figure "/images/posts/2017/css-custom-properties/kevingimbel_com-with-custom-settings.png", "The look of kevingimbel.com with the above settings" %}

Because we use inline styles to change the CSS variables the HTML element will have the following styles.

```html
<html style="--body-invert:0; --color-accent-primary:#3e934b; --color-accent-secondary:#70cfff;">
```

Once the JSON is saved to local storage, I load in on page load and apply it immediately - which happens to be really fast so it seems the styles are not even applyed again! The piece of JavaScript to load the custom styles is placed above all other content, at the beginning of the `<body>`. To not impact performance too much I minified it and it looks like this.

```javascript
<script type="text/javascript">(function(){if(localStorage.getItem("a11y_settings")){var a=JSON.parse(localStorage.getItem("a11y_settings"));for(var o in a){document.documentElement.style.setProperty(o,a[o]);if(o=="--body-invert"&&a[o]=="100"){document.body.classList.add("dark-mode")}}if(a11y){a11y.o=a}}})();</script>
```

Unminified the code reads as follows

```javascript
(function(){
  if(localStorage.getItem("a11y_settings")) {

    var styles = JSON.parse(localStorage.getItem("a11y_settings"));

    for(var rule in styles) {
      document.documentElement.style.setProperty(rule,styles[rule]);
      // Check if we "dark mode" is enabled (more below!)
      if(rule=="--body-invert" && styles[rule]=="100"){
        document.body.classList.add("dark-mode")
      }
    }
    // if a11y is defined, save the rules in the object
    if(a11y){
      a11y.styles = styles
    }
  }
})();
```

What happens here is the following:

- We check if there are custom setting (`if( localStorage.getItem("a11y_settings") )`)
- Then we read it in, it's a JSON string so we need to parse it
- Next we loop through all properties of the JSON, which are key-value pairs like `{ "--color": "#ddd" }`
- Each key-value pair is set as inline style on the documentElement (the `html` element)

For the `--body-invert` value we take an extra step and set a CSS class on the body. This is used for the dark mode which requires the extra class to work properly. The dark-mode is a CSS filter. What I do is invert the body with a CSS Filter so the default light theme becomes a dark theme. The CodePen below shows this in action. By adding the `dark-mode` class to the second block we invert all colors, which results in the block being dark.


{%codepen "604946ab15c48299b1f7b54b7a758cfb" "CSS Filter Invert" %}

That's exactly what happens when you click "Dark mode" in the settings menu above. So why do we need the class `dark-mode`? Because with `filter: invert(100%)` everything is inverted including images and videos - which we do not want. These elements should not be inverted so we need to apply a `filter: invert(100%)` to them when dark mode is active, which is done with the following CSS snipped.

```css
body.dark-mode img,
body.dark-mode video,
body.dark-mode iframe {
  filter: invert(100%) !important;
}
```

This basically means we apply the filter two times, which results in resetting it. The images get inverted from the first rule, then inverted again to normal color.

```css
// Invert everything
body.dark-mode {
  filter: invert(100%);
}
// Invert img, video, and iframe again, resulting in resetting the original invert.
body.dark-mode img,
body.dark-mode video,
body.dark-mode iframe {
  filter: invert(100%) !important;
}
```

And that's it for CSS variables and the redesign. You get to choose how my blog looks, what font-size to use and if a dark or light theme is best for you. I might add more options to the settings in the future, but for now I'm happy with the result.

Got any feedback? Want to tell me how much the news design sucks? Love it? Hit me up on [Twitter @kevingimbel](https://twitter.com/_kevinatari "Find me on twitter")
