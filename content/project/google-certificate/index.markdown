---
date: "2021-10-23"
external_link: ""


summary: Bike sharing case study. Capstone project for the Google Data Analytics Certificate.
tags:
- data analysis
- R

title: How Does a Bike-Share Navigate Spedy Success? - Google Certificate Capstone Project

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""
---



# Introduction

This is my case study for data analysis Google Certificate. I divide this in two parts.
First, I will discuss the findings and the insight. In the second part I will go into the details of the analysis. 

# Scenario

In this project we assume the role of a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company's future depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members

Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a
very good chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic
program and have chosen Cyclistic for their mobility needs.

As such, our assignment is to answer the following question: **How do annual members and casual riders use Cyclistic bikes differently?**

## Deliverables

Our analysis must create the following outcomes to be considered successful. 

1. A clear statement of the business task
2. A description of all data sources used
3. Documentation of any cleaning or manipulation of data
4. A summary of your analysis
5. Supporting visualizations and key findings
6. Your top three recommendations based on your analysis


For generating these deliverables we are going to follow the six steps of data analysis, as taugh in the course: *ask*, *prepare*, *process*, *analyze*,*share*,*and act*.

Outline of our discussion:

1. Statement of Business Task.
2. The data we used and its limitation.
3. Discussion of our main results.
4. Summary of findings and recommendations.
5. Detailed documentation of data manipulation.

## Ask - Statement of the business task

Cyclist has three different pricing plans: single-ride passes, full-day passes, and annual memberships.
Our task is to identify the differences between annual members and the other two customer groups, that we will define as casual riders.

As we analyze the difference between both group of riders, we should give at least three recommendations that can help the company in achieving its goals.

## The data we used and its limitations

We used 12 months from Cyclistic historical trip data. Ranging from October 2020, to September 2021.
Is our data good? For acessing this let's see if this data **ROCCC**.

ROCCC is a acronym for Reliable, Original, Comprehensive, Current, and Cited, as the gold standard of data.

While our data is no cited, because it's internal data. I understand that it's a good dataset for our usage. It is reliable. It does have some mistakes but they amount to an insignificant part of all observations. It covers the last 12 months, therefore it is current. It covers all rides in the period, so I don't expect any specific bias. After the cleaning, we ended up with more than 5 mln. observations. More than enough for almost any analysis.

While I believe the data is good enough, it does have some limitations such as:

* One possible source of bias is that we are using data collected during a pandemic. This probably affected some potential riders. For example, there were no in-site classes for most of this period and also some companies were in home office policy. 

* For anonymization concerns we are not able to identify the riders. It seems necessary to find ways to respect anonymization but get more information about our riders. I will come back to this point in the Recommendation section.

These question must be addressed and we will further discuss them in the recommendation section. Regardless of these points, I do think that the data we have is good enough for creating a useful analysis for the company.


## Preparing and processing the data

All our data work, from preparing the data to the visualizations, will be done using R and RStudio. In particular, I will use the tidyverse to wrangle the data, ggplot2 for creating the visualizations, and blogdown for creating the site where this analysis is being shared. R all the way down.


