---
title: Day 7 - Relationships
author: Tiago Irineu
date: '2022-01-03'
slug: day-7-relationships
categories: []
tags: R, visualization
subtitle: ''
summary: ''
authors: []
lastmod: '2022-01-03T07:39:44-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: [Data Visualization Course]
---

# Visualizing relationships

## Reflection

One of the complications of data visualization is how easy we are to be mislead. Be it by other or, even easier, by ourselves. When dealing with two variables and their relationships this becomes an ever bigger problem because there are new ways of misleading ourselves and our readers.

Anybody that has ever read anything about statistics knows that **correlation does not imply causation**. And yet, misleading correlation and causation is probably the most committed mistake when communicating relationships between two variables. This seems to be a silly mistake, but it is not. If we are plotting two variables together, it is a safe bet that we are interested in how they interact, and how one is able to "explain" the other. And given that our brain is always seeking patterns is not really surprising that we see relationships where there is none, and that after that we make the jump to discuss causation without having any statistical basis for that. But how to solve this and does not mislead our reader?

First, we must be aware of spurious relationships. Is there any theory or reasoning that would justify a relationship between the two variables? If not, does not create a narrative just because the graph seems to show that they move together. Of course, visualization is an important part of exploratory data analysis, and in many moments it may show us surprising relationships. But we **should not** use this as a new big explanation for a phenomena, but as a first step in the exploratory process. In this way, we must be clear with ourselves and our readers in what is the role of that visualization. And if want to make the case for a meaningful relationship we must provide a reasonable hypothesis for why this is not a spurious relationship. 

Besides the possible statistical misunderstanding, we must also consider the problems in the human perception. Most adults will treat the x-axis as the explanatory variable and the y-variable as the explained. So, when plotting the relationship between two variable we should probably follow this. Besides that, we should avoid using two y-axes because it creates a new range of difficulty for interpreting the results. 

In the end, we must strive to be truthful. If we do this, and consider the possible statistical traps and perception problems humans have, we will make a good job.

## Example

This example session is important because it goes through the basics of regression graphical analysis. This is a first step in statistical modelling 


### Loading and Cleaning the data


```r
library(tidyverse)
library(patchwork) # For combining ggplots plots
library(GGally) # For scatterplot matrices
library(broom)  # For converting model objects to data frames
```


```r
weather_atl <- read_csv("C:/Users/tiago/OneDrive/datasets/data_viz-course/atl-weather-2019.csv")
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

### Legal dual y-axes

It is bad practice to use two y-axes for two different variables. It create a lot of room for misinterpretation. But, it may be a good ideal to use two y-axes to show the same variable but with some transformation. For example, showing temperature in Fahrenheit and Celsius. Pounds and kilograms. Inches and centimeters.

For this, we must use the ggplot argument *scale_y_continuous()* with *sec.axis*.

Let's consider the formula for temperature transformation.

$$ C=(32-F)* \frac{5}{9}$$


```r
ggplot(weather_atl, aes(x = time, y = temperatureHigh)) +
    geom_line() +
    scale_y_continuous(sec.axis = sec_axis(trans = ~ (32-.)* -5/9,
                                           name = "Celsius")) +
    labs(x = NULL, y = "Fahrenheit") +
    theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Temperature plot-1.png" width="672" />

This works because we are not comparing two variables. Instead, we are just showing how the temperature evolved throughout the year. 

### Combining Plots

Now, let's use the package [patchwork](https://patchwork.data-imaginist.com/articles/patchwork.html) to combine multiple plots instead of using double-y axis.

To use **patchwork** we must first assign the plots for objects and then add them together using **+**.



* **Using patchwork to check the relationship between humidity and temperature in Atlanta.**


