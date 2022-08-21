# [weird performance](#)

> Performance Regressions due to configurations, env, etc.


### Resources

[https://bugs.chromium.org/p/chromium/issues/list?q=label:Performance-Browser](https://bugs.chromium.org/p/chromium/issues/list?q=label:Performance-Browser)

[https://bugs.chromium.org/p/chromium/issues/list?q=label:Performance-Responsiveness](https://bugs.chromium.org/p/chromium/issues/list?q=label:Performance-Responsiveness)

[https://bugs.chromium.org/p/chromium/issues/list?q=label:Hotlist-Polish](https://bugs.chromium.org/p/chromium/issues/list?q=label:Hotlist-Polish)


## Disable double tap to zoom to improve FID

For background see [Issue 1108987: Disable double tap to zoom when width=devicewidth or initial_scale=1?](https://bugs.chromium.org/p/chromium/issues/detail?id%3D1108987&sa=D&usg=AFQjCNEbEVB7yYOkxIKuB9z6v9m2Vpf0SA)

Recommend fix [https://developers.google.com/web/updates/2013/12/300ms-tap-delay-gone-away](https://developers.google.com/web/updates/2013/12/300ms-tap-delay-gone-away)

~~~diff
 <html ⚡ lang="en">
   <head>
     <meta charset="utf-8">
-    <meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
+    <meta name="viewport" content="width=device-width">
~~~

### First Input Delay Changes in Chrome 91

Disable double-tap-to-zoom on mobile viewports
Double-tap-to-zoom (DTZ) is a gesture used to zoom into text. Previously, DTZ was disabled when either zooming was disabled (min-zoom equal to max-zoom) or when the content width fits the viewport width. After this change, we also disable DTZ when the viewport meta tag specifies width=device-width or initial-scale>=1.0, even when implicitly doing so, like for example in minimum-scale=1.5, maximum-scale=2.

Because DTZ negatively impacts FID and the amount of pages where DTZ is disabled is increased, we expect some sites to see better FID scores.

Relevant bug

> When were users affected?
> Chrome 91 is currently scheduled to be released the week of July 20, 2021.

To remove the 300-350ms tap delay, all you need is the following in the <head> of your page:

```
<meta name="viewport" content="width=device-width">
```
This sets the viewport width to the same as the device, and is generally a best-practice for mobile-optimized sites. With this tag, browsers assume you've made text readable on mobile, and the double-tap-to-zoom feature is dropped in favour of faster clicks.

If for some reason you cannot make this change, you can use touch-action: manipulation to achieve the same effect either across the page or on particular elements:
 
```css
html {
    touch-action: manipulation;
}
```

 
## Style calculation takes much longer for multiple `<style>`s vs one big `<style>`
 
> [https://bugs.chromium.org/p/chromium/issues/detail?id=1337599&q=component%3ABlink%3ECSS](https://bugs.chromium.org/p/chromium/issues/detail?id=1337599&q=component%3ABlink%3ECSS)
 
Chrome seems to take much longer to calculate style for a DOM with multiple `<style>`s in the `<head>` than when those same stylesheets are concatenated into one big `<style>`.

For Firefox and Safari, there is no equivalent performance cliff. These browsers show essentially no difference between multiple `<style>`s versus one big `<style>`.

Here are the numbers for Chrome, Firefox, and Safari, all taken on the same machine (2021 Mac Book Pro), taking the median of 5 runs (milliseconds):

| Browser     | One big style | Multiple styles |
|-------------|---------------|-----------------|
| Chrome 102  | 28            | 1354            |
| Firefox 98  | 38            | 30              |
| Safari 15.5 | 827           | 855             |


 #### Solution
 
The best practice from Chromium's POV would be to concatenate stylesheets wherever possible, but to avoid doing so if any of those stylesheets will be individually modified in the future (or if route-splitting makes it more advantageous to cache them separately).
 
 ##  Slow redrawing of the window content and severe degradation after loading the content on the site
 
 >  Cause by long style recalc.

Potentially associated animation is "max-height". When the page gets very long, animating max-height requires re-layout and possibly repaint, and that might take time.

 
 ## Cumulative layout shift Adjustments
 
 #### 1. Lazy load images
```html
<img loading="lazy" />[^3]
```
- Prevents downloading/processing off-screen images

 #### 2. Compress to lossy webp (still looks great)
```console
 /repo/dir :~ $ cwebp input.png -q 90 -alpha_q 100 -m 6 -o out.webp
```
- Decreases the size of every image by many multiples: 400kb to 50-120kb.

 #### 3. Remove CLS
img { aspect-ratio: 1.4 }
- Keeps the images from causing jank when they download. Even without the image, the CSS knows the width and height to draw the box.
 
> [source, https://twitter.com/ryanflorence/status/1560274356579622914](https://twitter.com/ryanflorence/status/1560274356579622914)
 
#### Solution
 
CLS, or Cumulative layout shift, is i.e. how much your page jumps around as things load in. By setting the aspect ratio upfront the placeholder is presumably going to fill the right size so no jump when rendered in.
 
>**Note**     
> The question is very similar to "should this image be a <img> or a CSS background-image?" and the answer is the same: Is it content or design?

[^3]: [see https://jakearchibald.com/2022/img-aspect-ratio/](https://jakearchibald.com/2022/img-aspect-ratio/)
 
 ```css
 img {
  aspect-ratio: 4 / 3;
  width: 100%;
  object-fit: cover;
}
```

#### `auto` and `16 / 9;`
If both auto and a <ratio> are specified together, the preferred aspect ratio is the specified ratio of width / height unless it is a replaced element with a natural aspect ratio, in which case that aspect ratio is used instead.
```css
 .aspect-ratio-demo {
  aspect-ratio: auto 16 / 9;
}
```
 

