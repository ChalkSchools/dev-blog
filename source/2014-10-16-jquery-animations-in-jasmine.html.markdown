---
title: jQuery Animations in Jasmine
date: 2014-10-16
tags: jquery, jasmine, annoyances
authors: Rob Fletcher, JT Bowler
---

## Annoyance While Testing in Jasmine

For our front end tests, we use the Jasmine framework.  Some of our components
on the front end use various jQuery animations.  These animations are triggered
within the browser while running Jasmine tests, causing some annoying behavior.
In particular, when the tests finish running, we are not able to scroll down
until all the animations are finished.READMORE  See exhibit A below:

![Bad Scrolling](images/bad_scrolling.gif "Bad Scrolling")

For our fix, all we had to do is add the following line to spec_helper.js:

```js
$.fx.off = true;
```
<br />
And we were greeted with scrolling bliss.  See exhibit B:

![Smooth Scrolling](images/smooth_scrolling.gif "Smooth Scrolling")

That's all.  Goodbye and good luck.