```r
# Temperature in Atlanta

temp_plot <- ggplot(weather_atl, aes (x = time, y = temperatureHigh)) +
    geom_line() +
    geom_smooth() +
    scale_y_continuous(sec.axis = sec_axis(trans = ~(32 -.) * 5/9,
                                           name = "Celsius")) +
    labs(x = NULL, y = "Fahrenheit") +
    theme_minimal()

temp_plot
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Temperature in Atlanta-1.png" width="672" />


```r
humidity_plot <- ggplot(weather_atl, aes( x= time, y = humidity)) +
    geom_line() +
    geom_smooth() +
    labs(x = NULL, y = "humidity") +
    theme_minimal()

humidity_plot
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Humidity in Atlanta-1.png" width="672" />
Now let's combine both plots using patchwork


```r
temp_plot + humidity_plot +
    plot_layout(ncol = 1) # Defines that the plots must be one above the other
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Using patchwork to combine both plots-1.png" width="672" />
As we can see there is no clear relationship between both variables. While humidity is pretty constant throughout the year, temperature follows the expected pattern and it's much higher in Summer than in Winter.

By default patchwork adds the plots side by side. Using the *plot_layout()* argument it's possible to choose how the plots will be displayed. 


### Scatterplot matrices

Visualizing correlations between pairs of variables with the *ggpairs()* functions, from **GGally** package.


```r
weather_correlations <- weather_atl %>% 
    select(temperatureHigh, temperatureLow, humidity, windSpeed, precipProbability)

ggpairs(weather_correlations)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Using GGally-1.png" width="672" />


**This can be useful in exploratory data analysis for a general overview of the relationship between the variables.**

In the diagonal we have the density plots of the variables. Above, we have the correlation coefficient and below we have the scatter plots of the variables.


### Correlograms

Correlograms are basically heatmaps showing the relationship between the variables.

* The first step is to build a correlation matrix.


```r
things_to_correlate <- weather_atl %>%  # Creating the correlation matrix
    select(temperatureHigh, temperatureLow, humidity, windSpeed, precipProbability) %>% 
    cor()

# Dropping the lower half because has the same values as the top half

things_to_correlate[lower.tri(things_to_correlate)] <- NA
things_to_correlate
```

```
##                   temperatureHigh temperatureLow    humidity   windSpeed
## temperatureHigh                 1      0.9202715 -0.03010715 -0.37653399
## temperatureLow                 NA      1.0000000  0.11234206 -0.44974330
## humidity                       NA             NA  1.00000000  0.01078074
## windSpeed                      NA             NA          NA  1.00000000
## precipProbability              NA             NA          NA          NA
##                   precipProbability
## temperatureHigh         -0.12410693
## temperatureLow          -0.02550408
## humidity                 0.72194566
## windSpeed                0.19621653
## precipProbability        1.00000000
```

Now, let's wrangle the data so it is in the appropriate format for ploting. 



```r
things_to_correlate_long <- things_to_correlate %>% 
    #Convert to a data frame
    as.data.frame() %>% 
    # as.data.frame does not convert the matrix column names to columns in the df. So we must add it
    # manually
    rownames_to_column("measure2") %>% 
    
    # Make this long.
    pivot_longer(cols = -measure2,             # Selects all columns except for measure2
                 names_to = "measure1",
                 values_to = "cor") %>%
    mutate(nice_cor = round(cor, 2)) %>% 
    
    #Remove rows where the two measures are the same
     
    filter(measure2 != measure1) %>% 
    
    #Get rid of the empty triangle
    
    filter(!is.na(cor)) %>% 
    
    # Put these categories in order
    
    mutate(measure1 = fct_inorder(measure1),
           measure2 = fct_inorder(measure2))

things_to_correlate_long
```

```
## # A tibble: 10 x 4
##    measure2        measure1              cor nice_cor
##    <fct>           <fct>               <dbl>    <dbl>
##  1 temperatureHigh temperatureLow     0.920      0.92
##  2 temperatureHigh humidity          -0.0301    -0.03
##  3 temperatureHigh windSpeed         -0.377     -0.38
##  4 temperatureHigh precipProbability -0.124     -0.12
##  5 temperatureLow  humidity           0.112      0.11
##  6 temperatureLow  windSpeed         -0.450     -0.45
##  7 temperatureLow  precipProbability -0.0255    -0.03
##  8 humidity        windSpeed          0.0108     0.01
##  9 humidity        precipProbability  0.722      0.72
## 10 windSpeed       precipProbability  0.196      0.2
```

