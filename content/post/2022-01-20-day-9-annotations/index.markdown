---
title: Day 9 - Annotations
author: Tiago Irineu
date: '2022-01-20'
slug: day-9-annotations
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2022-01-20T07:50:15-03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


## Annotations

No reflections for this day.


### Example





```r
library(tidyverse)  # For ggplot, dplyr, and friends
library(WDI)        # Get data from the World Bank
library(ggrepel)    # For non-overlapping labels

library(ggtext)

indicators <- c("SP.POP.TOTL",     # Population
                "EN.ATM.CO2E.PC",  # CO2 emissions
                "NY.GDP.PCAP.KD")  # GDP per capta

wdi_co2_raw <- WDI(country = "all", indicators, extra = TRUE, 
                   start = 1995, end = 2015)
```

[Github page for the ggtext package](https://github.com/wilkelab/ggtext).



```r
wdi_clean <- wdi_co2_raw %>% 
  filter(region != "Aggregates") %>% 
  select(iso2c, iso3c, country, year, 
         population = SP.POP.TOTL,
         co2_emissions = EN.ATM.CO2E.PC, 
         gdp_per_cap = NY.GDP.PCAP.KD, 
         region, income)
```



```r
co2_rankings <- wdi_clean %>% 
  # Get rid of smaller countries
  filter(population > 200000) %>% 
  # Only look at two years
  filter(year %in% c(1995, 2014)) %>% 
  # Get rid of all the rows that have missing values in co2_emissions
  drop_na(co2_emissions) %>% 
  # Look at each year individually and rank countries based on their emissions that year
  group_by(year) %>% 
  mutate(ranking = rank(co2_emissions)) %>% 
  ungroup() %>% 
  # Only select a handful of columns, mostly just the newly created "ranking"
  # column and some country identifiers
  select(iso3c, country, year, region, income, ranking) %>% 
  # Right now the data is tidy and long, but we want to widen it and create
  # separate columns for emissions in 1995 and in 2014. pivot_wider() will make
  # new columns based on the existing "year" column (that's what `names_from`
  # does), and it will add "rank_" as the prefix, so that the new columns will
  # be "rank_1995" and "rank_2014". The values that go in those new columns will
  # come from the existing "ranking" column
  pivot_wider(names_from = year, names_prefix = "rank_", values_from = ranking) %>% 
  # Find the difference in ranking between 2014 and 1995
  mutate(rank_diff = rank_2014 - rank_1995) %>% 
  # Remove all rows where there's a missing value in the rank_diff column
  drop_na(rank_diff) %>% 
  # Make an indicator variable that is true of the absolute value of the
  # difference in rankings is greater than 25. 25 is arbitrary here—that just
  # felt like a big change in rankings
  mutate(big_change = ifelse(abs(rank_diff) >= 25, TRUE, FALSE)) %>% 
  # Make another indicator variable that indicates if the rank improved by a
  # lot, worsened by a lot, or didn't change much. We use the case_when()
  # function, which is like a fancy version of ifelse() that takes multiple
  # conditions. This is how it generally works:
  #
  # case_when(
  #  some_test ~ value_if_true,
  #  some_other_test ~ value_if_true,
  #  TRUE ~ value_otherwise
  #)
  mutate(better_big_change = case_when(
    rank_diff <= -25 ~ "Rank improved",
    rank_diff >= 25 ~ "Rank worsened",
    TRUE ~ "Rank changed a little"
  ))
```


```r
# These three functions make it so all geoms that use text, label, and
# label_repel will use IBM Plex Sans as the font. Those layers are *not*
# influenced by whatever you include in the base_family argument in something
# like theme_bw(), so ordinarily you'd need to specify the font in each
# individual annotate(geom = "text") layer or geom_label() layer, and that's
# tedious! This removes that tediousness.
update_geom_defaults("text", list(family = "IBM Plex Sans"))
update_geom_defaults("label", list(family = "IBM Plex Sans"))
update_geom_defaults("label_repel", list(family = "IBM Plex Sans"))

ggplot(co2_rankings,
       aes(x = rank_1995, y = rank_2014)) +
  # Add a reference line that goes from the bottom corner to the top corner
  annotate(geom = "segment", x = 0, xend = 175, y = 0, yend = 175) +
  # Add points and color them by the type of change in rankings
  geom_point(aes(color = better_big_change)) +
  # Add repelled labels. Only use data where big_change is TRUE. Fill them by
  # the type of change (so they match the color in geom_point() above) and use
  # white text
  geom_label_repel(data = filter(co2_rankings, big_change == TRUE),
                   aes(label = country, fill = better_big_change),
                   color = "white") +
  # Add notes about what the outliers mean in the bottom left and top right
  # corners. These are italicized and light grey. The text in the bottom corner
  # is justified to the right with hjust = 1, and the text in the top corner is
  # justified to the left with hjust = 0
  annotate(geom = "text", x = 170, y = 6, label = "Outliers improving", 
           fontface = "italic", hjust = 1, color = "grey50") +
  annotate(geom = "text", x = 2, y = 170, label = "Outliers worsening", 
           fontface = "italic", hjust = 0, color = "grey50") +
  # Add mostly transparent rectangles in the bottom right and top left corners
  annotate(geom = "rect", xmin = 0, xmax = 25, ymin = 0, ymax = 25, 
           fill = "#2ECC40", alpha = 0.25) +
  annotate(geom = "rect", xmin = 150, xmax = 175, ymin = 150, ymax = 175, 
           fill = "#FF851B", alpha = 0.25) +
  # Add text to define what the rectangles abovee actually mean. The \n in
  # "highest\nemitters" will put a line break in the label
  annotate(geom = "text", x = 40, y = 6, label = "Lowest emitters", 
           hjust = 0, color = "#2ECC40") +
  annotate(geom = "text", x = 162.5, y = 135, label = "Highest\nemitters", 
           hjust = 0.5, vjust = 1, lineheight = 1, color = "#FF851B") +
  # Add arrows between the text and the rectangles. These use the segment geom,
  # and the arrows are added with the arrow() function, which lets us define the
  # angle of the arrowhead and the length of the arrowhead pieces. Here we use
  # 0.5 lines, which is a unit of measurement that ggplot uses internally (think
  # of how many lines of text fit in the plot). We could also use unit(1, "cm")
  # or unit(0.25, "in") or anything else
  annotate(geom = "segment", x = 38, xend = 20, y = 6, yend = 6, color = "#2ECC40", 
           arrow = arrow(angle = 15, length = unit(0.5, "lines"))) +
  annotate(geom = "segment", x = 162.5, xend = 162.5, y = 140, yend = 155, color = "#FF851B", 
           arrow = arrow(angle = 15, length = unit(0.5, "lines"))) +
  # Use three different colors for the points
  scale_color_manual(values = c("grey50", "#0074D9", "#FF4136")) +
  # Use two different colors for the filled labels. There are no grey labels, so
  # we don't have to specify that color
  scale_fill_manual(values = c("#0074D9", "#FF4136")) +
  # Make the x and y axes expand all the way to the edges of the plot area and
  # add breaks every 25 units from 0 to 175
  scale_x_continuous(expand = c(0, 0), breaks = seq(0, 175, 25)) +
  scale_y_continuous(expand = c(0, 0), breaks = seq(0, 175, 25)) +
  # Add labels! There are a couple fancy things here.
  # 1. In the title we wrap the 2 of CO2 in the HTML <sub></sub> tag so that the
  #    number gets subscriped. The only way this will actually get parsed as 
  #    HTML is if we tell the plot.title to use element_markdown() in the 
  #    theme() function, and element_markdown() comes from the ggtext package.
  # 2. In the subtitle we bold the two words **improved** and **worsened** using
  #    Markdown asterisks. We also wrap these words with HTML span tags with 
  #    inline CSS to specify the color of the text. Like the title, this will 
  #    only be processed and parsed as HTML and Markdown if we tell the p
  #    lot.subtitle to use element_markdown() in the theme() function.
  labs(x = "Rank in 1995", y = "Rank in 2014",
       title = "Changes in CO<sub>2</sub> emission rankings between 1995 and 2014",
       subtitle = "Countries that <span style='color: #0074D9'>**improved**</span> or <span style='color: #FF4136'>**worsened**</span> more than 25 positions in the rankings highlighted",
       caption = "Source: The World Bank.\nCountries with populations of less than 200,000 excluded.") +
  # Turn off the legends for color and fill, since the subtitle includes that
  guides(color = FALSE, fill = FALSE) +
  # Use theme_bw() with IBM Plex Sans
  theme_bw(base_family = "IBM Plex Sans") +
  # Tell the title and subtitle to be treated as Markdown/HTML, make the title
  # 1.6x the size of the base font, and make the subtitle 1.3x the size of the
  # base font. Also add a little larger margin on the right of the plot so that
  # the 175 doesn't get cut off.
  theme(plot.title = element_markdown(face = "bold", size = rel(1.6)),
        plot.subtitle = element_markdown(size = rel(1.3)),
        plot.margin = unit(c(0.5, 1, 0.5, 0.5), units = "lines"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Plot-1.png" width="1152" />

### Exercises

The exercise for today is to create a plot and then annotate it. The code goes below.



```r
raw_adol_fertility <- WDI(country = "all",
                          indicator = c("adol_fertility" = "SP.ADO.TFRT",
                                        "pib_capita" = "NY.GDP.PCAP.KD"),
                          extra = TRUE)

glimpse(raw_adol_fertility)
```

```
## Rows: 16,226
## Columns: 12
## $ iso2c          <chr> "1A", "1A", "1A", "1A", "1A", "1A", "1A", "1A", "1A", "~
## $ country        <chr> "Arab World", "Arab World", "Arab World", "Arab World",~
## $ year           <int> 2006, 2010, 2007, 2005, 2001, 2002, 2008, 2009, 1991, 1~
## $ adol_fertility <dbl> 50.83960, 49.96690, 50.51954, 51.21296, 53.30941, 52.29~
## $ pib_capita     <dbl> 5384.491, 5720.204, 5500.379, 5192.348, 4681.965, 4607.~
## $ iso3c          <chr> "ARB", "ARB", "ARB", "ARB", "ARB", "ARB", "ARB", "ARB",~
## $ region         <chr> "Aggregates", "Aggregates", "Aggregates", "Aggregates",~
## $ capital        <chr> "", "", "", "", "", "", "", "", "", "", "", "", "", "",~
## $ longitude      <chr> "", "", "", "", "", "", "", "", "", "", "", "", "", "",~
## $ latitude       <chr> "", "", "", "", "", "", "", "", "", "", "", "", "", "",~
## $ income         <chr> "Aggregates", "Aggregates", "Aggregates", "Aggregates",~
## $ lending        <chr> "Aggregates", "Aggregates", "Aggregates", "Aggregates",~
```


```r
adol_fertility <- raw_adol_fertility %>% 
    select(iso2c, country, year, adol_fertility, pib_capita, region, income) %>% 
    filter(region != "Aggregates")

glimpse(adol_fertility)
```

```
## Rows: 13,176
## Columns: 7
## $ iso2c          <chr> "AD", "AD", "AD", "AD", "AD", "AD", "AD", "AD", "AD", "~
## $ country        <chr> "Andorra", "Andorra", "Andorra", "Andorra", "Andorra", ~
## $ year           <int> 1975, 1976, 1977, 1973, 1974, 1988, 1972, 1990, 1978, 1~
## $ adol_fertility <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,~
## $ pib_capita     <dbl> 36246.68, 36175.32, 36081.66, 37123.26, 37504.74, 29854~
## $ region         <chr> "Europe & Central Asia", "Europe & Central Asia", "Euro~
## $ income         <chr> "High income", "High income", "High income", "High inco~
```



```r
fert_plot <- adol_fertility %>% 
    mutate(should_be_labeled = ifelse(adol_fertility > 150 & log10(pib_capita) > 3.5,
           TRUE, FALSE)) %>% 
    filter(year == 2015)

fert_plot %>% 
    ggplot(aes(x = log10(pib_capita), y = adol_fertility)) +
    geom_point(aes(color = income),
               alpha = 0.5) +
    geom_text_repel(data = filter(fert_plot, should_be_labeled == TRUE), 
                    aes(label = country),
                    box.padding = 2, # creates the line from the point to label
                    nudge_y = 1 ,
                    nudge_x = 0.01
                    ) +
    annotate(geom = "rect", 
             xmin = 3.5, xmax = 4.2,
             ymin = 150, ymax = 175,
             fill = "#FF851B", alpha = 0.2) +
    
    annotate(geom = "label", 
             x = 3.9, y = 125,
             label = "Not so poor, but high fertility") +
    annotate(geom = "segment",
             x = 3.8, xend = 3.8,
             y = 132, yend = 147.5,
             arrow = arrow(
                 length = unit(0.1, "in"))
             ) + 
    
    labs(title = "In richer countries there are less adolescent pregnancies",
         subtitle = "Year: 2015",
         x = "Log of Pib per capita",
         y = "Fertility rate: 15 to 19 years old",
         caption = "Source: The World Bank") +
    
    theme_minimal(base_family = "IBM Plex Sans") +
    
    #Below I use the theme function to adjust specific aspect of the plot
    
    theme(plot.title = element_markdown(face = "bold", size = rel(1.2), family = "sans"),
          plot.subtitle = element_markdown(face = "bold", size = rel(1), family = "AppleGothic"),
          axis.title.x = element_text(family = "AppleGothic", face = "bold"),
          axis.title.y = element_text(family = "AppleGothic", face = "bold"),
          legend.title = element_blank(),
          legend.text = element_text(family = "AppleGothic"),
          
          plot.caption = element_markdown(size = rel(0.7), family = "AppleGothic"),
          panel.grid = element_blank(),
          panel.border =element_rect(fill = NA, size = 2)
          )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/fertility x pib per capita-1.png" width="672" />

End of lesson














