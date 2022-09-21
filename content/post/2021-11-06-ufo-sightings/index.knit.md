---
title: Data analysis project - UFO Sightings
author: Tiago Irineu
date: '2021-11-06'
slug: ufo-sightings
categories: []
tags: [R]
subtitle: ''
summary: 'In this project we analyze the UFO sightings dataset'
authors: []
lastmod: '2021-11-06T06:56:30-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []

output:
  blogdown::html_page:
    toc: true

---

![screen reader text](believe.jpg)




## Introduction

There are few things that are shared with all cultures and in all different time periods as the human fascination with the sky. Some look to the sky as a guide for their actions, and other as a study object. But many look above with one question in head **are we alone in the universe?**.

Within the group that answer No to the previous question, there is a subgroup that believes not only that there is extraterrestrial life, but also that we are regularly visited by the beings from outer space.

Until we make contact will be hard to determine if we are visited or not, but one *possible* evidence is **UFOs**. UFOs are Unidentified Flying Objects. While many of them can be easily explained, some remain mysterious. Take for example the [Tic Tac](https://www.youtube.com/watch?v=bJj9sS6adWs). 

In this project we analyze UFO sightings data for getting a better idea of the characteristics of these views.


### The data

The dataset is available at [maven analytics](https://www.mavenanalytics.io/data-playground?page=3&pageSize=5). It contains data for UFO sightings from 1949 to 2014, mainly in the USA. 

This is first and foremost an exploratory project. We do not have a hypothesis that we want to test. We want to look in the data and see if there is any interesting pattern.


## Analysis

My analysis tool is R.
I will share all the code, so if anybody is interest can replicate the results.

#### Loading necessay packages


```r
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --
```

```
## v ggplot2 3.3.5.9000     v purrr   0.3.4     
## v tibble  3.1.5          v dplyr   1.0.7     
## v tidyr   1.1.3          v stringr 1.4.0     
## v readr   2.0.0          v forcats 0.5.1
```

```
## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(janitor)
```

```
## 
## Attaching package: 'janitor'
```

```
## The following objects are masked from 'package:stats':
## 
##     chisq.test, fisher.test
```

```r
library(lubridate) #for working with dates
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```











