Now, finally. Let's make the correlogram.



```r
ggplot(things_to_correlate_long,
       aes( x = measure2, y = measure1, fill = cor)) +
    geom_tile() +    # Creates the tile heatmap
    geom_text(aes(label = nice_cor)) +
    scale_fill_gradient2(low = "#E16462", mid= "white", high = "#0D0887",      # Creates a divergent pallete
                         limits = c(-1,1)) +
    labs( x = NULL, y = NULL) +
    coord_equal() +
    theme_minimal() +
    theme(panel.grid = element_blank())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Correlogram-1.png" width="672" />

Now, let's do the same using points.



```r
ggplot(
    things_to_correlate_long,
    aes(x = measure2, y = measure1, color = cor)) +
    geom_point(aes(size = abs(cor))) +    # Use abs for getting the same color for -1 and 1
    scale_color_gradient2( low = "#E16462", mid = "white", high = "#0D0887",
                           limits = c(-1, 1)) +
    scale_size_area(max_size = 15, limits = c(-1,1), guide =FALSE) +
                        labs(x = NULL, y = NULL) +
                        coord_equal() +              # Guarantees that they are plotted proportionally
                        theme_minimal() +
                        theme(panel.grid = element_blank())
```

```
## Warning: It is deprecated to specify `guide = FALSE` to remove a guide. Please
## use `guide = "none"` instead.
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Correlograms with points-1.png" width="672" />

### Regression

#### Simple Regression

In a simple regression we are building a statistical model for measuring the relationship between an independent variable and a dependent one. It is called simple because we are using only one variable for understanding the variation in the dependent variable. 

We can represent a simple regression model as below.

$$ \hat{y} = \beta_0 + \beta_1x_1 + \epsilon$$

Where $ \beta_0 $ is the intercept. This would be the predicted value when $ x_1 $ is equal to zero. The $ \epsilon $ represents all the factors that influence the outcome variable but are not being considered in our model.

We interpret this model as saying that **one unit increase in $ x_1 $ is associated with a $ \beta_1 $ increase/decrease in $ \hat{y} $, on average.

____________
Let's first wrangle some data


```r
weather_atl_summer <- weather_atl %>% 
  filter(time >= "2019-05-01", time <= "2019-09-30") %>% 
  mutate(humidity_scaled = humidity * 100,
         moonPhase_scaled = moonPhase * 100,
         precipProbability_scaled = precipProbability * 100,
         cloudCover_scaled = cloudCover * 100)
```

Now we can fit the model.


```r
model_simple <- lm(temperatureHigh ~ humidity_scaled,
                   data = weather_atl_summer)

tidy(model_simple, conf.int = TRUE)  # Transforms the outcome matrix in a dataframe
```

```
## # A tibble: 2 x 7
##   term            estimate std.error statistic  p.value conf.low conf.high
##   <chr>              <dbl>     <dbl>     <dbl>    <dbl>    <dbl>     <dbl>
## 1 (Intercept)      104.       2.36       44.0  1.13e-87   99.3     109.   
## 2 humidity_scaled   -0.239    0.0359     -6.65 5.12e-10   -0.310    -0.168
```

In R we build regression models using the format **dependent_variable ~ independent_variable**. So, in our model we are looking at how humidity explains the variation in temperatureHigh.

 **Interpretation**
* Usually there is no relevant interpretation of the intercept. As we do not have a day with zero humidity, this value is not important. It is relevant only for plotting the regression line.

* The value estimated for humidity is saying that one percent increase in humidity is associated with a decrease of 0.238 °F degrees in temperature, on average.

___

