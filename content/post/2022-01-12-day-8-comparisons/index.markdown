---
title: Day 8 - Comparisons
author: Tiago Irineu
date: '2022-01-12'
slug: day-8-comparisons
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2022-01-12T07:35:18-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


## Comparisons

The readings for today had a discussion about different comparison methods. The reflection will be focused on that.

### Reflection

Usually we are interested in two types of comparisons. We can be comparing a variable across entities, such as the income per capita in different countries or states. Or we can be interested in the evolution of the variable in time, be it in the same entity or in different entities as exemplified before.

From the techniques, two caught my attention. When comparing across multiple levels, I believe ridge plots are the most beautiful and informative. They are stacked density plots, and density plots are great for visualizing distributions. What ridge plots does is to use this for helping us see how distribution is different across categories or how it has been evolving in time.

We just must be aware that density plots can be misleading if there are few data points and/or they are too spread. One way to control for this problem is to also plot the points representing the data. In reality **we must always look at data** before visualizing it in graphs that summarize the data.

Another important topic raised is how we choose the *differences* we want to report. Should we report absolute change or percent variation? The answer is simple, there is not correct answer. This is context specific. But I do have a personal rule. Whenever a story reports the percent change, look for the absolute numbers. When the absolute number are reported, look for the percent number. Because usually the most impactful metric is the one chosen, and the most impactful not translate always as the most insightful. And when we are the creators, what we have to do is choose the metric that gives a true picture of reality for our reader.

### Example

