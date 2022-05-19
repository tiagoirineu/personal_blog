---
title: Day 5 - Themes
author: Tiago Irineu
date: '2021-11-30'
slug: day-5-themes
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-11-30T06:18:27-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

### Themes - Day 5

#### Reflection

CRAP is the principle design that we've learned so far. It is an acronym for Contrast, Repetition, Alignment, and Proximity. It guide us in how to build images, considering the human perception. 

CRAP principles should also be considered when designing plots, given that one of the aspects we must consider when creating data visualizations is how the user will interpret the elements on it. For example, proximity is important because a legend must be near the element it is labeling. Repetition can be important when you are writing a piece with multiple plots on it, and therefore you can repeat the main design elements in all the plots, as colors and fonts. 

Contrast is one of the most important considerations when building a plot. We want that the relevant information is emphasized. We want to use graphical elements that do not distract the reader from the real message. Alignment also matters. Title, subtitles, and any text element must be aligned in a sensible and pleasant way. 

The easiest way to guarantee that you are applying the principles of CRAP in a work is to use themes. In ggplot themes it's possible to adjust all the elements in a plot. If we use the same theme throughout a piece, we can guarantee that we are applying the CRAP principles in every plot.

#### Should we use gridlines in our plots?

Considering the points raised by [Naomi Robbins](https://www.forbes.com/sites/naomirobbins/2012/02/22/are-grid-lines-useful-or-chartjunk/?sh=6541e0d54283) and [Stephen Few](http://www.perceptualedge.com/articles/dmreview/grid_lines.pdf), I would say that gridlines can be useful and should be used in some situations.

Stephen and Naomi give some examples of when gridlines are useful, but for me it is not key to have a list of when you should ou should not use gridlines. Whant really matters is that when you use gridlines, you must remember that it is a support element for the plot. Therefore, it should have muted colors, that let the elements that are linked to data be the most relevant in your plot.