As previously mentioned I used data from 2020 October, to 2021 September. You can access the data [here](https://divvy-tripdata.s3.amazonaws.com/index.html). As it is in .zip format, it's necessary to download it locally, extract and then import to R. These steps I cannot record. All the others steps and documented below.

We first import the data for the 12 months we are going to analyze:


```r
library(tidyverse)
library(lubridate) # For working with dates
```



```r
td_2020_10 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202010-divvy-tripdata.csv")
td_2020_11 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202011-divvy-tripdata.csv")
td_2020_12 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202012-divvy-tripdata.csv")
td_2021_01 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202101-divvy-tripdata.csv")
td_2021_02 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202102-divvy-tripdata.csv")
td_2021_03 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202103-divvy-tripdata.csv")
td_2021_04 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202104-divvy-tripdata.csv")
td_2021_05 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202105-divvy-tripdata.csv")
td_2021_06 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202106-divvy-tripdata.csv")
td_2021_07 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202107-divvy-tripdata.csv")
td_2021_08 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202108-divvy-tripdata.csv")
td_2021_09 <-  read_csv("C:/Users/tiago/OneDrive/google_certificate/raw_data/202109-divvy-tripdata.csv")
```

the function *read_csv* always print some information about the data it just read. If we read them carefully we will see that in 2020 December, the type for the variables in the columns *start_station_id* and *end_station_id* changed. For October and November they are numeric, but later it was encoded as character. 

For keeping consistency we will coerce the values for October and November to characters.


```r
td_2020_10 <- td_2020_10 %>% mutate(start_station_id = as.character(start_station_id), end_station_id = as.character(end_station_id))
td_2020_11 <- td_2020_11 %>% mutate(start_station_id = as.character(start_station_id), end_station_id = as.character(end_station_id))
```

Now, that all the twelve datasets have consistent data for each column we can combine them in one single dataset.


```r
trip_data <- bind_rows(td_2020_10, td_2020_11, td_2020_12,td_2021_01,td_2021_02, td_2021_03,        #Combining the data
                       td_2021_04, td_2021_05, td_2021_06, td_2021_07, td_2021_08, td_2021_09)

rm(td_2020_10, td_2020_11, td_2020_12,td_2021_01,td_2021_02, td_2021_03,      #Removing the individual data sets from the env.
   td_2021_04, td_2021_05, td_2021_06, td_2021_07, td_2021_08, td_2021_09)
```



Now let's coerce the dataframe to a tibble. Tibbles are common data frames but with some adjustments for working better with the tidyverse and also showing better in the screen. You can find more information [here](https://r4ds.had.co.nz/tibbles.html).




```r
trip_data <- as_tibble(trip_data)
colnames(trip_data)
```

```
##  [1] "ride_id"            "rideable_type"      "started_at"        
##  [4] "ended_at"           "start_station_name" "start_station_id"  
##  [7] "end_station_name"   "end_station_id"     "start_lat"         
## [10] "start_lng"          "end_lat"            "end_lng"           
## [13] "member_casual"
```

We still have to work a lot with our data before beginning our analysis. Regardless of that, I want to have an idea about the proportion of members and casual rides.


```r
n = length(trip_data$member_casual)  # Gets the total number of observations

trip_data %>% 
  group_by(member_casual) %>% 
  summarise(Proportion = n()/n)  # The function n() counts the number of elements for each group.
```

```
## # A tibble: 2 x 2
##   member_casual Proportion
##   <chr>              <dbl>
## 1 casual             0.459
## 2 member             0.541
```

So we know that 54% of members are casual and 46% are casual. Let's work more on the data, and if we end up dropping any row, we calculate this again.


### Creating new variables

An integral part of data preparation is creating new variables by using current data. We can apply transformation and calculations, in a way that create variables there are useful in posterior analysis. 

Along the way, we also have to look for any inconsistent data and then decide what to do about it. Let's go.

First, as we have the hour that the ride began and hour it finished, we can create a variable for lenght ride.

As we have when the rides began and when they finished, we can calculate how long they were. Let's do this.



```r
trip_data <- trip_data %>% 
  mutate("ride_lenght" = ended_at - started_at)
```


Let's now get an overall idea about this new variable:



```r
trip_data %>% 
  select(ride_lenght) %>% 
  summarise("mean_ride" = mean(ride_lenght), "shortest ride" = min(ride_lenght), "longest ride" = max(ride_lenght))
```

```
## # A tibble: 1 x 3
##   mean_ride     `shortest ride` `longest ride`
##   <drtn>        <drtn>          <drtn>        
## 1 1237.408 secs -1742998 secs   3356649 secs
```
Unless somebody was the Flash and could go back in time, this info show to us that there are some erros in the data. As min_ride is negative, it is telling us that there are negative entries. Let's find out how many entries fall in this situation.


```r
sum(trip_data$ride_lenght <=0) #By summing a logical vector you get the number of values that evaluates to FALSE
```

```
## [1] 3762
```
In a dataset with more that 5 mln. observations, it seems ok to me to drop less than 4.000 entries. 
This has potential for creating a problem, if the factor that explain the mistake is one we are interested in. But it does not seems to be the case, so let's drop these observation.


As a rule, whenever I drop a row I will create a dataset with a new name. 
I do this because if there is any problem, we can easily go back to the previous dataframe.


```r
# Here I use dplyr filter call to drop all the rows with ride_lenght lower than 0
trip_data_2 <- trip_data %>% 
  filter(!ride_lenght <= 0)
```



















