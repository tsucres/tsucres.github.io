---
layout: post
title: 'How to progressively load large images in Javascript'
description: 'A summary and explanation of different techniques I tried to mimic the way Medium loads the images on its posts.'
tags: [Js]
date-string: DECEMBER 6, 2017
image:
  feature: 2017-12-06/feature.jpg
  tinyimg: 2017-12-06/feature_mini.jpg
---

When building a webpage that contains large images, you don't want to force your users to wait till all the images of the page are loaded before they can actually use it. You most probably want to control the way those images are loaded. 

The solution described here consists in showing a very small miniature of the real image, and transitioning to the full image when it's done loading. A lot of popular websites, as for example [Medium](https://www.medium.com), [Quora](https://www.quora.com) or even [Facebook](https://www.facebook.com), already implement this functionality.

<p align="center">
<img src="https://github.com/tsucres/BluePil.js/raw/master/screenshots/bluepil_mini.gif">
</p>
<!-- TODO: demo button -->


Several libraries already exists for this. I listed some of them at the end of the article. However, they generally don't handle background-images and lack of customisability.


Here, I'll focus on a method inspired from the [pilpil.js](https://github.com/zafree/pilpil) library that I adapted to work with background-images. 

I wrapped the whole method in a js library that I randomly called [BluePil.js](https://github.com/tsucres/BluePil.js). Its [demo page](https://tsucres.github.io/BluePil.js/) is a good illustration of what is discussed in this post.


<center><a href="https://tsucres.github.io/BluePil.js/">DEMO</a></center>




# Background images

For this section, I assume that the `background-size` is `cover`, which is the most common option. 



## Markup

The markup is fairly simple, it consists in adding the following elements: 
1. the class `progressive-bg-image` to the element with the background
2. an `img` tag for the miniature image and one for the full image
3. a canvas to draw a blurred version of the miniature


```html
<div class="progressive-bg-image">
    <!-- Progressively loads the background -->
    <img class="hidden thumbnail" src="<mini image path>"> 
    <img class="full-bg-image hidden" src="<full image path>">
    <canvas class="full-absolute progressive-img-load-canvas"></canvas>
    <!-- =============== -->

    <!-- your content (relatively positioned) ... -->
</div>
```

```css
.full-absolute {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    width: 100%;
    height: 100%;
}
.progressive-img-load-canvas {
    -webkit-transition: opacity .5s;
            transition: opacity .5s; 
}
.full-loaded .progressive-img-load-canvas {
    opacity: 0;
}
.progressive-bg-image {
    background-position: center center;
    background-size: cover;
    position: relative;
}
.hidden {
    display: none;
    visibility: hidden;
}
```

- the class `progressive-bg-image` will be used by the js code to locate the DOM elements the effect has to be applied on. 
- the `full-absolute` class makes the canvas fit the parent element (and makes it look like its background). 
- the `img.thumbnail` and `img.full-bg-image` elements are only used to specify to the browser which images to download and to make it download them asap. This is why they're hidden.
- the canvas is used to render the blurred miniature. The `progressive-img-load-canvas` class defines the transition on the visibility of the canvas. It will also be used by the js code to identify the canvas.
- the `full-loaded` class will be added (by the js code) to the root div (`.progressive-bg-image`) when the full image is loaded on the page. It's used to hide the canvas.

When the full image is finished downloading, it will be assigned as the background for the `.progressive-bg-image` element. Then the `full-loaded` class will be added to the root element to make the canvas disappear smoothly.

Concerning the size of the miniature, I usually use images with a width of around 20 pixels. An image of this resolution usually weights a couple of kilobytes and, thanks to the blur effect, is sufficient to look good.

## Javascript

Now, let's make that markup come to live! 

We will implement two functions: 
- the first one (`initBgImage`) will download and draw the miniature in the canvas
- the second one (`loadFullBgImage`) will download the full image and display it instead of the miniature. We'll also implement a transition effect.

Implementing those methods separately will allow us to call them at different times. For example, (spoiler alert) drawing the miniature when the page loads, but start downloading the full image only when the element appears in the viewport (typically when the user scrolls).

```js
function initBgImage(root_el) {
    var miniatureImg = root_el.querySelector(".thumbnail");
    var canvas = root_el.querySelector(".progressive-img-load-canvas");
    drawMiniatureForBgImage(root_el, miniatureImg, canvas);
}

function drawMiniatureForBgImage(root_el, miniatureImg, canvas) {
    var thumbnail = new Image();
    thumbnail.onload = function () {
        var canvasImage = new CanvasImage(canvas, thumbnail);
        canvasImage.blur(2);
    };
    thumbnail.src = miniatureImg.src;
}

```



The `CanvasImage` object is borrowed from the [pilpil.js](https://github.com/zafree/pilpil) library. It acts as an extension to the htmlCanvasElement and adds a `blur` method to it. This method is then used (in the `drawMiniatureForBgImage` function) to draw the miniature image.

```js
// source: pilpil.js
CanvasImage = function (canvasEL, image) {
    this.image = image;
    this.element = canvasEL;
    canvasEL.width = image.width;
    canvasEL.height = image.height;
    this.context = canvasEL.getContext('2d');
    this.context.drawImage(image, 0, 0);
};
CanvasImage.prototype = {
    blur:function(e) {
        this.context.globalAlpha = 0.5;
        for(var t = -e; t <= e; t += 2) {
            for(var n = -e; n <= e; n += 2) {
                this.context.drawImage(this.element, n, t);
                var blob = n >= 0 && t >= 0 && this.context.drawImage(this.element, -(n -1), -(t-1));
            }
        }
    }
};
```


As for the `loadFullBgImage` function, its implementation is pretty straightforward: 

```js
function loadFullBgImage(root_el) {
    var fullBgImg = root_el.querySelector(".full-bg-image"),
    canvas = root_el.querySelector(".progressive-img-load-canvas");
    
    // Callback when the full image is downloaded.
    // Shows the full image and add the full-loaded class to root_el to hide the miniature.
    function fullBackgroundImageLoaded() {
        root_el.style.backgroundImage = 'url(' + fullBgImg.src + ')';
        window.requestAnimationFrame(function() { 
            // requestAnimationFrame makes sure the image is showed before removing the miniature.
            root_el.classList.add("full-loaded");
        });
    } 

    // Start downloading the full image.
    if (fullBgImg.src && fullBgImg.complete) {
        fullBackgroundImageLoaded();
    } else {
        fullBgImg.addEventListener('load', fullBackgroundImageLoaded);
    }
}
```




Note that we can easily call those functions for every `.progressive-bg-image` element of the page with this simple javascript: 

```js

// After window has loaded! (in the window.onload)

var progressiveBgs = document.getElementsByClassName("progressive-bg-image");

// 1) load/show all miniatures
for (var i = 0; i < progressiveBgs.length; i++) {
    try {
        this.initBgImage(progressiveBgs[i]);
    } catch(err) {
        // Let's catch the errors so that if one image in the document 
        // fails to load, it doesn't affect the other ones.
        console.error(err); 
    }
}

// 2) start downloading the full images
for (var i = 0; i < progressiveBgs.length; i++) {
    try {
        this.loadFullBgImage(progressiveBgs[i]);
    } catch(err) {
        console.error(err); 
    }
}

```


You can see the result in the jsfiddle bellow: 


<iframe width="100%" height="300" src="//jsfiddle.net/tsucres/63vhfaxt/7/embedded/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## Ameliorations


### noscript fallback

The problem with the previous markup is that if the user's browser doesn't run javascript, the image will never be assigned as the background of the `div` and stay invisible to the user. An easy workaround is to add an `img` tag inside a `noscript` tag and give it a style such that it looks like a background image.

This gives us the following markup: 

```html
<div class="progressive-bg-image">
    <noscript><img class="full-absolute object-fit-cover" src="<full image path>"></noscript>
    <img class="hidden thumbnail" src="<mini image path>"> 
    <img class="full-bg-image hidden" src="<full image path>">
    <canvas class="full-absolute progressive-img-load-canvas"></canvas>
</div>
```

With the additional style: 

```css
.object-fit-cover {
    -o-object-fit: cover;
       object-fit: cover;
}
```


This class with the `.full-absolute` class make the `img` appearing like a background.


### Background position

You may want to change the position of the background image through the css property `background-position`. If you do so with the current javascript, you will see that the position of the image in the canvas doesn't match the background image. 

To solve this, we have to modify the way we draw the miniature in the canvas. We will use the following function that mimics the behavior of the `background-position: cover` setting: 

```js
// This function was conveniently found on this stackoverflow question: 
// https://stackoverflow.com/questions/21961839/simulation-background-size-cover-in-canvas/21961894
/**
 * By Ken Fyrstenberg Nilsen
 *
 * drawImageProp(context, image [, x, y, width, height [,offsetX, offsetY]])
 *
 * If image and context are only arguments rectangle will equal canvas
*/
function drawImageProp(ctx, img, x, y, w, h, offsetX, offsetY) {

    if (arguments.length === 2) {
        x = y = 0;
        w = ctx.canvas.width;
        h = ctx.canvas.height;
    }

    /// default offset is center
    offsetX = typeof offsetX === 'number' ? offsetX : 0.5;
    offsetY = typeof offsetY === 'number' ? offsetY : 0.5;

    /// keep bounds [0.0, 1.0]
    if (offsetX < 0) offsetX = 0;
    if (offsetY < 0) offsetY = 0;
    if (offsetX > 1) offsetX = 1;
    if (offsetY > 1) offsetY = 1;

    var iw = img.width,
        ih = img.height,
        r = Math.min(w / iw, h / ih),
        nw = iw * r,   /// new prop. width
        nh = ih * r,   /// new prop. height
        cx, cy, cw, ch, ar = 1;

    /// decide which gap to fill    
    if (nw < w) ar = w / nw;
    if (nh < h) ar = h / nh;
    nw *= ar;
    nh *= ar;

    /// calc source rectangle
    cw = iw / (nw / w);
    ch = ih / (nh / h);

    cx = (iw - cw) * offsetX;
    cy = (ih - ch) * offsetY;

    /// make sure source rectangle is valid
    if (cx < 0) cx = 0;
    if (cy < 0) cy = 0;
    if (cw > iw) cw = iw;
    if (ch > ih) ch = ih;

    /// fill image in dest. rectangle
    ctx.drawImage(img, cx, cy, cw, ch,  x, y, w, h);
}
```


Using this function, we can easily change the position of the drawn image using the `xOffset` and `yOffset` parameters. Those must be numbers between 0 and 1.

To make the code as versatile as possible, we will retrieve the position of the background using the following code: 

```js

// in drawMiniatureForBgImage()

offsets = [];
window.getComputedStyle(root_el, null).backgroundPosition.split(" ").forEach(function(offset) {
    offsets.push(parseInt(offset) / 100);
});
// At this point, xOffset = offsets[0] and yOffset = offsets[1]
```

This will ensure that the position of the background will be matched by the miniature in the canvas. 

Finally, we adapt the `CanvasImage` like this: 

```js
/**
 * CanvasImage(canvasEl, image [, w, h [, xOffset, yOffset]])
 * canvasEl: the canvas to draw the image on
 * image: the image to draw
 * w: the width to give to the image on the canvas
 * h: the height to give to the image on the canvas
 * xOffset: number in [0 : 1], default: 0.5
 * yOffset: number in [0 : 1], default: 0.5
 */
CanvasImage = function (canvasEl, image, w, h, xOffset, yOffset) {
    this.image = image;
    this.element = canvasEl;
    canvasEl.width = typeof w === 'number' ? w : image.width;
    canvasEl.height = typeof h === 'number' ? h : image.height;
    
    xOffset = typeof xOffset === 'number' ? xOffset : 0.5;
    yOffset = typeof yOffset === 'number' ? yOffset : 0.5;
    this.context = canvasEl.getContext('2d');
    drawImageProp(this.context, image, 0, 0, canvasEl.width, canvasEl.height, xOffset, yOffset);
};

// ......

// in drawMiniatureForBgImage()

var canvasImage = new CanvasImage(canvas, thumbnail, canvas.offsetWidth, canvas.offsetHeight, offsets[0], offsets[1]);
canvasImage.blur(2);

```

Now you can just change the `background-position` of the `.progressive-bg-image` element and observe that the miniature in the canvas matches its position.


### Make the images load only when they enter the viewport

Now we want to prevent the images to load straight away and trigger the download only when they appear in the viewport (as the user scrolls down). 

To achieve this, we need to slightly change the markup to remove the `src` attribute from the full-background image (otherwise, the browser will start loading the image as soon as it parses the DOM): 

```html

...
    <img class="full-bg-image hidden" data-src="<full image path>">
...

```


When we finally want to start download the image, we have to assign the value of `data-src` to the `src` attribute. To that end, we add the following code in the `loadFullBgImage` function: 

```js
// ...

    // Start downloading the full image.
    if (fullBgImg.src && fullBgImg.complete) {
        fullBackgroundImageLoaded();
    } else {
        fullBgImg.addEventListener('load', fullBackgroundImageLoaded);
        if (fullBgImg.hasAttribute("data-src")) {
            fullBgImg.src = fullBgImg.dataset.src;
        }
    }
 
// ...

```


All we have to do now, is call `loadFullBgImage` when it pleases us. This can be when the image enters the viewport. To do so, the idea is to use a function (let's call it `loadNewlyAppearedImages`) to check, for each of the "not yet loaded" `.progressive-bg-image`, if they are in the viewport and to bind it to the `scroll` event of the window. 


To simplify the detection of "not yet loaded" elements, we can add the class `scroll-loaded` to the elements to load with the scroll event.


```js

function loadNewlyAppearedImages() {

    // 1) Get all the "not yet loaded" elements
    var scrollLoadedElements = document.getElementsByClassName("scroll-loaded");

    // 2) get window measurements
    var window_height = window.innerHeight;
    var window_top_position = (window.pageYOffset !== undefined) ? window.pageYOffset : (document.documentElement || document.body.parentNode || document.body).scrollTop;
    var window_bottom_position = (window_top_position + window_height);

    for (var i = 0; i < scrollLoadedElements.length; i++) {
        // 3) calculate position of each element according to the viewport
        var el_rect = scrollLoadedElements[i].getBoundingClientRect();
        var element_top_position = window_top_position + el_rect.top;
        var element_bottom_position = element_top_position + el_rect.height;
        if ((element_bottom_position > window_top_position) && (element_top_position < window_bottom_position)) {

            // 4) load the elements inside the viewport
            if (scrollLoadedElements[i].classList.contains("progressive-bg-image")) {
                PIL.loadBgImage(scrollLoadedElements[i])
            } 

            // 5) Mark the element as loaded by the removing the 'scroll-loaded' class
            scrollLoadedElements[i].classList.remove("scroll-loaded");

        }
    }
}

document.addEventListener("scroll", loadNewlyAppearedImages);
document.addEventListener("resize", loadNewlyAppearedImages);


```


Another implementation would consist in loading the images sequentially, i.e. start loading an image as soon as the previous image (in the DOM) has loaded. This would prioritize the first images and hopefully make them load faster. I will not explicit the implementation here (it's pretty straightforward). Checkout [bluepil's demo](https://tsucres.github.io/BluePil.js/demo/sequential_loading.html) for an example.


### Automatically generate the markup

The code as it is now works just fine, but it's a bit heavy isn't it? If you're like me, you don't want to have to copy-paste it each time you want to put a background image somewhere on your webpages. So why not make a javascript function that builds the markup for us?

We will make a js function that generates the full markup described earlier out of this: 

```html
<div class="progressive-bg-image" data-full-image-path="<full image path>" data-mini-image-path="<mini image path>">
    <noscript><img class="full-absolute object-fit-cover" src="<full image path>"></noscript>
</div>
```



```js

function generateProgressiveBgImgMarkup(progressiveBgImageEl) {
    if (progressiveBgImageEl.dataset.hasOwnProperty("fullImagePath") 
        && progressiveBgImageEl.dataset.hasOwnProperty("miniaturePath")) {

        if (progressiveBgImageEl.dataset.hasOwnProperty("scrollLoaded")) {
            progressiveBgImageEl.classList("scroll-loaded");
        }


        var fullImagePath = progressiveBgImageEl.dataset["fullImagePath"];
        var miniaturePath = progressiveBgImageEl.dataset["miniaturePath"];

        var c = document.createDocumentFragment();
        var thumbnailImg = document.createElement("img");
        thumbnailImg.classList.add("hidden");
        thumbnailImg.classList.add("thumbnail");
        thumbnailImg.src = miniaturePath;
        c.appendChild(thumbnailImg);
        var fullBgImg = document.createElement("img");
        fullBgImg.classList.add("hidden");
        fullBgImg.classList.add("full-bg-image");
        fullBgImg.setAttribute("data-src", fullImagePath);
        c.appendChild(fullBgImg);
        var canvas = document.createElement("canvas");
        canvas.classList.add("full-absolute");
        canvas.classList.add("progressive-img-load-canvas");
        c.appendChild(canvas);
        progressiveBgImageEl.prepend(c);

        // little hack to make the css transition work with the dynamically created canvas
        window.getComputedStyle(canvas).opacity; 
    }
}

```

The js code above isn't the most elegant and concise way to dynamically add elements to the DOM but it's, as far as I know, one of the most efficient ones.

Basically, what it does is adding the two `img` tags and the `canvas` in the `div` we created, and append all the necessary classes to them.

**Note** about the little hack: [it seems](https://stackoverflow.com/questions/12088819/css-transitions-on-new-elements) that there's a race between the css transition and layout completing that sometimes makes the canvas disappearing before the image had the time to show up (you can try to remove it to see how it affects the result). 



# Img tags

As said in the introduction, there are already several libraries and articles that cover progressively loaded images. So I won't describe it in as much details as I did for the background-images. Beside, it's quite similar to the way background-images works.

## Markup

Here's the markup I use: 

```html
<div class="aspectRatioPlaceholder">
    <div class="aspectRatioPlaceholder-fill"></div>
    <div class="progressiveMedia full-absolute" data-width="<full image width>" data-height="<full image height>">
        <img class="progressiveMedia-thumbnail hidden" src="<mini image path>"/>
        
        <img class="progressiveMedia-image full-absolute hidden" data-src="<full image path>" />
        <canvas class="progressiveMedia-canvas full-absolute"></canvas>
    </div>
</div>

```


With its associated css:

```css
.aspectRatioPlaceholder {
  position: relative;
  width: 100%;
  margin: 0 auto;
  display: block; 
}

.progressiveMedia-canvas {
    -webkit-transition: opacity .5s;
    transition: opacity .5s; 
}
.full-loaded .progressiveMedia-canvas {
    opacity: 0;
}

```




As you can see it's pretty much the same as the one used by [pilpil.js](https://github.com/zafree/pilpil).

The main difference with background images is the need to explicitly specify the size of the image. This will allow the `.aspectRatioPlaceholder-fill` element to take the place of the image in the layout until the image finishes loading.

## Javascript

As for the background images, the js code is divided in two main functions: `initImage` to draw the blurred miniature in the canvas and `loadFullImage` to load the full size image.

```js

function initImage(root_el) {
    var miniatureImg = root_el.querySelector(".progressiveMedia-thumbnail");
    var canvas = root_el.querySelector(".progressiveMedia-canvas");
    var placeholderFilDiv = root_el.previousElementSibling;
    var width = root_el.dataset.width;
    var height = root_el.dataset.height;
    drawMinatureForImage(miniatureImg, canvas, placeholderFilDiv, width, height);
};

function drawMinatureForImage(miniatureImg, canvas, placeholderFilDiv, width, height) {
    var fill = height / width * 100;
    placeholderFilDiv.style = 'padding-bottom:'+fill+'%;';

    var smImageWidth = miniatureImg.width,
    smImageheight = miniatureImg.height;

    canvas.height = smImageheight;
    canvas.width = smImageWidth;

    var img = new Image();
    img.onload = function () {
        var canvasImage = new CanvasImage(canvas, img);
        canvasImage.blur(2);
    };
    img.src = miniatureImg.src;
}

```


```js

function loadFullImage(root_el) {
    var fullImg = root_el.querySelector(".progressiveMedia-image"),
    canvas = root_el.querySelector(".progressiveMedia-canvas");
    
    fullImg.onload = function() {
        fullImg.classList.remove("hidden");
        root_el.classList.add("full-loaded");
    }
    fullImg.src = fullImg.dataset.src;
}

```



The final result is illustrated in the following jsfiddle: 



<iframe width="100%" height="300" src="//jsfiddle.net/tsucres/1sjksa7a/embedded/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>


## Ameliorations

The markup can easily be improved to support the previously described features. 


### Noscript fall-back

To make the code functional even if javascript is disabled, we can add this simple `noscript` tag just after the `.aspectRatioPlaceholder`:

```html
<div class="aspectRatioPlaceholder">
// ...
</div>
<noscript><img src="<full image path>" class="full-bg-image"></noscript>
```


### Markup generation

To simplify the integration of new images in my documents, I created the following function to dynalically generate the required markup: 

```js

/* 
FROM (root_el): 
<img class="progressive-image" data-full-image-path="<full image path>" 
        data-miniature-path="<mini image path>" 
        data-full-image-height="<height of full image>" 
        data-full-image-width="<width of full image>">

TO (returned node): 

<div class="aspectRatioPlaceholder">
    <div class="aspectRatioPlaceholder-fill"></div>
    <div class="progressiveMedia full-absolute" data-width="<full image width>" data-height="<full image height>">
        <img class="progressiveMedia-thumbnail hidden" src="<mini image path>"/>
        
        <img class="progressiveMedia-image full-absolute hidden" data-src="<full image path>" />
        <canvas class="progressiveMedia-canvas full-absolute"></canvas>
    </div>
</div>

*/
function generateProgressiveImgMarkup(root_el) {
    if (root_el.dataset.hasOwnProperty("fullImagePath") 
        && root_el.dataset.hasOwnProperty("miniaturePath")
        && root_el.dataset.hasOwnProperty("fullImageWidth")
        && root_el.dataset.hasOwnProperty("fullImageHeight")) {

        var fullImagePath = root_el.dataset["fullImagePath"];
        var miniaturePath = root_el.dataset["miniaturePath"];
        var fullImageWidth = root_el.dataset["fullImageWidth"];
        var fullImageHeight = root_el.dataset["fullImageHeight"];
        
        var c = document.createDocumentFragment();
        
        var rootDiv = document.createElement("div");
        rootDiv.className = root_el.className;
        rootDiv.classList.add("aspectRatioPlaceholder");

        var aspectRatioPlaceholderFill = document.createElement("div");
        aspectRatioPlaceholderFill.classList.add("aspectRatioPlaceholder-fill");
        rootDiv.appendChild(aspectRatioPlaceholderFill);

        var progressiveMedia = document.createElement("div");
        progressiveMedia.classList.add("progressiveMedia");
        progressiveMedia.classList.add("full-absolute");
        progressiveMedia.setAttribute("data-width", fullImageWidth);
        progressiveMedia.setAttribute("data-height", fullImageHeight);
        if (root_el.dataset.hasOwnProperty("scrollLoaded")) {
            progressiveMedia.classList.add("scroll-loaded");
        }
        rootDiv.appendChild(progressiveMedia);

        var thumbnailImg = document.createElement("img");
        thumbnailImg.classList.add("progressiveMedia-thumbnail");
        thumbnailImg.classList.add("hidden");
        thumbnailImg.src = miniaturePath;
        progressiveMedia.appendChild(thumbnailImg);

        var fullImg = document.createElement("img");
        fullImg.classList.add("progressiveMedia-image");
        fullImg.classList.add("hidden");
        fullImg.classList.add("full-absolute");
        fullImg.setAttribute("data-src", fullImagePath);
        fullImg.alt = root_el.alt;
        progressiveMedia.appendChild(fullImg);

        var canvas = document.createElement("canvas");
        canvas.classList.add("progressiveMedia-canvas");
        canvas.classList.add("full-absolute");
        progressiveMedia.appendChild(canvas);

        c.appendChild(rootDiv)

        root_el.parentNode.insertBefore(c, root_el.nextSibling);
        root_el.remove()

        return progressiveMedia;
    }
}
```

# Conclusion

I wrapped the whole code in a portable library that I randomly and unoriginally called BluePil.js. It basically does everything described in this article, but with support for more browsers and with several additional features. Check out its [github page](https://github.com/tsucres/BluePil.js) and its [demo page](https://tsucres.github.io/BluePil.js/) for more info.


There are also several other libraries that do a similar job: 

- [pilpil.js](https://github.com/zafree/pilpil): lightweight, vanilla js, basically a simpler version of what is described in this article.
- [pil.js](https://github.com/gilbitron/Pil): very similar to pilpil.js, with simpler markup.
- [beLazy.js](http://dinbror.dk/blazy/): lightweight, highly customisable, vanilla js, but less straightforward to implement.
- [progressive-image.js](https://github.com/craigbuckler/progressive-image.js): lightweight, support for src-set, 
- jquery modules


For any questions/remark, please comment on the [medium page](...)



