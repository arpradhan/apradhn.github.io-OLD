---
layout: post
title: "Let's Talk About Flex: Building UI's with CSS Flexboxes"
date: 2015-09-09 15:42:39 -0400
comments: true
categories: 
---
CSS has evolved much since its early days on the web, but until recently centering elements on a page has proven an absurdly difficult task. It's the earliest hurdle you face as a web designer, and until flex boxes there hasn't been a major consensus on how to center elements. There are thousands of articles on the internet describing thousands of different ways to center elements. And if you want to center an element vertically? Forget about it. Theses guides usually involve some complex process of combining floats, calculating margin differences, line heights, etc. Line heights? Seriously? You should not need to alter an elements line height just to get a thing centered in a box. 

Thankfully, Flexboxes allow you to center elements easily, rendering all those guides obsolete. Additionally, flexboxes allow the web designer to design a user interface elements for an application that is more similar to what a user expects from a smartphone application. I'm excited for flexboxes to become more mainstream because it will move the quality of web design that much closer to print.

Flexbox is a newer CSS element, but the majority of browsers have implemented the flexbox specification. Only Safari and Safari iOS require you to prefix the CSS declaration with ```-webkit-```. You can learn more about the status of Flexbox's implementation in browsers at [caniuse.com](http://caniuse.com/#feat=flexbox).

What makes Flexbox so magical? At the heart of the Flexbox specification is an algorithim that automatically calculates the appropriate margins 

As an example, lets design the AKAI MPC 2000 using flexboxes: 
![AIKI MPC 2000](http://medias.audiofanzine.com/images/normal/akai-mpc2000-618501.jpg)