# [performance3](#)


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
