---
layout: post
title: Keeping responsive aspect ratio
category: web
---
What to do if you need to make a responsive YouTube video?
That is, when you shrink your window your content should properly adapt and do not look silly like
having 480x640 instead of 480x320 (for 4:3 aspect ratio).

It is natural to use a 100% width on your content,
but the sad thing is that `iframe` doesn't count content height
and it will be set to some default value (like 150px, depends on browser).

Actually there is a simple way to make it work.
We will add a dummy `div` placeholder right before our naughty content that will act like spacer in our layout.

```html
<div class="container">
  <div class="placeholder"></div>
  <iframe class="content"
          src="//www.youtube.com/embed/24NrM040osk"
          frameborder="0">
  </iframe>
</div>
```

And then we need to fix our content with `position: absolute` to display on top of the placeholder:

```css
.container {
  position: relative;
}

.placeholder {
  padding-bottom: 75%; /* 100% / (4 / 3) = 75% */
}

.content {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
}
```
Done! So that if you want to fit a 16:9 video you should set your placeholder height to 100% / (16 / 9) &asymp; 56%.

I use this technique in my blog to fit video and `iframe`'s in various screen sizes. Hope it will help you too.

Have a nice day!
