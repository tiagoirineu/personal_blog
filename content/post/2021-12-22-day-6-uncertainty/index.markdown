---
title: Day 6 - Uncertainty
author: Tiago Irineu
date: '2021-12-22'
slug: day-6-uncertainty
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-12-22T06:24:43-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []

output:
    html_document:
        toc: true
    
---

## Uncertainty - day 6

### Reflection

When we are reading a plot we expect that every element in the chart is represent one aspect of the data. The problem with adding uncertainty to our plots is that most of the time it is not obvious to the reader what we are representing, given that the uncertainty is not a direct representation of the data.

Take for example the needle movement in the [New York Time's election dial](https://flowingdata.com/2016/11/15/showing-uncertainty-during-the-live-election-forecast/). For a reader, a possible interpretation was that it was moving given the arrival of live data. But this was not the case. It was just a random movement intended to show the uncertainty related to any forecast. But instead of informing people about the inherent uncertainty, it created anxiety and confusion.

It is important to be careful when trying to show uncertainty in our plots because uncertainty and randomness are not concepts that humans easily grasp. It is our obligation to not mislead our readers showing more confidence in some data than it is warranted, but when doing this we must consider the difficulty of perception that human have. Or we may end up building plots that create more confusion than inform the reader.

### Example

In these examples we will work through different visualization techniques for one variable.


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
library(lubridate)
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
library(ggridges)
library(gghalves)
```



```r
weather_raw <- read_csv("C:/Users/tiago/OneDrive/datasets/data_viz-course/atl-weather-2019.csv")
```

```
## Rows: 365 Columns: 40
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr   (3): summary, icon, precipType
## dbl  (29): moonPhase, precipIntensity, precipIntensityMax, precipProbability...
## dttm  (8): time, sunriseTime, sunsetTime, precipIntensityMaxTime, temperatur...
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
head(weather_raw)
```

```
## # A tibble: 6 x 40
##   time                summary     icon   sunriseTime         sunsetTime         
##   <dttm>              <chr>       <chr>  <dttm>              <dttm>             
## 1 2019-01-01 05:00:00 Light rain~ rain   2019-01-01 12:44:00 2019-01-01 22:41:00
## 2 2019-01-02 05:00:00 Rain start~ rain   2019-01-02 12:44:00 2019-01-02 22:42:00
## 3 2019-01-03 05:00:00 Rain throu~ rain   2019-01-03 12:44:00 2019-01-03 22:42:00
## 4 2019-01-04 05:00:00 Heavy rain~ rain   2019-01-04 12:44:00 2019-01-04 22:43:00
## 5 2019-01-05 05:00:00 Clear thro~ partl~ 2019-01-05 12:44:00 2019-01-05 22:44:00
## 6 2019-01-06 05:00:00 Clear thro~ clear~ 2019-01-06 12:44:00 2019-01-06 22:45:00
## # ... with 35 more variables: moonPhase <dbl>, precipIntensity <dbl>,
## #   precipIntensityMax <dbl>, precipIntensityMaxTime <dttm>,
## #   precipProbability <dbl>, precipType <chr>, temperatureHigh <dbl>,
## #   temperatureHighTime <dbl>, temperatureLow <dbl>, temperatureLowTime <dbl>,
## #   apparentTemperatureHigh <dbl>, apparentTemperatureHighTime <dbl>,
## #   apparentTemperatureLow <dbl>, apparentTemperatureLowTime <dbl>,
## #   dewPoint <dbl>, humidity <dbl>, pressure <dbl>, windSpeed <dbl>, ...
```


```r
weather_atl <- weather_raw %>% 
    mutate(Month = month(time, label = TRUE, abbr = FALSE), # Extract the month
           Day = wday(time, label = TRUE, abbr = FALSE))  # Extract the weekday as logical vector
```


#### Histograms


```r
ggplot(weather_atl, aes(x = windSpeed)) +
    geom_histogram(binwidth = 1, color = "white") # the color argument created a boundary between the bars
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Windspeed-1.png" width="672" />
*Note that geom_histogram() takes only one argument. The y variable will be a count of each element of the x variable.*




Now, we can improve the plot by forcing the boundaries to be set at whole numbers and also to show each step:


```r
ggplot(weather_atl, aes(x = windSpeed)) +
    geom_histogram(binwidth = 1, color = "white", boundary = 1) +
    scale_x_continuous(breaks = seq(0, 12, by = 1))  # defines the presentation at x scales
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />



#### Wind Speed by month


```r
ggplot(weather_atl, aes(x = windSpeed, fill = Month)) +
    geom_histogram(binwidth = 1, color = "white", boundary = 1) +
    scale_x_continuous(breaks = seq(0, 12, by = 1))  # defines the presentation at x scales
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

It's hard to interpret this graph. Let's facet it.


```r
ggplot(weather_atl, aes(x = windSpeed, fill = Month)) +
    geom_histogram(binwidth = 1, color = "white", boundary = 1) +
    scale_x_continuous(breaks = seq(0, 12, by = 1)) +  # defines the presentation at x scales
    guides(fill = FALSE) + # Does not show the information about the fill argument
    facet_wrap(vars(Month))
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Faceting-1.png" width="672" />


#### Density plots




```r
ggplot(weather_atl, aes(x = windSpeed)) +
  geom_density(color = "grey20", fill = "grey50")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/A basic density plot-1.png" width="672" />

Let's fill with month.


```r
ggplot(weather_atl, aes(x = windSpeed, fill = Month)) +
  geom_density(alpha = 0.5)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Density with fill-1.png" width="672" />

Really hard to see anything, except for the fact that it seems clear that from January to March we have the most windy months. 
Let's facet it.


```r
ggplot(weather_atl, aes(x = windSpeed, fill = Month)) +
  geom_density(alpha = 0.5)+
    guides(fill = FALSE) +
    facet_wrap(vars(Month))
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />


**Creating a ridge plot**


```r
ggplot(weather_atl, aes(x = windSpeed, y = fct_rev(Month), fill = Month)) +
  geom_density_ridges() +
  guides(fill = FALSE)
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

```
## Picking joint bandwidth of 0.521
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Ridge Plot-1.png" width="672" />
This is probably the clear graph so far. 

It's possible to add quantile information to the graph.


```r
ggplot(weather_atl, aes(x = windSpeed, y = fct_rev(Month), fill = Month)) +
  geom_density_ridges(quantile_lines = TRUE, quantiles = 2) + #As it divides in two quantiles, it creates the median line
  guides(fill = FALSE)
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

```
## Picking joint bandwidth of 0.521
```

<img src="{{< blogdown/postref >}}index_files/figure-html/ridge plot with the median-1.png" width="672" />


Now that we have a good working graph, we can use it to plot other variables such as temperature.





```r
ggplot(weather_atl, aes(x = temperatureHigh, y = fct_rev(Month), fill = Month)) +
  geom_density_ridges(quantile_lines = TRUE, quantiles = 2) + #As it divides in two quantiles, it creates the median line
  guides(fill = FALSE)
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

```
## Picking joint bandwidth of 2.72
```

<img src="{{< blogdown/postref >}}index_files/figure-html/ridge plot with the median and better color-1.png" width="672" />


As we are dealing with temperature, let's show it in the color.


```r
ggplot(weather_atl, aes(x = temperatureHigh, y = fct_rev(Month), fill = ..x..)) + #..x... makes the fill to use the var mapped to x
  geom_density_ridges_gradient(quantile_lines = TRUE, quantiles = 2) +
  scale_fill_viridis_c(option = "plasma") +
  labs(x = "High temperature", y = NULL, color = "Temp")
```

```
## Picking joint bandwidth of 2.72
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Temperature ggrides with viridis-1.png" width="672" />


As a final step we want to create a graph that shows the low temperature and the high.

First, some data manipulation.


```r
weather_atl_long <- weather_atl %>% 
    pivot_longer(cols = c(temperatureLow, temperatureHigh),
                 names_to = "temp_type", #column names will be mapped to this variable
                 values_to = "temp" ) %>% 
    # Changes the values in the new column
    mutate(temp_type = recode(temp_type,
                              temperatureHigh = "High",
                              temperatureLow = "Low")) %>% 
    select(time, temp_type, temp, Month)

head(weather_atl_long)
```

```
## # A tibble: 6 x 4
##   time                temp_type  temp Month  
##   <dttm>              <chr>     <dbl> <ord>  
## 1 2019-01-01 05:00:00 Low        50.6 janeiro
## 2 2019-01-01 05:00:00 High       63.9 janeiro
## 3 2019-01-02 05:00:00 Low        49.0 janeiro
## 4 2019-01-02 05:00:00 High       57.4 janeiro
## 5 2019-01-03 05:00:00 Low        53.1 janeiro
## 6 2019-01-03 05:00:00 High       55.3 janeiro
```


```r
ggplot(weather_atl_long, aes(x = temp, y = fct_rev(Month), 
                             fill = ..x.., linetype = temp_type)) +
  geom_density_ridges_gradient(quantile_lines = TRUE, quantiles = 2) +
  scale_fill_viridis_c(option = "plasma") +
  labs(x = "High temperature", y = NULL, color = "Temp")
```

```
## Picking joint bandwidth of 2.72
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Creating the graph-1.png" width="672" />


#### Box, violin, and rain cloud plots

**Boxplot**


```r
ggplot(weather_atl,
       aes( y = windSpeed, fill = Day)) +
    geom_boxplot()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/boxplot-1.png" width="672" />

**Violin Plot**


```r
ggplot(weather_atl,
       aes( y= windSpeed, x = Day, fill = Day)) +
    geom_violin() +
    guides(fill = FALSE)
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/violin-1.png" width="672" />

With violin plots it's useful to overlay other geoms, so we have a clearer picture.


```r
ggplot(weather_atl,
       aes(y = windSpeed, x = Day, fill = Day)) +
  geom_violin() +
  geom_point(size = 0.5, position = position_jitter(width = 0.1)) +
  xlab("") +
  guides(fill = FALSE)
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

**Emphasizing the mean point**


```r
ggplot(weather_atl,
       aes(y = windSpeed, x = Day, fill = Day)) +
    geom_violin() +
    stat_summary(geom = "point", fun = "mean", size = 2, color = "white") +
    geom_point(size = 0.5, position = position_jitter(width = 0.1)) +
    guides(fill = FALSE)  #hides the fill info box
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Using stat_summary-1.png" width="672" />

**Using gghalves**

gghalves enables us to plot only half of a violin plot or a box plot. This makes the plot more clean,showing the same amount of information with less overlay.


```r
ggplot(weather_atl,
       aes(x = fct_rev(Day), y = temperatureHigh)) +
    geom_half_point(aes(color = Day), side = "l", size = 0.5) +
    geom_half_boxplot(aes(fill = Day), side = "r") + #side arg sets which side will be shown
    guides(color = FALSE, fill = FALSE) +
    theme_light() +
    labs( x = "", y = "Temperature")
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/gghalves 1-1.png" width="672" />





```r
ggplot(weather_atl,
       aes(x = fct_rev(Day), y = temperatureHigh)) +
  geom_half_point(aes(color = Day), side = "l", size = 0.5) +
  geom_half_violin(aes(fill = Day), side = "r") +
  guides(color = FALSE, fill = FALSE) +
    theme_light()
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/half_violin-1.png" width="672" />


And the last example is a rain cloud plot. It is basically the last plot, but flipped.


```r
ggplot(weather_atl,
       aes(x = fct_rev(Day), y = temperatureHigh)) +
  geom_half_boxplot(aes(fill = Day), side = "l", width = 0.5, nudge = 0.1) +
  geom_half_point(aes(color = Day), side = "l", size = 0.5) +
  geom_half_violin(aes(fill = Day), side = "r") +
  guides(color = FALSE, fill = FALSE) + 
  coord_flip() +
    theme_light()
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/rain cloud plot-1.png" width="672" />

Good luck interpreting such graph.



### Exercise - Visualizing uncertainty with gapminder


```r
library(gapminder)
```


I. Make a histogram of logged GDP per capita for 1997 only, across all five continents


```r
gapminder %>% 
    filter(year == 1997) %>% 
    group_by(continent) %>% 
    ggplot() +
    geom_histogram(aes(x = log10(gdpPercap), fill = continent), binwidth = 0.1, color = "white", boundary = 1) + 
    facet_wrap(vars(continent)) +
    labs(x = "gdp per capita", y = "Number of countries") +
    scale_y_continuous(breaks = seq(0,10, by = 1)) +
    theme_light() +
    guides(fill = FALSE)
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Exercise 1-1.png" width="672" />

II. Make a ridge plot of global life expectancy over time, from 1952 to 2007. You’ll need to use the full gapminder data, not the 1997-only data. Each ridge should show the distribution of the world’s life expectancy for each given year




```r
gapminder %>% 
    ggplot(aes(x = lifeExp, y =as.factor(year))) +
    geom_density_ridges(aes(fill = as.factor(year)),quantile_lines = TRUE, quantiles = 2) +
    labs(title = "Life expectancy has been on the rise",
         subtitle = "Distribution of World life expectancy: 1956 to 2007",
         x = "",
         y = "Life expectancy") +
    guides(fill = FALSE) +
    theme_minimal() 
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

```
## Picking joint bandwidth of 3.88
```

<img src="{{< blogdown/postref >}}index_files/figure-html/ridge plot for life expectancy-1.png" width="672" />




III. Make a filtered dataset that selects data from only 2007 and removes Oceania. Show the distribution of logged GDP per capita across the four continents using some combination of boxplots and/or violin plots and/or strip plots, either overlaid on top of each other, or using their geom_half_*() counterparts from gghalves.


```r
gapminder %>% 
    filter(year==2007, continent != "Oceania") %>% 
    ggplot(aes(x = continent, y= log10(gdpPercap))) +
    geom_half_violin(aes(color = continent), side = "l") + 
    labs( x = "", y = "Gdp per capita") +
    geom_half_point(size = 0.5, side = "r", aes(color = continent)) +
    guides(color = FALSE) +
    labs(title = "Distribution of gdp per capita in 2007") +
    theme_gray()
```

```
## Warning: `guides(<scale> = FALSE)` is deprecated. Please use `guides(<scale> =
## "none")` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Exercise 3-1.png" width="672" />




