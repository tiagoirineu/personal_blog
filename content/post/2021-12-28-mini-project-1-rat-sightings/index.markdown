---
title: 'Mini Project 1: Rat sightings'
author: Tiago Irineu
date: '2021-12-28'
slug: mini-project-1-rat-sightings
categories: []
tags: []
subtitle: ''
summary: 'In this project I analyze the data regarding rat sightins in New York. The data range is from 2015 to 2017'
authors: []
lastmod: '2021-12-28T21:05:01-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

### Rats in New York

This is the documentation for my final plot.

Instead of using the basic script shared by the teacher, I chose to work from a clean scrip. So I could at least think a bit for myself.

First, I will load the data and apply the function *clean_names* from the **janitor** package. This function transform all column names to snake_case.


```r
# Loading packages

library(tidyverse)
library(scales)
library(lubridate)
library(ggthemes)

# Reading the data

rats <-  read_csv("C:/Users/tiago/OneDrive/datasets/data_viz-course/Rat_Sightings.csv")

rats <- janitor::clean_names(rats)
```
Now, I have to prepare the data for creating the plot.

The main work is with the *created_date* column. A brief work on it and we can extract date from it.

R is a bit dumb, so I have to use the function *mdy_hms()* to tell R the format of the characters in the relevant column.





```r
rats_date <- rats %>% 
    mutate(created_date = mdy_hms(created_date),
           year = year(created_date),
           sighting_weekday = wday(created_date, label = TRUE, abbr = FALSE)) %>% 
    group_by(year, borough) %>% 
    summarise( n_sightings = n()) %>% 
    filter(n_sightings > 1)
```

Below, is the my final plot.

I have experimented with different visualizations, but mostly weren't informative.

In the graph below, we see the evolution of rats sightings through the year and divided by borough.




```r
rats_date %>% 
    ggplot() +
    geom_point(aes(x = year, y = n_sightings, color = borough), size = 2) +
    geom_line(aes(x = year, y = n_sightings, color = borough), size = 1.5, alpha = 0.8) +
    xlim(2010, 2018) +
    
    scale_color_viridis_d( option = "B") +
    
    # Adding labels for borough in the end of the line
    geom_text(
        data = rats_date %>% filter( year == 2017),
        aes(label =borough, x = year, y = n_sightings, color = borough),
             nudge_x = 0.5, alpha = 1, size = 3, angle = -15, fontface = "bold"
    ) +
    
    labs(title = "Rats sightings have increased in NY",
         subtitle = "Sights per borough: 2010 to 2017",
         x = "", y = "") +
    
    guides(color = "none") +
    theme_minimal() +
    theme(panel.grid = element_blank(),
          plot.margin = margin(t = 5, r= 5, b = 5, l =2),
          axis.text.y   = element_text(family = "Source Sans Pro", color = "gray24"),
          axis.text.x = element_text(family = "Source Sans Pro", color = "gray24"),
          axis.title.y =  element_blank(), #element_text(family = "Source Sans Pro", face = "bold", color = "gray24"),
          axis.title.x = element_blank(),
          plot.title = element_text(family = "AppleGothic", face = "bold", size = 18),
          plot.subtitle = element_text(family = "Source Sans Pro", size = 14, color = "gray24", face = "bold"), 
          panel.background = element_rect(color = "cyan3", fill = "cyan3"),
          plot.background = element_rect(fill = "cyan2")
          )
```

```
## Warning in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)): font family not
## found in Windows font database

## Warning in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)): font family not
## found in Windows font database

## Warning in grid.Call(C_stringMetric, as.graphicsAnnot(x$label)): font family not
## found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Evolution of rat sighting per borough-1.png" width="672" />


We can see that the number of rat sightings raised in almost all boroughs, but has fallen in 2017.

Brooklyn is the borough with more sightings and Staten Island the one with fewer complaints.

#### Discution of aesthetic choices.

I've used no ticks nor grid lines because precise comparison between the states does not matter.
I've wanted to emphasize the evolution in time, and I added the point so it makes easier to see each year.

I've dropped the legends and added the boroughs' name directly in the graph, just because it seems more organic.

There is strong contrast between the lines, so it is clear that they are representing different entities. Besides that,
I tried to create contrast between the title and the subtitle.





**The End**





