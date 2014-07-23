---
title: Speedy Template Loading
date: 2014-07-22 21:53 UTC
tags: javascript,speed,templates,underscore,jquery
authors: JT Bowler
---

### Background

Like many web apps, Chalkschools makes use of some popular libraries like jQuery and Underscore to manage browser inconsistencies and common tasks.  Some of the most useful functionalities we have gained from these libraries are the abilities to easily manipulate DOM elements and to create templates for views.

Most of the pages on our site do not require too much "heavy lifting".  However, we do have a couple very JavaScript-heavy pages on our website - one of these being our document editor.  When working with some of the larger forms on our website, we started to notice problems that weren't present on the simpler forms.  Namely, when rendering tens (or hundreds) of templates on a form, the load time started to increase a noticeable amount.

### The Problem

DOM manipulations are always expensive.  That's just a byproduct of the fact that browsers need to maintain a large tree of elements to be rendered on a computer screen.  Even when you have direct access to a given element in memory, it is possible that the addition or removal of that element will cause most other elements to be positioned a bit differently (see [here](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/) for more information on how browsers rerender the screen after changes).

Take, for example, the following setup:

    var $container = $("#container");

    var $divs = [];
    for (var i = 0; i < 100; i++) {
        divs.push($("<div>"));
    }

---

If we tried to insert 100 of these divs into the page one at a time, the code would look like the following:


    $container.empty();

    for (var i = 0; i < 100; i++) {
        $container.append($divs[i]);
    }

---

Using this technique, one can insert perform about 615 operations per second (as measured on the MacBook pro here at the office).  It is not uncommon for some ChalkSchools forms to have 100 or more elements, each of which is instantiated as a JS object and attached various callbacks / CSS properties.  Thus, it can take several tenths of a second for the initial page load to happen.  But what if we batch insert the entire array at one?

    $container.empty();
    $container.append($divs);

Using this approach, I was able to achieve a rate of 4,277 operations per second (just under 7 times faster!).  Visit [here](http://jsperf.com/jquery-append-one-by-one-vs-bulk) to run the tests on your own machine.
