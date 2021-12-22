---
title: 'Day 3: Mapping data to graphics'
author: Tiago Irineu
date: '2021-11-20'
slug: day-3-mapping-data-to-graphics
categories: []
tags: R; data visualization
subtitle: ''
summary: ''
authors: []
lastmod: '2021-11-20T09:11:52-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---
These are my notes for that Day 3 of the dataviz course. Today the focus is on mapping data to graphs. 

### Reading chapter 3 from [Kieran Healy's book](https://socviz.co/makeplot.html)

The **gg** in ggplot2 stands for grammar of graphics. This implies that we have an underlying logic that can be applied for the creation of any plot using ggplot.

Below, an example of how ggplot works.


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
p <- ggplot(gapminder,              #step one: Tell ggplot the data you want to use.
            mapping = aes(x = gdpPercap, y = lifeExp, # What variables are mapped to x-axis and yo y-axis?
                          size = pop, color = continent))  #Map variables to other elements in the plot
p+ geom_point() +  # geoms determine the object used to show the mapped data

   coord_cartesian() + # enable to work with the limits of the graph - Useful for zooming in some important data
    scale_x_log10() +  # Applied log10 to the x axis; this changes the Scale, but not the underlying data


  labs(x = "log GDP",
        y = "Life Expectancy",
        title = "A Gapminder Plot")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Example of how ggplot works-1.png" width="672" />


*mapping = aes()* links data to the elements we will in the plot. This is important.
In our example, we wanted that the color was mapped to the continent variable, so we used *mapping = aes(color = continent)* 
Whe you use that, ggplot looks at the provided data for a variable called "continent" and uses it to color the plot. If we wanted a color not related to the data, we should use the argument color outside of the aes() call.


### Steps in creating a plot in ggplot2

The advantage of using ggplot2 is that there is a process that you want to plot a chart. 

0. Tidy you data

1. Tell the ggplot() function what our data is.

2. Tell ggplot() what relationships we want to see.

3. Tell ggplot how we want to see the relationships in our data, by choosing a geom.

4. Layer on geoms as needed, by adding them to the p object one at a time.

5. Use some additional functions to adjust scales, labels, tick marks, titles. 


### Notes

* *scale_x_log10()* Scales the axis before plotting the smooth line, but does not changes the underlying data. 

* When setting values, pass the argument to the geom_ call.
* If possible, match color and fill to the same variable


#### Example


```r
p <- ggplot(data = gapminder,
            aes( x = gdpPercap,
                 y = lifeExp,
                 color = continent,
                 fill = continent))

p + geom_point() +
    geom_smooth( method = "loess") +
    scale_x_log10(labels = scales:: comma)  #loads the package scales
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Example 1-1.png" width="672" />

```r
p <- ggplot(data = gapminder,
            aes( x = gdpPercap,
                 y = lifeExp,
                ))

p + geom_point( aes(color = continent)) +
    geom_smooth( method = "loess") +
    scale_x_log10(labels = scales:: comma)
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Example 2-1.png" width="672" />


### Exercise - The fellowship of the Ring analysis

#### First, let's load the data and combine the three tables in only one dataset.


```r
fellowship <- read_csv("C:/Users/tiago/OneDrive/datasets/lord_of_the_rings/The_Fellowship_Of_The_Ring.csv")
```

```
## Rows: 3 Columns: 4
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr (2): Film, Race
## dbl (2): Female, Male
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
return <- read_csv("C:/Users/tiago/OneDrive/datasets/lord_of_the_rings/The_Return_Of_The_King.csv")
```

```
## Rows: 3 Columns: 4
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr (2): Film, Race
## dbl (2): Female, Male
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
towers <- read_csv("C:/Users/tiago/OneDrive/datasets/lord_of_the_rings/The_Two_Towers.csv")
```

```
## Rows: 3 Columns: 4
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr (2): Film, Race
## dbl (2): Female, Male
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```


```r
lotr <- as_tibble(bind_rows(fellowship, return, towers))
```


1. **Use group_by() and summarize() on the lotr data to find the total number of words spoken by race. Don’t worry about plotting it. How many words did male hobbits say in the movies?**


```r
lotr %>% 
    group_by(Race) %>% 
    summarise(words_spoken = sum(Female, Male))
```

```
## # A tibble: 3 x 2
##   Race   words_spoken
##   <chr>         <dbl>
## 1 Elf            3737
## 2 Hobbit         8796
## 3 Man            8712
```
As we can, hobbit is the race that spoke the most, closely followed by Man.


```r
lotr %>% 
    group_by(Race) %>% 
    filter(Race == "Hobbit") %>% 
    summarise(male_hobbit = sum(Male))
```

```
## # A tibble: 1 x 2
##   Race   male_hobbit
##   <chr>        <dbl>
## 1 Hobbit        8780
```
Wow, only 16 hobbit lines were spoken by females. 



2 . **Use group_by() and summarize() to answer these questions with bar plots (geom_col())**


```r
lotr %>% 
    group_by(Race) %>% 
    summarise(spoken_words = sum(Female, Male)) %>% 
    ggplot(mapping = aes(x = Race, y = spoken_words)) +
    geom_col() +
    theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />
For answering the next question we must make a small correction in our data.
It is not in long format, and we need to adjust this by creating a Gender variable.



```r
lotr_2 <- lotr %>%  pivot_longer(Male:Female, names_to = "Gender", values_to = "Spoken_words") # Creating a long format

lotr_2
```

```
## # A tibble: 18 x 4
##    Film                       Race   Gender Spoken_words
##    <chr>                      <chr>  <chr>         <dbl>
##  1 The Fellowship Of The Ring Elf    Male            971
##  2 The Fellowship Of The Ring Elf    Female         1229
##  3 The Fellowship Of The Ring Hobbit Male           3644
##  4 The Fellowship Of The Ring Hobbit Female           14
##  5 The Fellowship Of The Ring Man    Male           1995
##  6 The Fellowship Of The Ring Man    Female            0
##  7 The Return Of The King     Elf    Male            510
##  8 The Return Of The King     Elf    Female          183
##  9 The Return Of The King     Hobbit Male           2673
## 10 The Return Of The King     Hobbit Female            2
## 11 The Return Of The King     Man    Male           2459
## 12 The Return Of The King     Man    Female          268
## 13 The Two Towers             Elf    Male            513
## 14 The Two Towers             Elf    Female          331
## 15 The Two Towers             Hobbit Male           2463
## 16 The Two Towers             Hobbit Female            0
## 17 The Two Towers             Man    Male           3589
## 18 The Two Towers             Man    Female          401
```
3. **Which gender is dominant?**


```r
lotr_2 %>% 
    group_by(Film, Gender) %>% 
    summarise(words = sum(Spoken_words)) %>% 
    ggplot(mapping = aes(x = Film, y = words )) +
    geom_col(aes(fill = Gender), position = "dodge")
```

```
## `summarise()` has grouped output by 'Film'. You can override using the `.groups` argument.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />
    

Surprising nobody, men dominate the number of lines in all three films.

4. **Create a plot that visualizes the number of words spoken by race, gender, and film simultaneously.** 


```r
  lotr_2%>% 
    ggplot(mapping = aes(x = Film, y = Spoken_words )) +
    geom_col(aes(fill = Gender), position = "dodge") +
    facet_grid(rows = vars(Race)) +
    labs(title = "Female hobbits had nothing to say in lotr",
         subtitle = "Lines spoken by Gender and Race",
         x = "",
         y = "Number of words spoken")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

In the first film, the only race where female said anything were the elf. 
It's crazy that no female woman should up in the first move.

This closes the exercises for day 3. 
