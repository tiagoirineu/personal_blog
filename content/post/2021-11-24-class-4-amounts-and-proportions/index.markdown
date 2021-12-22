---
title: 'Class 4: Amounts and proportions'
author: Tiago Irineu
date: '2021-11-24'
slug: class-4-amounts-and-proportions
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-11-24T21:25:19-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

### Exercise

#### Preliminary ----

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

**Loading the data** ----


```r
essential_construction <- read_csv("C:/Users/tiago/OneDrive/datasets/data_viz-course/EssentialConstruction.csv")
```

```
## Rows: 7209 Columns: 7
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr (5): ADDRESS, BOROUGH, CATEGORY, SUBCATEGORIES, JOB NUMBERS
## dbl (2): BIN, CD
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
essential_construction <- clean_names(essential_construction)
```

* Show the count or proportion of approved projects by borough using a bar chart

```r
essential_construction %>%
    mutate(borough = snakecase:: to_title_case(borough)) %>% 
    group_by(borough) %>% 
    summarise(n = n()) %>% 
    ggplot(aes(x = reorder(borough,n), y = n)) + 
    geom_col(fill = "#1B9E77") +
    coord_flip() +
    labs(x = "", y = "",
         title = "Number of projects approved by borough") +
    theme_light()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />





