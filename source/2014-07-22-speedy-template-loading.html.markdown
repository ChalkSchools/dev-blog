---
title: Speedy Template Loading
date: 2014-07-22 21:53 UTC
tags: javascript,speed,templates,underscore,jquery
authors: JT Bowler
---

**Background**

Like many other web applications, Chalkschools makes use of some popular libraries like jQuery and Underscore to manage browser inconsistencies and common tasks.  Some of the most useful functionalities we gain from these libraries are the abilities to easily manipulate DOM elements and to create templates for our views.  While we've got plenty of JavaScript powering our site, most of our pages do not require extensive JavaScript in order to function properly.

However, we have a couple very JavaScript-heavy pages on our website - one of these being our document editor.  When working with some of the larger forms on our website, we started to notice problems that weren't present on the simpler forms.  Namely, when rendering tens (or hundreds) of templates on a form, the load time started to increase a noticeable amount.

**The Problem**

DOM manipulations are always expensive.  That's just a byproduct of the fact that browsers need to maintain a large tree of elements to be rendered on a computer screen.  Even when you have direct access to a given element in memory, it is possible that the addition or removal of that element will cause most other elements to be positioned a bit differently.

Take, for example, the following line:

```$(document).prepend('<div></div>');```

This is a pretty simple command, but the browser must update the DOM tree and then rerender the page.  This probably won't take a whole lot of time since it is just a single prepend command... but imagine that instead of one div, you have 300 to load into the page.

Time to load 1:     0.062ms

Time to load 300:   23.584ms

You can see that this is now actually having an impact on the user experience.  How can you get around that?  The answer is actually pretty simple - we can create a jQuery object in memory and then append our new DIV elements to that jQuery object.  Working with elements in memory is orders of magnitude faster than doing DOM manipulations.  Once we have appended all the DIVs in memory, we can perform the most expensive operation last: adding our container object into the DOM.

[Time to load 1]
[Time to load 300 from the container]

This is why it takes roughly the same amount of time to load one field on our document editor as it does to load 300.

[more examples needed]