This example section use information downloaded from the [World Bank data portal](https://datavizm20.classes.andrewheiss.com/example/08-example/#load-and-clean-data). I believe it will be a very interesting section.


First, we load the necessary packages.


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
library(WDI)        # package for getting data form the World Bank
library(geofacet)   #map-shaped facets
library(scales)     # for using scale functions
```

```
## 
## Attaching package: 'scales'
```

```
## The following object is masked from 'package:purrr':
## 
##     discard
```

```
## The following object is masked from 'package:readr':
## 
##     col_factor
```

```r
library(ggrepel)
```

For downloading data from the WDI base, we must pass the indicator code to the *WDI()* function.

* [WDI package page](https://cran.r-project.org/web/packages/WDI/index.html)
* 


```r
# We set eval=FALSE so the data will not be downloaded everytime we run this script

indicators <- c("SP.DYN.LE00.IN",  # Life expectancy
                "EG.ELC.ACCS.ZS",  # Access to electricity
                "EN.ATM.CO2E.PC",  # CO2 emissions
                "NY.GDP.PCAP.KD")  # GDP per capita

wdi_raw <- WDI(country = "all", indicators, extra = TRUE, 
               start = 1995, end = 2015)
```

For not having to access the WDI repository every time we want to work with this data we just downloaded, we can save it locally.


```r
#Use a path that makes sense for you.

write_csv(wdi_raw, "C:/Users/tiago/OneDrive/datasets/data_viz-course/wdi_raw.csv")
```

Now, that we have downloaded and saved the data. We will create a chunk for loading the saved data and then put *eval = FALSE* for the previous two chunks of code. In this way we will not download it and save it every time we run this code.


```r
wdi_raw <- read_csv("C:/Users/tiago/OneDrive/datasets/data_viz-course/wdi_raw.csv")
```

```
## Rows: 5586 Columns: 14
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr (7): iso2c, country, iso3c, region, capital, income, lending
## dbl (7): year, SP.DYN.LE00.IN, EG.ELC.ACCS.ZS, EN.ATM.CO2E.PC, NY.GDP.PCAP.K...
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
glimpse(wdi_raw)
```

```
## Rows: 5,586
## Columns: 14
## $ iso2c          <chr> "1A", "1A", "1A", "1A", "1A", "1A", "1A", "1A", "1A", "~
## $ country        <chr> "Arab World", "Arab World", "Arab World", "Arab World",~
## $ year           <dbl> 1998, 1997, 2002, 1999, 2000, 2001, 2010, 2011, 2012, 2~
## $ SP.DYN.LE00.IN <dbl> 67.21809, 66.89714, 68.34025, 67.51769, 67.80059, 68.07~
## $ EG.ELC.ACCS.ZS <dbl> 78.11158, 77.25362, 77.51198, 78.69106, 78.87248, 79.55~
## $ EN.ATM.CO2E.PC <dbl> 3.163145, 3.213285, 3.347227, 3.131299, 3.163305, 3.282~
## $ NY.GDP.PCAP.KD <dbl> 4515.898, 4388.554, 4607.143, 4504.817, 4705.559, 4681.~
## $ iso3c          <chr> "ARB", "ARB", "ARB", "ARB", "ARB", "ARB", "ARB", "ARB",~
## $ region         <chr> "Aggregates", "Aggregates", "Aggregates", "Aggregates",~
## $ capital        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,~
## $ longitude      <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,~
## $ latitude       <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,~
## $ income         <chr> "Aggregates", "Aggregates", "Aggregates", "Aggregates",~
## $ lending        <chr> "Aggregates", "Aggregates", "Aggregates", "Aggregates",~
```

Now, data cleaning.

*One thing that I did not know. It is possible to change the name of the columns while you are using select. This will save time.*


```r
wdi_clean <- wdi_raw %>% 
    # First let's drop rows that are not for individual countries
    filter(region != "Aggregates") %>% 
    select(iso2c, country, year,
           life_expectancy = SP.DYN.LE00.IN,
           access_to_electricity = EG.ELC.ACCS.ZS,
           co2_emissions = EN.ATM.CO2E.PC,
           gdp_per_cap = NY.GDP.PCAP.KD,
           region, income)
glimpse(wdi_clean)
```

```
## Rows: 4,536
## Columns: 9
## $ iso2c                 <chr> "AD", "AD", "AD", "AD", "AD", "AD", "AD", "AD", ~
## $ country               <chr> "Andorra", "Andorra", "Andorra", "Andorra", "And~
## $ year                  <dbl> 2015, 2002, 2003, 2004, 2005, 1997, 1998, 1999, ~
## $ life_expectancy       <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, ~
## $ access_to_electricity <dbl> 100, 100, 100, 100, 100, 100, 100, 100, 100, 100~
## $ co2_emissions         <dbl> 6.026182, 7.566240, 7.242416, 7.344262, 7.353780~
## $ gdp_per_cap           <dbl> 35770.78, 36158.59, 37620.21, 39042.96, 39782.93~
## $ region                <chr> "Europe & Central Asia", "Europe & Central Asia"~
## $ income                <chr> "High income", "High income", "High income", "Hi~
```

#### Small multiples

Small multiples is a combination of small and simple plots showing one variable for different individuals..


```r
life_expectancy_small <- wdi_clean %>% 
    filter(country %in% c("Argentina", "Bolivia", "Brazil",
                          "Belize", "Canada", "Chile"))

ggplot(data = life_expectancy_small,
       aes(year, life_expectancy)) +
    geom_line(size = 1) +
    facet_wrap(vars(country))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Small multiples-1.png" width="672" />

* **A minimalist Small multiples**


```r
ggplot(data = life_expectancy_small,
       aes(x = year, y = life_expectancy)) +
    geom_line(size = 1) +
    facet_wrap(vars(country), scales = "free_y") +
    theme_void() +
    theme(strip.text = element_text(face = "bold")) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/minimalist small multiples-1.png" width="672" />

*Good luck discovering what this plot is showing. But it is beautiful.*

If we so desire, we can plot the data for an entire region.


```r
wdi_clean %>% 
    filter(region == "Middle East & North Africa") %>% 
    
    #Building the plot
    
    ggplot(aes(x = year, y = life_expectancy)) +
    geom_line(size = 1) +
    facet_wrap(vars(country), scales = "free_y", nrow = 3) +
    theme_void() +
    theme(strip.text = element_text(face = "bold"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Middle East & North Africa plot-1.png" width="672" />


Now let's use the [geofacet package](https://hafen.github.io/geofacet/) for ploting our data in a map format.



```r
wdi_clean %>% 
    filter( region == "Europe & Central Asia" & country != "Malta") %>% 
    
    ggplot(aes(x = year, y = life_expectancy)) +
    geom_line(size = 1) +
    facet_geo(vars(country), grid = "eu_grid1", scales = "free_y")+
    labs(x = NULL, y = NULL, title = "Life expectancy from 1995-2015",
         caption = "Source: The World Bank (SP.DYN.LE))>IN") +
    theme_minimal() +
    theme(strip.text = element_text(face = "bold"),
          plot.title = element_text(face = "bold"),
          axis.text.x = element_text(angle = 45, hjust = 1),
          axis.text.y = element_blank()) +
    scale_x_continuous(breaks = NULL)
```

```
## Some values in the specified facet_geo column 'country' do not match
##   the 'name' column of the specified grid and will be removed: Andorra,
##   Albania, Armenia, Azerbaijan, Bosnia and Herzegovina, Belarus,
##   Switzerland, Faroe Islands, Georgia, Gibraltar, Greenland, Isle of
##   Man, Iceland, Channel Islands, Kyrgyz Republic, Kazakhstan,
##   Liechtenstein, Monaco, Moldova, Montenegro, North Macedonia, Norway,
##   Serbia, Russian Federation, Slovak Republic, San Marino, Tajikistan,
##   Turkmenistan, Turkey, Ukraine, Uzbekistan, Kosovo
```

<img src="{{< blogdown/postref >}}index_files/figure-html/life_expectancy for Europe and Asia-1.png" width="672" />

#### Sparklines

Sparklines are very small line or bar charts.


```r
india_co2 <- wdi_clean %>% 
    filter(country == "India") 

plot_india <- ggplot(india_co2, aes(x = year, y = co2_emissions)) +
    geom_line() +
    theme_void()

plot_india
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Spark 1-1.png" width="672" />

```r
ggsave("output/india_co2.pdf", plot_india, width = 0.5, height = 0.15, units = "in")
ggsave("output/india_co2.png", plot_india, width = 0.5, height = 0.15, units = "in")
```


```r
china_co2 <- wdi_clean %>% 
    filter(country == "China")

plot_china <- ggplot(china_co2, aes(x = year, y = co2_emissions)) +
    geom_line() +
    theme_void()


## Saving the graphs

ggsave("output/china_co2.pdf", plot_china, width = 0.5, height = 0.1, units = "in")
ggsave("output/china_co2.png", plot_china, width = 0.5, height = 0.1, units = "in")
```

Now, we can use the images we saved in our text.

China:<img src="output/china_co2.png" width="150" /> 
And this is the evolution of co2 emission in India: <img src="output/india_co2.png" width="150" />



#### Slopegraphs


```r
gdp_south_asia <- wdi_clean %>% 
    filter(region == "South Asia") %>% 
    
    # Now, we filter the start and end point of the slope graph.
    filter(year %in% c(1995, 2015)) %>%
    
    #look at each country individually
    group_by(country) %>% 
    
    # Remove countries with missing data
    # As the data is grouped by country !any() evaluates if there is NA for 1995 or 2015 for each country, and if so it drops it.
    filter(!any(is.na(gdp_per_cap))) %>% 
    ungroup() %>% 
    
    mutate(year = factor(year)) %>% 
    
    #Create some labels
    # If the year is 1995, format it like "Country name: $GDP".
    #  If the year is 2015, format it like "$GDP"
    mutate(label_first = ifelse(year == 1995, paste0(country, ": ", dollar(round(gdp_per_cap))), NA),
           label_last = ifelse(year == 2015, dollar(round(gdp_per_cap)), NA))


# Now we can plot the filtered data

# We need to use group = country so the line goes through the two extremes

ggplot(gdp_south_asia, aes (x = year, y = gdp_per_cap, group = country, color = country)) +
    geom_line(size = 1.5)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Building a slope graph for gdp South Asia-3.png" width="672" />

While this is good, it would be better to have the name of it country in it line.
In the example code, there are four steps until the final graph. I will replicate only the final version. Each step is discussed in the comments in the code


```r
gdp_south_asia %>% 
    ggplot(aes(x = year, y = gdp_per_cap, group = country, color = country)) +
    
    #makes the line thicker
    geom_line(size = 1.5) +
    
    #Now we add the labels in the graph.
    # We could use geom_text but then the names would overlap the line. So we use geom_text_repel that repels the label from the line
    geom_text_repel( aes(label = label_first), direction = "y", nudge_x = -1, seed = 30) +
    geom_text_repel(aes(label = label_last), direction = "y", nudge_x = 1, seed = 30) +
    
    # now we can drop the info about the line colors
    guides(color = FALSE) +
    
    # Pick a better color palette
    scale_color_viridis_d(option = "magma", end = 0.9) +
    labs( x = NULL, y = "GDP per capita") +
    theme_gray()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Final Slope chart-1.png" width="672" />


A slope graph is a neat way of comparing the evolution of a metric in time. Nice tool to have in your toolbox.

#### Bump Charts

We can build bump charts to show movement in rank through the years.


```r
sa_co2 <- wdi_clean %>% 
    filter(region == "South Asia") %>% 
    filter(year >= 2004, year < 2015) %>% 
    group_by(year) %>% 
    # Creates a rank based on the column co2_emissions
    mutate(rank = rank(co2_emissions))

sa_co2 %>% 
    ggplot(aes(x = year, y = rank, color = country)) +
    geom_line() +
    geom_point() +
    scale_y_reverse(breaks = 1:8)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Bump Charts-1.png" width="672" />
Bumps graphs are nice. What we can do to improve is to add the name of each country directly in the graph.



```r
sa_co2 %>% 
    ggplot(aes(x = year, y = rank, color = country)) +
    geom_line(size = 2) +
    geom_point(size = 4) +
    geom_text(data = filter(sa_co2, year == 2004),
              aes(label = iso2c, x = 2003.25),
              fontface = "bold"
              ) +
    geom_text(data = filter(sa_co2, year == 2014),
              aes(label = iso2c, x = 2014.75),
              fontface = "bold") +
    guides(color = FALSE) +
    scale_y_reverse(breaks = 1:8) +
    scale_x_continuous(breaks = 2004 : 2014) +
    scale_colour_viridis_d(option = "magma", begin = 0.2, end = 0.9) +
    labs(x = NULL, y = "Rank", title = "Afeganisthan was the most polluting country in South Asia in 2014") +
    theme_minimal() +
    theme(panel.grid.major.y = element_blank(),
          panel.grid.minor.y = element_blank(),
          panel.grid.minor.x = element_blank())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Bump Graph 2-1.png" width="672" />

As an extra I will add the flags. Just because.


```r
library(ggflags)

# Preparing the data with the positions for the flags.
c_codes <- unique(sa_co2$iso2c)

#We must use tolower because ggflags understand only the code in lowercase letters
flags_countries_start <- tibble(x = 2003.5, y = 1:8,
                          country = tolower(c_codes))
flags_countries_end <- tibble(x = 2014.5, y = 1:8,
                          country = tolower(c_codes))

#Almost the same code we used for labels, but using ggflags instead.

sa_co2 %>% 
    ggplot(aes(x = year, y = rank, color = country)) +
    geom_line(size = 2) +
    geom_point(size = 4) +
    geom_flag(data = flags_countries_start, aes(x = x, y = y, country = country, size = 8)) +
    geom_flag(data = flags_countries_end, aes(x = x, y = y, country = country, size = 8)) +
    guides(color = FALSE, size = FALSE) +
    scale_y_reverse(breaks = 1:8) +
    scale_x_continuous(breaks = 2004 : 2014) +
    scale_colour_viridis_d(option = "magma", begin = 0.2, end = 0.9) +
    labs(x = NULL, y = NULL, title = "Afeganisthan was the most polluting country in South Asia in 2014") +
    theme_minimal() +
    theme(panel.grid.major.y = element_blank(),
          panel.grid.minor.y = element_blank(),
          panel.grid.minor.x = element_blank())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/ggflags-1.png" width="672" />




### Exercise

#### Small Multiples

Use data from the US Bureau of Labor Statistics (BLS) to show the trends in employment rate for all 50 states between 2006 and 2016. What stories does this plot tell? Which states struggled to recover from the 2008–09 recession?


```r
library(tidyverse)
library(ggflags)
library(geofacet)   #map-shaped facets
library(scales)     # for using scale functions
library(ggrepel)
library(skimr)

unemployment <- read_csv("C:/Users/tiago/OneDrive/datasets/data_viz-course/unemployment.csv")
```

```
## Rows: 6732 Columns: 8
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## chr  (4): state, month_name, region, division
## dbl  (3): year, month, unemployment
## date (1): date
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
skim_without_charts(unemployment)
```


Table: (\#tab:Loading the packages and the data)Data summary

|                         |             |
|:------------------------|:------------|
|Name                     |unemployment |
|Number of rows           |6732         |
|Number of columns        |8            |
|_______________________  |             |
|Column type frequency:   |             |
|character                |4            |
|Date                     |1            |
|numeric                  |3            |
|________________________ |             |
|Group variables          |None         |


**Variable type: character**

|skim_variable | n_missing| complete_rate| min| max| empty| n_unique| whitespace|
|:-------------|---------:|-------------:|---:|---:|-----:|--------:|----------:|
|state         |         0|             1|   4|  20|     0|       51|          0|
|month_name    |         0|             1|   3|   9|     0|       12|          0|
|region        |         0|             1|   4|   9|     0|        4|          0|
|division      |         0|             1|   7|  18|     0|        9|          0|


**Variable type: Date**

|skim_variable | n_missing| complete_rate|min        |max        |median     | n_unique|
|:-------------|---------:|-------------:|:----------|:----------|:----------|--------:|
|date          |         0|             1|2006-01-01 |2016-12-01 |2011-06-16 |      132|


**Variable type: numeric**

|skim_variable | n_missing| complete_rate|    mean|   sd|     p0|     p25|    p50|     p75|   p100|
|:-------------|---------:|-------------:|-------:|----:|------:|-------:|------:|-------:|------:|
|year          |         0|             1| 2011.00| 3.16| 2006.0| 2008.00| 2011.0| 2014.00| 2016.0|
|month         |         0|             1|    6.50| 3.45|    1.0|    3.75|    6.5|    9.25|   12.0|
|unemployment  |         0|             1|    6.29| 2.20|    2.4|    4.60|    5.9|    7.80|   14.6|

We can see that there are no missing values in the dataset.
Besides that, we see that we have data from 2006 to 2016. 

Now, we create a small multiple to visualize the evolution of this data


```r
unemployment %>% 
    ggplot(aes(x = date, y= unemployment)) +
    geom_line() +
    facet_wrap(vars(state), nrow = 5) +
    
    labs(x = NULL, y= "Unemployment rate", 
         title = "Unemployment rate for US: 2006 to 2016") +
    theme_void() +
    # We can use the function theme to fine tune our plot
    theme(plot.title = element_text(face = "bold", color = "blue", family = "sans"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Unemp. in time-1.png" width="960" />

As we can see, some states as Vermont, Nebraska, and South Dakota almost did not fell the impact fo the 2008 crisis.



facet_geo(vars(country), grid = "eu_grid1", scales = "free_y")+


```r
unemployment %>% 
    ggplot(aes(x = date, y= unemployment)) +
    geom_line(aes(color = region), size = 1) +
    facet_geo(vars(state), grid = "us_state_grid2") +
    
    labs(x = NULL, y= NULL, 
         title = "Unemployment rate for US: 2006 to 2016",
         caption = "Source: US Bureau of Labor Statistics") +
    theme_minimal() +
    # We can use the function theme to fine tune our plot
    theme(plot.title = element_text(face = "bold", family = "sans"),
          axis.text.x = element_blank(),
          axis.text.y = element_blank(),
          # Makes all the text in the graph bold
          strip.text = element_text(face = "bold"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/geofacet-1.png" width="1248" />

#### Slope Charts

Use data from the BLS to create a slopegraph that compares the unemployment rate in January 2006 with the unemployment rate in January 2009, either for all 50 states at once (good luck with that!) or for a specific region or division. Make sure the plot doesn’t look too busy or crowded in the end.

I will work with the South and Midwest. Just because.


```r
regions <- c("South", "Midwest")

unemployment %>% 
    
    #First select the desired regions
    filter(region %in% regions) %>% 
    
    #Filter for years
    filter(year == 2006 | year == 2009) %>%
    
    #Select only January
    filter(month == 1) %>% 
    
    #For plotting the year in the x-axis as a separate element, we make it a factor
    mutate(year = factor(year)) %>%
    
    # Make labels for the plots
    mutate( label_first = ifelse(year == 2006, paste0(state,": ",unemployment), NA),
            label_end = ifelse(year == 2009, paste0(state,": ",unemployment),NA)
            ) %>% 
    
    ggplot(aes(x = year, y = unemployment, group = state, color = state)) +
    geom_line(size = 1) +
    # direction = "y" makes the labs to move only up. nudge_x = -1, makes it move -1 point in the x axis
    geom_text_repel(aes(label = label_first), direction = "y", nudge_x = -1, seed = 30) +
    geom_text_repel(aes(label = label_end), direction = "y", nudge_x = 1, seed = 30) +
    facet_wrap(vars(region), nrow = 1) +
    guides(color = "none") +
    labs(title = "Michigan had the highest unemployment rate in the aftermath of the financial crisis",
         y = "Unemployment rate",
         x = NULL) +
    theme(axis.title = element_text(face = "bold", family = "sans", size = 10),
          strip.text = element_text(face = "bold")) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Slope graph-1.png" width="1152" />
For avoiding confusion, we had to make the plot extra large.

In January 2009 all of the analyzed states had a bigger unemployment rate than they had in 2009. There we some states as South Dakota, Nebraska, and Mississippi that had a small rise of less that 1 p.p. But it is important to note that this last country already had a high unemployment rate before the crisis. 
Of the analyzed states Michigan was the hardest affected. Even though it already had the highest unemployment rate in the Midwest, its unemployment rate was 4.1 p.p. higher in 2009 that it was in 2006.



