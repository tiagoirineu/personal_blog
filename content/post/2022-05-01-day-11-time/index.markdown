---
title: 'Day 11: Time'
author: Tiago Irineu
date: '2022-05-01'
slug: day-11-time
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2022-05-01T15:46:15-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---
This lesson is about how to analyze time series in R.

### Example


```r
library(tidyverse)
library(tidyquant)   # Tidyquant R package for financial data manipulation
library(scales)
```
We will use the *tidyquant* package to access data from [FRED](https://fred.stlouisfed.org/).




```r
fred_raw <- tq_get(c("RSXFSN", # Advance retail sales
                     "GDPC1", # GDP
                     "ICSA", #Initial Unemployment claims
                     "FPCPITOTLZGUSA", # Inflation
                     "UNRATE", # Unemployment Rate
                     "USREC"), # Recessions
                   get = "economic.data", # data source, FRED
                   from = "1990-01-01"
                     )
```
It is not wise to access the FRED server everytime we run this script. Therefore, we will save the data we just downloaded for future use.


```r
write_csv(fred_raw, "fred_raw.csv")
```


```r
fred_raw <- read_csv("fred_raw.csv")
```

```
## Rows: 2983 Columns: 3
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr  (1): symbol
## dbl  (1): price
## date (1): date
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```


```r
fred_raw
```

```
## # A tibble: 2,983 x 3
##    symbol date        price
##    <chr>  <date>      <dbl>
##  1 RSXFSN 1992-01-01 130683
##  2 RSXFSN 1992-02-01 131244
##  3 RSXFSN 1992-03-01 142488
##  4 RSXFSN 1992-04-01 147175
##  5 RSXFSN 1992-05-01 152420
##  6 RSXFSN 1992-06-01 151849
##  7 RSXFSN 1992-07-01 152586
##  8 RSXFSN 1992-08-01 152476
##  9 RSXFSN 1992-09-01 148158
## 10 RSXFSN 1992-10-01 155987
## # ... with 2,973 more rows
```
Every data we pull using tidyquant comes in this format with three columns. If you remember, we downloaded multiple variables, and these variables are in the symbol column. While the price column will show the value associated with that symbol in a specific moment in time.

If we know that some variables have values for the same time period, we can use pivot_wider to transform it to more familiar syle.


```r
fred_monthly_things <-  fred_raw %>% 
    filter(symbol %in% c("UNRATE", 'RSXFSN')) %>% 
    # Get column names from the symbol column
    # respective values from the price column
    pivot_wider(names_from = symbol, values_from = price) %>% 
    rename(unemployment = UNRATE, retail_sales = RSXFSN)

fred_monthly_things
```

```
## # A tibble: 387 x 3
##    date       retail_sales unemployment
##    <date>            <dbl>        <dbl>
##  1 1992-01-01       130683          7.3
##  2 1992-02-01       131244          7.4
##  3 1992-03-01       142488          7.4
##  4 1992-04-01       147175          7.4
##  5 1992-05-01       152420          7.6
##  6 1992-06-01       151849          7.8
##  7 1992-07-01       152586          7.7
##  8 1992-08-01       152476          7.6
##  9 1992-09-01       148158          7.6
## 10 1992-10-01       155987          7.3
## # ... with 377 more rows
```

This was just a brief demonstration of what can be done with the package.

### Plotting time

Usually lines are used to represent time series.


```r
gdp_only <- fred_raw %>% 
    filter(symbol == "GDPC1")

ggplot(gdp_only, aes(x = date, y = price)) +
    geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

**Unemployment Claims**


```r
fred_raw %>% 
    filter(symbol == "ICSA") %>% 
    ggplot(aes( x = date, y = price)) +
    geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />


#### Improving graphics


```r
ggplot(gdp_only , aes(x = date, y = price)) +
    geom_line(color = "#0074d9", size = 1.5) +
    scale_y_continuous(labels = dollar) + #Adds dollar signals in the number in the y x
    labs( y = "Billions of 2012 dollars",
          x = NULL,
          title = "US Gross Domestic Product",
          subtitle = "Quartely data; real 2012 dollars",
          caption = "Source: US Bureau of Economic Analysis and FRED") +
    theme_bw(base_family = "Roboto Condensed") +
    theme(plot.title = element_text(face = "bold"))
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

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

```
## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Now ,this is a simple and beautiful plot.

#### Adding an extra layer : Recessions


```r
recessions_tidy <- fred_raw %>% 
            filter(symbol == "USREC") %>% 
    mutate(recession_change = price - lag(price)) # If 1, it marks the start of a recession, if -1 it marks the end of a recession
recessions_tidy
```

```
## # A tibble: 387 x 4
##    symbol date       price recession_change
##    <chr>  <date>     <dbl>            <dbl>
##  1 USREC  1990-01-01     0               NA
##  2 USREC  1990-02-01     0                0
##  3 USREC  1990-03-01     0                0
##  4 USREC  1990-04-01     0                0
##  5 USREC  1990-05-01     0                0
##  6 USREC  1990-06-01     0                0
##  7 USREC  1990-07-01     0                0
##  8 USREC  1990-08-01     1                1
##  9 USREC  1990-09-01     1                0
## 10 USREC  1990-10-01     1                0
## # ... with 377 more rows
```


```r
recessions_start_end <- fred_raw %>% 
    filter(symbol == 'USREC') %>% 
    mutate(recession_change = price - lag(price)) %>% 
    filter(recession_change != 0 )
recessions_start_end 
```

```
## # A tibble: 8 x 4
##   symbol date       price recession_change
##   <chr>  <date>     <dbl>            <dbl>
## 1 USREC  1990-08-01     1                1
## 2 USREC  1991-04-01     0               -1
## 3 USREC  2001-04-01     1                1
## 4 USREC  2001-12-01     0               -1
## 5 USREC  2008-01-01     1                1
## 6 USREC  2009-07-01     0               -1
## 7 USREC  2020-03-01     1                1
## 8 USREC  2020-05-01     0               -1
```

Now, we combine both datasets.



```r
recessions <- tibble(start = filter(recessions_start_end, recession_change == 1)$date,
                     end=filter(recessions_start_end, recession_change == -1)$date)
recessions
```

```
## # A tibble: 4 x 2
##   start      end       
##   <date>     <date>    
## 1 1990-08-01 1991-04-01
## 2 2001-04-01 2001-12-01
## 3 2008-01-01 2009-07-01
## 4 2020-03-01 2020-05-01
```

Now, we add the recession years as a shade in our plot. As the line must be over the rectangular, it must be added after.


```r
ggplot(gdp_only, aes(x = date, y = price)) +
    geom_rect(data = recessions,
              aes(xmin = start, xmax = end, ymin = -Inf, ymax = Inf),
              inherit.aes = FALSE, fill = "#B10DC9", alpha = 0.3)+
    geom_line(color = '#0074D9', size = 1) +
    scale_y_continuous(labels = dollar) +
    labs( y = "Billions of 2012 dollars",
          x = NULL,
          title = "US Gross Domestic Product",
          subtitle = "Quartely data; real 2012 dollars",
          caption = "Source: US Bureau of Economic Analysis and FRED") +
    theme_bw(base_family = "Roboto Condensed") +
    theme(plot.title = element_text(face = "bold"))
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

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

```
## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />

And that's it. A beautiful plot, ready for publication

### Decomposition

*A time series can be divided in three components: the trend, the seasonality, and a random aspect that it's not explained by either.* 

**Example 1**

```r
library(tsibble) #For embedding time things into data frames
```

```
## 
## Attaching package: 'tsibble'
```

```
## The following object is masked from 'package:zoo':
## 
##     index
```

```
## The following object is masked from 'package:lubridate':
## 
##     interval
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, union
```

```r
retail_sales <- fred_raw %>% 
    filter(symbol == "RSXFSN") %>% 
    mutate(year_month = yearmonth(date)) %>% 
    as_tsibble(index = year_month) 
retail_sales
```

```
## # A tsibble: 363 x 4 [1M]
##    symbol date        price year_month
##    <chr>  <date>      <dbl>      <mth>
##  1 RSXFSN 1992-01-01 130683   1992 jan
##  2 RSXFSN 1992-02-01 131244   1992 fev
##  3 RSXFSN 1992-03-01 142488   1992 mar
##  4 RSXFSN 1992-04-01 147175   1992 abr
##  5 RSXFSN 1992-05-01 152420   1992 mai
##  6 RSXFSN 1992-06-01 151849   1992 jun
##  7 RSXFSN 1992-07-01 152586   1992 jul
##  8 RSXFSN 1992-08-01 152476   1992 ago
##  9 RSXFSN 1992-09-01 148158   1992 set
## 10 RSXFSN 1992-10-01 155987   1992 out
## # ... with 353 more rows
```

Now, we create a model for decomposing this time series. We will use the [STL decomposition](https://otexts.com/fpp2/stl.html)(**S**easonal and **T**rend decomposition **L**oess).

```r
library(feasts) 
```

```
## Carregando pacotes exigidos: fabletools
```

```r
retail_model <- retail_sales %>% 
    model(stl = STL(price))
retail_model
```

```
## # A mable: 1 x 1
##       stl
##   <model>
## 1   <STL>
```


```r
retail_components <- components(retail_model)
retail_components
```

```
## # A dable: 363 x 7 [1M]
## # Key:     .model [1]
## # :        price = trend + season_year + remainder
##    .model year_month  price   trend season_year remainder season_adjust
##    <chr>       <mth>  <dbl>   <dbl>       <dbl>     <dbl>         <dbl>
##  1 stl      1992 jan 130683 148390.     -21901.    4195.        152584.
##  2 stl      1992 fev 131244 148904.     -22686.    5026.        153930.
##  3 stl      1992 mar 142488 149418.      -1804.   -5126.        144292.
##  4 stl      1992 abr 147175 149932.      -2815.      57.9       149990.
##  5 stl      1992 mai 152420 150478.       5337.   -3394.        147083.
##  6 stl      1992 jun 151849 151023.       3011.   -2185.        148838.
##  7 stl      1992 jul 152586 151569.        350.     667.        152236.
##  8 stl      1992 ago 152476 152145.       3967.   -3635.        148509.
##  9 stl      1992 set 148158 152720.      -5497.     935.        153655.
## 10 stl      1992 out 155987 153296.        116.    2575.        155871.
## # ... with 353 more rows
```


But time series really need a plot to get a idea of what it's happening.


```r
autoplot(retail_components) +
    labs(x = NULL) +
    theme_bw(base_family = "Roboto Condensed") +
    theme(plot.title = element_text(face = "bold"))
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
```

```
## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database

## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" />


It is interesting that we can the Covid impact in the remainder, but the recuperation was so fast that it did not impact the trend. Contrast this with the 2009 crisis, where actually see a change in the retail sales trend.

If we want, we can plot just a component. Below, we plot just the trend. So we can see more clearly the impact of the 2009 crisis and the pandemic.


```r
ggplot(retail_components, 
       aes(x = as_date(year_month), y = trend)) +
  geom_rect(data = recessions, 
            aes(xmin = start, xmax = end, ymin = -Inf, ymax = Inf),
            inherit.aes = FALSE, fill = "#B10DC9", alpha = 0.3) +
  geom_line() + 
  scale_y_continuous(labels = dollar) +
  labs(x = NULL, y = "Trend, millions of dollars",
       title = "Seasonally adjusted trends in retail sales",
       subtitle = "Nominal US dollars") +
  theme_bw(base_family = "Roboto Condensed") +
  theme(plot.title = element_text(face = "bold"))
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
```

```
## Warning in grid.Call.graphics(C_text, as.graphicsAnnot(x$label), x$x, x$y, :
## font family not found in Windows font database
```

```
## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database

## Warning in grid.Call(C_textBounds, as.graphicsAnnot(x$label), x$x, x$y, : font
## family not found in Windows font database
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />



#### A more detailed version

**First, let's create a tidy dataset**.


```r
retail_components_tidy <- retail_components %>% 
    #Get rid of this colum %>% 
    select(-season_adjust) %>% 
    
    # Combine all components in a column called components
    
    pivot_longer(cols = c(price, trend, season_year, remainder),
                 names_to = "component", values_to = "value") %>% 
    
    # Recode this values so they're nicer
    mutate(component = recode(component,
                             price = "Actual data",
                             trend = "Trend",
                             season_year = "Seasonality",
                             remainder = "Remainder")) %>% 
    # Make the component categories follow the order they're in the data
    
    mutate( component = fct_inorder(component)) 

retail_components_tidy
```

```
## # A tsibble: 1,452 x 4 [1M]
## # Key:       .model, component [4]
##    .model year_month component     value
##    <chr>       <mth> <fct>         <dbl>
##  1 stl      1992 jan Actual data 130683 
##  2 stl      1992 jan Trend       148390.
##  3 stl      1992 jan Seasonality -21901.
##  4 stl      1992 jan Remainder     4195.
##  5 stl      1992 fev Actual data 131244 
##  6 stl      1992 fev Trend       148904.
##  7 stl      1992 fev Seasonality -22686.
##  8 stl      1992 fev Remainder     5026.
##  9 stl      1992 mar Actual data 142488 
## 10 stl      1992 mar Trend       149418.
## # ... with 1,442 more rows
```


Now, we can create the final plot:


```r
ggplot(retail_components_tidy, 
       aes(x = as_date(year_month), y = value)) +
  geom_rect(data = recessions, 
            aes(xmin = start, xmax = end, ymin = -Inf, ymax = Inf),
            inherit.aes = FALSE, fill = "#B10DC9", alpha = 0.3) +
  geom_line() + 
  scale_y_continuous(labels = dollar) +
  labs(x = NULL, y = "Millions of dollars",
       title = "Decomposed US Advance Retail Sales",
       subtitle = "Nominal US dollars",
       caption = "Source: US Census Bureau and FRED (RSXFSN)") +
  facet_wrap(vars(component), ncol = 1, scales = "free_y") +
  theme_minimal(base_family = "Roboto Condensed") +
  theme(plot.title = element_text(face = "bold"),
        plot.title.position = "plot",
        strip.text = element_text(face = "bold", hjust = 0))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-16-1.png" width="672" />