Since we are dealing with only two variables we can easily visualize it now.


```r
ggplot( weather_atl_summer,
        aes(x = humidity_scaled, y = temperatureHigh)) +
    geom_point() +
    geom_smooth(method = "lm") +
    labs(x = "Humidity", y = "Temperature") +
    theme_minimal()
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Visualizign a simple regression model-1.png" width="672" />

Now we can visualize the fact that as humidity increases, temperature decreases.


#### Multiple Regression models

Sadly for us we do not live in a simple world where one variable is usually enough for explaining the problems we want to understand. Therefore, we need to consider multiple variables when studying any phenomena. This makes impossible to use simple visualization such as scatterplots, but there are still some options. Below, we first fit a multiple regression, and after this we go through some visualization options. 

A multiple regression has the following form:

$$ \hat{y} = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \space ... \space + \beta_n x_n + \epsilon $$
where $ x_i $ represents an independent/explanatory variable and $ \beta_i $ is the estimated coefficient for this variable. 

We can interpret the coefficient in two ways, conditional in the type of the variable.

* For **continuous** variables: Holding everything else constant, one unit increase in X is associated with a $ \beta_i $ increase/decrease in Y, on average.


* For **categorical** variables: $ X_n $ is $ \beta_n $ units larger/smaller than the $ X_omitted $ variable on average.
*An explanation is necessary here. When using categorical variables, one value of the variable will be omitted and used as a standard for comparison *


**Fitting the model**


```r
model_complex <- lm(temperatureHigh ~ humidity_scaled + moonPhase_scaled +
                        precipProbability_scaled + windSpeed + pressure + cloudCover_scaled,
                    data = weather_atl_summer)

tidy(model_complex, conf.int = TRUE)    # It will return the confidence intervals form each coefficient
```

```
## # A tibble: 7 x 7
##   term                  estimate std.error statistic  p.value conf.low conf.high
##   <chr>                    <dbl>     <dbl>     <dbl>    <dbl>    <dbl>     <dbl>
## 1 (Intercept)           265.      126.         2.11   3.66e-2 16.7      514.    
## 2 humidity_scaled        -0.111     0.0759    -1.47   1.44e-1 -0.261      0.0387
## 3 moonPhase_scaled        0.0124    0.0128     0.965  3.36e-1 -0.0129     0.0377
## 4 precipProbability_sc~   0.0358    0.0204     1.75   8.16e-2 -0.00454    0.0761
## 5 windSpeed              -1.76      0.417     -4.22   4.30e-5 -2.58      -0.936 
## 6 pressure               -0.160     0.123     -1.30   1.96e-1 -0.403      0.0834
## 7 cloudCover_scaled      -0.0950    0.0304    -3.12   2.17e-3 -0.155     -0.0349
```

#### Coefficient plots

Now, we can use this table for plotting the coefficients. This is a good way of seeing how big is the impact and if it seems to be statistically different from zero. 



```r
# First, we drop the intercept. It's coefficient plot after all

model_tidied <- tidy(model_complex, conf.int = TRUE) %>%  
    filter(term != "(Intercept)")

ggplot(model_tidied,
       aes(x = estimate, y = term)) +
    # Adding a vertical line at 0, for helping the interpretation
    geom_vline(xintercept = 0, color = "red", linetype = "dotted") + 
    # Creates the confidence intervals
    geom_pointrange(aes(xmin = conf.low, xmax = conf.high)) +
    labs(x = "Coefficient estimate", y = NULL) +
    theme_minimal()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Coefficient plots-1.png" width="672" />

We can see that wind speed has a considerable impact on temperature, while the other variables have effects close to zero. 

#### Marginal effects plots

A marginal effect is the effect that the movement in one independent variable has on the dependent variable. For getting this, we keep all the other variable constants and then consider the possible values for the variable of our interest.

Below, we use the **broom** library to simulate this.

* **augment()** enables us to plug a data frame of explanatory variables into a model equation and then get predicted values.











