---
created_at: 2012-01-24 17:10:00
timestamp: 1318349999
excerpt: "Using class names to find elements with JavaScript leads to confusion and maintenance issues. For true separation of styling and behavior you should use custom data attributes instead, with added benefits."
kind: article
publish: true
disqus: true
title: "Don't use class names to find HTML elements with JS"
---

In the days before HTML5, when we wanted to bind JavaScript events to an element, we would find the element based on its class name (or the `rel` attribute). After all, using a class name won't invalidate your HTML, is easy to target (specially with a library such as jQuery) and provides a hook for styling. But those days are over with the introduction of better suitable techniques, and using class names to find elements leads to confusion and maintenance issues. For true separation of styling and behavior you should use custom data attributes instead, with added benefits.

## It's not "wrong"

In the sense of "not being according the specifications" it isn't wrong (when using HTML5). In the HTML4 specification the W3C is clear on the intended use of the `class` attribute, saying it should be used "as a style sheet selector (when an author wishes to assign style information to a set of elements)." The HTML5 specification on the other hand, says: "Assigning classes to an element affects class matching in selectors in CSS, the `getElementsByClassName()` method in the DOM, and other such features." So when writing HTML5, you're creating completely valid code.

## Our example

Imagine a situation where you want to apply some interaction to a list of people in various age groups. You have groups of 0 to 9 and 10 to 19 year olds. In your HTML you use `class="0-9"` and `class="10-19"`. Now you can find all elements belonging to the group of 0 to 9 year olds with `$('.0-9')` (jQuery syntax), so you can bind an `onclick` event to them. After a while you decide it's fun to style all children with a playful icon, so in your CSS you add a background image for any elements having the `.0-9` class (I know that won't work, as it isn't valid CSS, it's just for illustrative purposes). You're doing test driven development, so you make sure to have a test that validates that the JavaScript interaction on the element behaves as intended. Works like a charm!

## The problem

After a while you're replaced or a new developer is added to the team, and the project owner says he would like that little icon we show for small kids to be visible for everyone up to 12 years of ago. Piece of cake for the new guy: just rename the class name `0-9` to the more generic `kid` in the CSS and make sure the HTML reflects all children up to 12 use this new class name. From all other elements the `10-19` class can be removed; it's inaccurate and not used anyway. Your new guy or girl reloads the page and sees it's all good.

If you're not doing any JavaScript testing, you're already screwed at this point, since there's a fair chance that this change will go into production unnoticed. Looking great, but without the JavaScript events, because there are no elements to find when looking for `$('.0-9')`. Sure you could expect your developers to hit every single element after making a trivial change, but that would be too expensive (and you know they won't anyway).

Bonus points for you if you run tests against your JavaScript; the above should definitely trigger an error. Now you're tasked with fixing your broken code. Changing your JavaScript to use `$('.kid')` doesn't work, as it finds more than just children up to 9 years old. Time to add a second class name, reinstating the previous structure. Congratulations: you've spent over an hour on essentially _adding_ `kid` as a second class name to some elements.

Now imagine that in the starting situation you want to target everyone up to age 12 from your JavaScript. The simple fix is to simply change `0-9` to `0-12` in both your HTML and JavaScript and you're done. Unless you're doing rigorous manual testing, you won't spot what just broke. There's no automated tool out there that will fail on "show playful background image on all children up to 9 years old", which is what will happen when not changing the class name in your CSS as well.

## Time to _really_ start separating structure, styling & behavior

You're doing a good job by binding events in your JavaScript instead of using nasty `onclick` attributes in your HTML; separation of structure and behavior is important for various reasons. Of course your HTML and CSS need to be linked in some form, as do your HTML and JS. But can targeting elements based on their class name really be considered separation between styling and behavior? You're tying all three layers together, leading to the issues outlined above.

## Use custom data attributes instead of class names

HTML5 brings the joy of being able to use custom data attributes. I suggest using those exclusively to tie HTML and JavaScript together, leaving class names for styling with CSS. It's a simple solution to a common problem and doesn't require more work than the previous approach. We could just use our `kid` class name for the background image and add `data-age-group="0-12"` (or something more generic) to the elements we want to target in JavaScript. jQuery doesn't have any problems with finding all elements in a certain age group with `$('[data-age-group="0-12"]')`, or if you want any element with a certain data attribute regardless of value, `$('[data-age-group]')` (this works in IE6 when using jQuery or the `getAttribute()` method).

## Key/value pairs have more semantic meaning

You'll gain more semantic meaning when using data attributes. Where "0-9" could mean basically anything, "age-group" is quite clear on what its value is about. It enables you to add a similar metric later, without refactoring. If you want to add the number of pets someone owns you can just add `data-pets="0-9"`. When using class names, you would have to refactor your `0-9` to something like `age-group-0-9` first, change it in your JavaScript and then add `pets-0-9` as a second class name.

## Data attributes enable a modular approach

When writing a piece of JavaScript that will be reused throughout various of your projects you want something that just works, regardless of styling. Say you created a "plugin" that takes a regular unordered list and displays all items in the list _n_ at a time with some fading effects. When not relying on class names for styling you can just drop this in any existing code, for instance by simple adding `data-carousel-items="2"` to your `<ul>`. Your script can find all elements with that attribute and use their values to determine how many items to show at a time, doubling as a config option. It doesn't require any change in the HTML apart from adding that attribute and the CSS can remain untouched.

Let's make this a best practice.

## Update: It's slower, and that doesn't bother me

As discussed in the comments, looking up elements by data attribute in jQuery is slower than by class name. In [this jsPerf test](http://jsperf.com/long-selectors-vs-data/2) the `$('.class')` operation can be performed ± 124,000 times per second. The `$('data-foo="bar"'])` operation only ± 26,000 times (tested on Chrome). Yes, it's over 70% slower. No, it isn't slow. Imagine there are 2,500 elements to search through; it will take under 100 _milliseconds_. IE sucks of course, with IE9 handling only just over 7,000 operations/second (sorry for the wait). It works for me.
