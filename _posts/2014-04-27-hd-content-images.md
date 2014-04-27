---
layout: post
category: web
tags: css
title: High Definition Content Images
---

Just want to share a simple technique I used not so long ago to display different resolution content imagery. I like this solution because it uses only a bunch of HTML with some CSS and media queries. There is exactly zero lines of JavaScript.

Unless there is [no strong support](http://caniuse.com/#search=srcset) for `srcset` attribute ([W3C spec](http://www.w3.org/html/wg/drafts/srcset/w3c-srcset/)) I'll prefer this one over any JS-based solutions because I belive that JavaScript should stay out of any non-interactive presentation details as much as possible.

## Example
For our content we use three 400x400px kitten images (from great placeholder service [placekitten.com](http://placekitten.com/)) for three different resolutions: 1x, 1.5x and 2x. For the sake of example these pictures are not the same. So any screen with pixel density below 1.5 will display first image, from 1.5 to 2 – second one and over 2 – third.

```html
<div class="img_resolution_96dpi">
    <i class="img" style="background-image: url(http://placekitten.com/400/400)"></i>
</div>

<div class="img_resolution_144dpi">
    <i class="img" style="background-image: url(http://placekitten.com/600/600)"></i>
</div>

<div class="img_resolution_192dpi">
    <i class="img" style="background-image: url(http://placekitten.com/800/800)"></i>
</div>
```

Here we use `background-image` instead of `<img>` because latter would load it's image regardless of visibility of the image itself (e.g. CSS `display` property).

It's quite a lot of boilerplate html just for one image, so I encourage you to consider using some kind of template engine if not yet.

In CSS we use `min-device-pixel-ratio` to match devices pixel density and display appropriate image:

```css
.img {
  width: 400px;
  height: 400px;

  display: inline-block;

  border: 0;
  background: no-repeat;
  background-size: cover;
}

/* Hiding hi res images by default */
.img_resolution_144dpi { display: none; }
.img_resolution_192dpi { display: none; }

@media only screen and (-webkit-min-device-pixel-ratio: 1.5),
       only screen and (     -o-min-device-pixel-ratio: 1.5),
       only screen and (min-resolution: 144dpi) {
    .img_resolution_96dpi { display: none; }
    .img_resolution_144dpi { display: initial; }
}

@media only screen and (-webkit-min-device-pixel-ratio: 2),
       only screen and (     -o-min-device-pixel-ratio: 2),
       only screen and (min-resolution: 192dpi) {
    .img_resolution_96dpi { display: none; }
    .img_resolution_144dpi { display: none; }
    .img_resolution_192dpi { display: initial; }
}
```

## Demo
Now you should see different kittens on screens with different pixel density. Here is [a little demo](http://cdpn.io/xsjnf):

<p data-height="587" data-theme-id="0" data-slug-hash="xsjnf" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/anton-rudeshko/pen/xsjnf/'>Responsive image</a> by Anton Rudeshko (Tesla) (<a href='http://codepen.io/anton-rudeshko'>@anton-rudeshko</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

I've also managed to capture a short video of me switching pixel density and showing network requests:

<div class="post__container">
  <div class="post__placeholder_16x9"></div>
  <iframe width="853" height="480" src="//www.youtube.com/embed/-1AcnFgXzJM?rel=0" frameborder="0" allowfullscreen></iframe>
</div>

As you can see browser is requesting only the right image that we need. And that's it!

## Future
In the future with `srcset` you can do the same thing with pretty much no code at all:

```html
<img src="http://placekitten.com/200/200" srcset="http://placekitten.com/400/400 2x" />
```

Isn't it great? The only downside is that you can't use fractional pixel density.

Is there any better way to achieve this? Feel free to leave me a comment.

Thanks for joining me this week and have a nice day!
