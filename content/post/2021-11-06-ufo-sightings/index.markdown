---
title: UFO Sightings
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
---

![screen reader text](believe.jpg)


## Data Analysis Project - UFO sightings

I'm working in this projects with Kristy and Adel. Both are coleagues from the Google Data Analytics Certificate. The idea of working together in a data project came from Kristy.

## Introduction

There are few things that are shared with all cultures and in all different time periods as the human fascination with the sky. Some look to the sky as a guide for their actions, and other as a study object. But many look above with one question in head **are we alone in the universe?**.

Within the group that answer No to the previous question, there is a subgroup that believes not only that there is extraterrestrial life, but also that we are regularly visited by the beings from outer space.

Until we make contact will be hard to determine if we are visited or not, but one *possible* evidence is UFOs. UFOs are Unidentified flying objects. Many of them can be explained, but some remain mysterious, such as the [Tic Tac](https://www.youtube.com/watch?v=bJj9sS6adWs). In this project we will analyze UFO sighting data.


### The data

The dataset is available at [maven analytics](https://www.mavenanalytics.io/data-playground?page=3&pageSize=5). It contains data for UFO sightings from 1949 to 2014, mainly in the USA. 

This is first and foremost an exploratory project. We do not have a hypothesis that we want to test. We want to look in the data and see if there is any interesting pattern.


## Analysis

#### Loading necessay packages


```r
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --
```

```
## v ggplot2 3.3.5     v purrr   0.3.4
## v tibble  3.1.5     v dplyr   1.0.7
## v tidyr   1.1.3     v stringr 1.4.0
## v readr   2.0.0     v forcats 0.5.1
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


```r
ufo_sightings <- read_csv("C:/Users/tiago/Downloads/ufo_sightings_scrubbed.csv (1)/ufo_sightings.csv")
```

```
## Rows: 80332 Columns: 11
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr  (6): city, state, country, shape, duration (hours/min), comments
## dbl  (3): duration (seconds), latitude, longitude
## dttm (1): datetime
## date (1): date posted
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
ufo_sightings <-  clean_names(ufo_sightings)
glimpse(ufo_sightings)
```

```
## Rows: 80,332
## Columns: 11
## $ datetime           <dttm> 1949-10-10 20:30:00, 1949-10-10 21:00:00, 1955-10-~
## $ city               <chr> "san marcos", "lackland afb", "chester (uk/england)~
## $ state              <chr> "tx", "tx", NA, "tx", "hi", "tn", NA, "ct", "al", "~
## $ country            <chr> "us", NA, "gb", "us", "us", "us", "gb", "us", "us",~
## $ shape              <chr> "cylinder", "light", "circle", "circle", "light", "~
## $ duration_seconds   <dbl> 2700, 7200, 20, 20, 900, 300, 180, 1200, 180, 120, ~
## $ duration_hours_min <chr> "45 minutes", "1-2 hrs", "20 seconds", "1/2 hour", ~
## $ comments           <chr> "This event took place in early fall around 1949-50~
## $ date_posted        <date> 2004-04-27, 2005-12-16, 2008-01-21, 2004-01-17, 20~
## $ latitude           <dbl> 29.88306, 29.38421, 53.20000, 28.97833, 21.41806, 3~
## $ longitude          <dbl> -97.941111, -98.581082, -2.916667, -96.645833, -157~
```
The first thing I want to see is the evolution of the number of sighting per year. For doing this I will have to extract the year from the datetime column.


```r
ufo_sightings <- ufo_sightings %>% 
    mutate(year = year(datetime)) # Use the year function to extract the year information

glimpse(ufo_sightings)
```

```
## Rows: 80,332
## Columns: 12
## $ datetime           <dttm> 1949-10-10 20:30:00, 1949-10-10 21:00:00, 1955-10-~
## $ city               <chr> "san marcos", "lackland afb", "chester (uk/england)~
## $ state              <chr> "tx", "tx", NA, "tx", "hi", "tn", NA, "ct", "al", "~
## $ country            <chr> "us", NA, "gb", "us", "us", "us", "gb", "us", "us",~
## $ shape              <chr> "cylinder", "light", "circle", "circle", "light", "~
## $ duration_seconds   <dbl> 2700, 7200, 20, 20, 900, 300, 180, 1200, 180, 120, ~
## $ duration_hours_min <chr> "45 minutes", "1-2 hrs", "20 seconds", "1/2 hour", ~
## $ comments           <chr> "This event took place in early fall around 1949-50~
## $ date_posted        <date> 2004-04-27, 2005-12-16, 2008-01-21, 2004-01-17, 20~
## $ latitude           <dbl> 29.88306, 29.38421, 53.20000, 28.97833, 21.41806, 3~
## $ longitude          <dbl> -97.941111, -98.581082, -2.916667, -96.645833, -157~
## $ year               <dbl> 1949, 1949, 1955, 1956, 1960, 1961, 1965, 1965, 196~
```

```r
ufo_sightings %>% 
    arrange(datetime) %>%
    group_by(year) %>% 
    summarise(n_sightings = n()) %>% 
    ggplot(aes(x = year, y = n_sightings)) +
    geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Sightings vs year-1.png" width="672" />
This raises a question. Has sightings really gone up in the last 20 years or we just have a problem of data collection? We can really answer with the available data, but we could get a look at the date posted column. In particular, I want to know how many recordings were posted by year, and since when they are posting. If this is concentrated in recent years, we may have evidence that maybe the recent interest in recording the data explain the the jump in the numbers of sightings recorded.




```r
ufo_sightings <- ufo_sightings %>% 
    mutate(year_posted = year(date_posted)) # Use the year function to extract the year information
```



```r
ufo_sightings %>% 
    arrange(datetime) %>%
    group_by(year_posted) %>% 
    summarise(n_posted = n()) %>% 
    ggplot(aes(x = year_posted, y = n_posted)) +
    geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Year posted-1.png" width="672" />
So, we can see that the recording began in the 1990's and this is when we see a jump in the the number of cases. This may lead to two explanations.

First, the fact that we began recording all the sightings automatically lead to more cases being recorded. So, it's possible that there were many sightings before, that went un-recorded.

A second explanation is that a third factor lead to more sightings and also created enough interest in sightings that lead to a more systematic recording of events. With this data, we cannot answer. But these are possibilities that should be further investigated.

____

As the numbers of sightings become too big from the 1990's onward, it becomes difficult to see the variation that happened before.

Just to see how thing were, I will create a smaller dataset, with data up to 1995.



```r
ufo_sightings %>% 
    arrange(datetime) %>%
    group_by(year) %>% 
    filter(year <= 1995 & year >= 1940) %>% 
    summarise(n_sightings = n()) %>% 
    ggplot(aes(x = year, y = n_sightings)) +
    geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Before Independence day-1.png" width="672" />
Now we can see how much the sightings suffered variation until the 1990's, when it really changes the behavior. This would say that there is something that change in the 1990's. Maybe the ETs discovered our adress.

### What are the most commonly seen shapes

When I thing in UFOs, I think about a disc. But was it always like that? I don't know. Let's let the data tell us.



```r
n_distinct(ufo_sightings$shape)
```

```
## [1] 30
```
Oj, there are 30 different shapes. Let's see how they are divided between categories.



```r
ufo_sightings %>% 
  group_by(shape) %>% 
  summarise(n())
```

```
## # A tibble: 30 x 2
##    shape    `n()`
##    <chr>    <int>
##  1 changed      1
##  2 changing  1962
##  3 chevron    952
##  4 cigar     2057
##  5 circle    7608
##  6 cone       316
##  7 crescent     2
##  8 cross      233
##  9 cylinder  1283
## 10 delta        7
## # ... with 20 more rows
```
There are many different formats. But there are many with few observation. So I will drop any with lest that 10 views. Those guys are just no regular visitors. There is also the groups of unknown, NAs, and others. These I will combine as "unknown" because I have no idea between which people were selection, for pick another.


```r
ufo_sightings$shape[is.na(ufo_sightings$shape)] <- "unknown"
ufo_sightings$shape[ufo_sightings$shape == "other"] <- "unknown"

# Circles and disk are two closely reated shapes. I will recode them as circular shaped objects
ufo_sightings$shape[ufo_sightings$shape == "disk"] <- "circular"
ufo_sightings$shape[ufo_sightings$shape == "circle"] <- "circular"

# We also have oval and egg

ufo_sightings$shape[ufo_sightings$shape == "egg"] <- "oval"

#

ufo_sightings$shape[ufo_sightings$shape == "cigar"] <- "cylinder"

ufo_sightings %>% 
  group_by(shape) %>% 
  summarise(n())
```

```
## # A tibble: 25 x 2
##    shape    `n()`
##    <chr>    <int>
##  1 changed      1
##  2 changing  1962
##  3 chevron    952
##  4 circular 12821
##  5 cone       316
##  6 crescent     2
##  7 cross      233
##  8 cylinder  3340
##  9 delta        7
## 10 diamond   1178
## # ... with 15 more rows
```
now, let's drop those types with few appearances. We only want regular visitors.



```r
shapes_to_drop <- ufo_sightings %>% 
  group_by(shape) %>% 
  summarise(views = n()) %>% 
  filter(views <= 10) %>% 
  select(shape)

shapes_to_drop
```

```
## # A tibble: 8 x 1
##   shape   
##   <chr>   
## 1 changed 
## 2 crescent
## 3 delta   
## 4 dome    
## 5 flare   
## 6 hexagon 
## 7 pyramid 
## 8 round
```

```r
# This code filter all values in shape, that are different from those we #declared
ufo_2 <- ufo_sightings %>% 
  filter(!shape %in% c("changed", "crescent", "delta", "dome", "flare",
                      "hexagon", "pyramid", "round"))
```


```r
ufo_2 %>% 
  group_by(shape) %>% 
  summarise(n = n()) %>% 
  ggplot(aes(reorder(shape, -n), n, fill =shape)) + #The reoder function reorder the x axis, by using the n variable. The - implies descending order
  geom_col() +
  theme(axis.text.x = element_text(angle = 90))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Checking the results-1.png" width="672" />

Many different shapes were reported. But, if you ignore the unknown type, 


```r
top_8 <- c("light", "unknown", "circular", "triangle", "fireball",
           "sphere", "oval","cylinder")

ufo_top_8 <- ufo_2 %>% 
    filter(shape %in% top_8)


ufo_top_8 %>% 
  group_by(shape) %>% 
  summarise(n = n()) %>% 
  ggplot(aes(reorder(shape, -n), n, fill =shape)) + #The reoder function reorder the x axis, by using the n variable. The - implies descending order
  geom_col() +
  theme(axis.text.x = element_text(angle = 90))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/top 7 sightings shapes-1.png" width="672" />
___ 

Let's analyze now in what moment the sightings are happening..
I would say that they are probably happenning at night, but let's check.



```r
ufo_2 <- ufo_2 %>% 
    mutate(datetime = round_date(datetime, "hour")) %>% 
    mutate(sighting_time = hms::: hms(second(datetime), minute(datetime), hour(datetime)))

str(ufo_2)
```

```
## spec_tbl_df [80,316 x 14] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ datetime          : POSIXct[1:80316], format: "1949-10-10 21:00:00" "1949-10-10 21:00:00" ...
##  $ city              : chr [1:80316] "san marcos" "lackland afb" "chester (uk/england)" "edna" ...
##  $ state             : chr [1:80316] "tx" "tx" NA "tx" ...
##  $ country           : chr [1:80316] "us" NA "gb" "us" ...
##  $ shape             : chr [1:80316] "cylinder" "light" "circular" "circular" ...
##  $ duration_seconds  : num [1:80316] 2700 7200 20 20 900 300 180 1200 180 120 ...
##  $ duration_hours_min: chr [1:80316] "45 minutes" "1-2 hrs" "20 seconds" "1/2 hour" ...
##  $ comments          : chr [1:80316] "This event took place in early fall around 1949-50. It occurred after a Boy Scout meeting in the Baptist Church"| __truncated__ "1949 Lackland AFB&#44 TX.  Lights racing across the sky &amp; making 90 degree turns on a dime." "Green/Orange circular disc over Chester&#44 England" "My older brother and twin sister were leaving the only Edna theater at about 9 PM&#44...we had our bikes and I "| __truncated__ ...
##  $ date_posted       : Date[1:80316], format: "2004-04-27" "2005-12-16" ...
##  $ latitude          : num [1:80316] 29.9 29.4 53.2 29 21.4 ...
##  $ longitude         : num [1:80316] -97.94 -98.58 -2.92 -96.65 -157.8 ...
##  $ year              : num [1:80316] 1949 1949 1955 1956 1960 ...
##  $ year_posted       : num [1:80316] 2004 2005 2008 2004 2004 ...
##  $ sighting_time     : 'hms' num [1:80316] 21:00:00 21:00:00 17:00:00 21:00:00 ...
##   ..- attr(*, "units")= chr "secs"
##  - attr(*, "spec")=
##   .. cols(
##   ..   datetime = col_datetime(format = ""),
##   ..   city = col_character(),
##   ..   state = col_character(),
##   ..   country = col_character(),
##   ..   shape = col_character(),
##   ..   `duration (seconds)` = col_double(),
##   ..   `duration (hours/min)` = col_character(),
##   ..   comments = col_character(),
##   ..   `date posted` = col_date(format = ""),
##   ..   latitude = col_double(),
##   ..   longitude = col_double()
##   .. )
##  - attr(*, "problems")=<externalptr>
```
Now, let's make a plot


```r
ufo_2 %>% 
    group_by(sighting_time) %>% 
    summarise(n_sightings =n()) %>% 
    ggplot() +
    geom_step(aes(x = sighting_time, y = n_sightings), color = "blue") +
    theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />
While this plot is not the most beautiful in the world, it does make it clear that there are more sightings at night, peaking after 08:00 p.m. and before 01:00 a.m.

### Local of sightings

First, let's check from which countries these sightings are coming from.


```r
table(ufo_2$country)
```

```
## 
##    au    ca    de    gb    us 
##   538  2999   105  1905 65101
```
As we can see, most of the data comes from the US.
We also have a column called state. I believe that this column inputs the data form sighting in the US. 

I will create a US data set, and then use it to investigate.


```r
usa_ufo <- ufo_2 %>% 
    filter(country == "us")

table(usa_ufo$country)
```

```
## 
##    us 
## 65101
```
As you can see, now we have only the us data. Let's see which state has more sightings recorded.

### Which american state is the favorite destination for UFOs?

First, let's see which states have more recorded UFO sightings.


```r
usa_ufo %>% 
    group_by(state) %>% 
    summarise(n_sightings = n()) %>% 
    ggplot(aes(x= reorder(state, n_sightings), n_sightings)) +
    geom_col() +
    coord_flip() +
    geom_text(aes(label = state), hjust = -0.5)+
    theme_light()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" />
While we can see that California where there are more sighting recorded, this is just not a beautiful plot. There are too many columns. So, let's focus on the states that have more sightings than the median.


```r
usa_ufo %>% 
    group_by(state) %>% 
    summarise(n_sightings = n()) %>% 
    filter(n_sightings >= median(n_sightings)) %>%
    ggplot(aes(x= reorder(state, n_sightings), n_sightings)) +
    geom_col() +
    coord_flip() +
    geom_text(aes(label = state), hjust = -0.5)+
    theme_light()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />

In raw numbers it's clear that most of the sightings are happening in California. The problem with this is that California is the most populated state in the U.S., therefore it's expected that there will be more recordings there. 
To account for this fact, we need data on state population. What is not available in our current dataset.

So, I will add [US Census data](https://www.census.gov/programs-surveys/popest/technical-documentation/research/evaluation-estimates/2020-evaluation-estimates/2010s-state-total.html) so we can consider this. I will also need a support data set with the link between state names and codes. This data is available [here](https://worldpopulationreview.com/states/state-abbreviations).



```r
state_names <- read_csv("C:/Users/tiago/OneDrive/datasets/state_names.csv")
```

```
## Rows: 51 Columns: 3
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr (3): State, Abbrev, Code
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
states_population <- read_csv("C:/Users/tiago/OneDrive/datasets/states_populations.csv")
```

```
## Rows: 57 Columns: 151
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr   (5): SUMLEV, REGION, DIVISION, STATE, NAME
## dbl (146): CENSUS2010POP, ESTIMATESBASE2010, POPESTIMATE2010, POPESTIMATE201...
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
Now, we will work a bit with these datasets, keeping just the columns that we need.


```r
states <- state_names %>% 
    select(State, Code) %>% 
    clean_names()

population <- states_population %>% 
    filter(STATE != "00") %>%               #drop data not related to states
    select(STATE, NAME, CENSUS2010POP) %>% 
    clean_names()                        # select the 2010 Census population

states_pop <- left_join(states, population, by = c("state" = "name")) %>% 
    select(code, census2010pop)
states_pop$code <- tolower(states_pop$code)
```

Now, I have the state population in 2010. I can use it to create a dataset with sightings per 100.000 inhabitants.

* Combine the population data with a summarized dataset
* Create a sightings rate


```r
usa_ufo %>% 
    group_by(state) %>% 
    summarise(n_sightings = n()) %>% 
    left_join(states_pop, by = c("state" = "code")) %>% 
    mutate(sightings_rate = (n_sightings/census2010pop)*100000) %>% 
    filter(n_sightings > 500) %>% 
    ggplot(aes(reorder(state, sightings_rate), sightings_rate)) +
    geom_col() +
    coord_flip() +
    geom_text(aes(label = state), hjust = -0.5)+
    theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Sightings per 100.000 people-1.png" width="672" />

Whe we consider the state populations we see that the fact that California, Texas, and Florida are the states with more sightings are driven mainly due to population size. 
The state that are really being visited a lot, is Washington. **Evidence that the americans are being governed by reptilians.**

## Do movies affect UFO sightings?

