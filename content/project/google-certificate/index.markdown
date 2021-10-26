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

output:
  blogdown::html_page:
    toc: true

---



{{< toc >}}

# Introduction

This is my case study for the Google Data Analytics Certificate.
This Certificate is make of 7 mandatory courses and this optional Capstone Project. You can find more information about this certificate [here](https://www.coursera.org/professional-certificates/google-data-analytics).


# Scenario

In this project we assume the role of a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company's future depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members

Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a
very good chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs. The data analysis team has the mission of working with past data and finding insights that can be used to convert casual users to annual members.

My assignment as the junior data analyst is to answer the following question: **How do annual members and casual riders use Cyclistic bikes differently?**

## Deliverables

Our analysis must create the following outcomes to be considered successful. 

1. A clear statement of the business task
2. A description of all data sources used
3. Documentation of any cleaning or manipulation of data
4. A summary of the analysis analysis
5. Supporting visualizations and key findings
6. Your top three recommendations based on your analysis


For generating these deliverables we are going to follow the six steps of data analysis, as taugh in the course: *ask*, *prepare*, *process*, *analyze*,*share*,*and act*.


# Business Task - Ask Phase

Cyclist has three different pricing plans: single-ride passes, full-day passes, and annual memberships.
Our task is to identify the differences between annual members and the other two customer groups, that we will define as casual riders.

As we analyze the difference between both group of riders, we should give at least three recommendations that can help the company in achieving its goals.

Our main stakeholders are:

* Lily Moreno, our manager and the director of marketing. She designed the goal of developing strategies to convert casual users to annual riders and assigned our specific task. 

* Cyclistic executive team. They will decide if they approve the recommended marketing program. They are notoriously detail-oriented.

# The data we used and its limitations - Prepare phase

We used 12 months from Cyclistic historical trip data. Ranging from October 2020, to September 2021. The data is available in this [repository](https://divvy-tripdata.s3.amazonaws.com/index.html).

The data is stored in .zip files. This create a small nuisance that is the fact that we will have to download it locally to our working station. Besides that, the data seems to be good.

* It is current. They have been updating it in a monthly basis for the last years. This will help us to have a better idea on how people are using our bikes now.

* The data seems to be reliable. While we still have to check the data for possible errors, the fact that it is collected by our own system gives a certain confidence that the data is really showing what it should.

* The data is relevant. It have enough variables that will enable us to get an idea of the differences between casual riders and annual members. After the analysis, I concluded that more data could be useful. I discuss this in the [recommendations](#Recommendations). 

While I believe the data is good enough, it does have some limitations such as:

* One possible source of bias is that we are using data collected during a pandemic. This probably affected some potential riders. For example, there were no in-site classes for most of this period and also some companies were in home office policy. 

* For anonymization concerns we are not able to identify the riders. It seems necessary to find ways to respect anonymization but get more information about our riders. I will come back to this point in the Recommendation section.

These question must be addressed and we will further discuss them in the recommendation section. Regardless of these points, I do think that the data we have is good enough for creating a useful analysis for the company.


# Processing the data

All our data work, from preparing the data to the visualizations, will be done using R and RStudio. In particular, I will use the tidyverse to wrangle the data, ggplot2 for creating the visualizations, and blogdown for creating the site where this analysis is being shared. R all the way down.


As previously mentioned I used data from 2020 October, to 2021 September. You can access the data [here](https://divvy-tripdata.s3.amazonaws.com/index.html). As it is in .zip format, it's necessary to download it locally, extract and then import to R. These steps I cannot record. All the others steps are documented below.

## Importing and combining the data


We first load the packages necessary for our work. After that, we import the data for the 12 months we are going to analyze:


```r
library(tidyverse)
library(lubridate) # For working with dates
library(scales)
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

In December 2020 the type of the variables *start_station_id* and *end_station_id* changed from numeric to character.[^1]
[^1]:  When we run *read_csv* we get messages that describe the data imported. This is how I discovered this particular problem. I suppressed this message from the final document for aesthetic reasons. 


As we need that every column have the same type, we will coerce the values for October and November to character.



```r
td_2020_10 <- td_2020_10 %>% mutate(start_station_id = as.character(start_station_id), end_station_id = as.character(end_station_id))
td_2020_11 <- td_2020_11 %>% mutate(start_station_id = as.character(start_station_id), end_station_id = as.character(end_station_id))
```

Now, the data is consistent and we can combine them in one data frame.


```r
trip_data <- bind_rows(td_2020_10, td_2020_11, td_2020_12,td_2021_01,td_2021_02, td_2021_03,        #Combining the data
                       td_2021_04, td_2021_05, td_2021_06, td_2021_07, td_2021_08, td_2021_09)

rm(td_2020_10, td_2020_11, td_2020_12,td_2021_01,td_2021_02, td_2021_03,      #Removing the individual data sets from the env.
   td_2021_04, td_2021_05, td_2021_06, td_2021_07, td_2021_08, td_2021_09)
```



Now let's coerce the dataframe to a tibble. Tibbles are data frames but with some adjustments for working better with the tidyverse and also showing better in the screen. You can find more information [here](https://r4ds.had.co.nz/tibbles.html).




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

## Cleaning the data

We still have to work a lot with our data before beginning our analysis. Regardless of that, I want to have an idea about the proportion of members and casual rides. I run the code below to get this information.


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


### Creating new variables and treating the data

An integral part of data preparation is creating new variables by using current data. We can apply transformation and calculations, in a way that create variables there are useful in posterior analysis. 

Along the way, we also have to look for any inconsistent data and then decide what to do about it. Let's go.

#### Creating a variable for the lenght of the ride.

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
Unless somebody was the Flash and could go back in time, this info show to us that there are some errors in the data. As min_ride is negative, it is telling us that there are negative entries. I also don't like the fact that our length ride is in seconds. So, let's first analyze these rides with negative length and after that we will transform it units to minutes.


Let's first check how many observations we have with this error.


```r
sum(trip_data$ride_lenght <=0) #By summing a logical vector you get the number of values that evaluates to FALSE
```

```
## [1] 3762
```
In a data frame with more that 5 mln. observations, it seems ok to me to drop less than 4.000 entries. 
This has potential for creating a problem, if the factor that explain the mistake is one we are interested in. But it does not seems to be the case here, so let's drop these observation.


As a rule, whenever I drop a row I will create a data frame with a new name. 
I do this because if there is any problem, we can easily go back to the previous dataframe.


```r
# Here I use dplyr filter call to drop all the rows with ride_lenght lower than 0
trip_data_2 <- trip_data %>% 
  filter(!ride_lenght <= 0)
```


```r
trip_data_2$ride_lenght = trip_data_2$ride_lenght/60
```

We are done treating the *ride_lenght* variable.

#### Creating date variables: Weekdays, months, and hour.

We can use the *started_at* variable for extracting which day of the week the ride took place. We can use this variable later to check if there any day preference difference between casual and members.


We use the code below to create a variable *day_of_week*.
As my system is in Brazilian Portuguese, I have to set its language to English so I can get the weekdays with the name I expect.


```r
Sys.setlocale("LC_TIME", "English")  # This is necessary so I get dates in English.
```

```
## [1] "English_United States.1252"
```

```r
trip_data_2 <- trip_data_2 %>% 
  mutate(day_of_week = weekdays(started_at)) %>% 
  mutate( day_of_week = factor(day_of_week, levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")))


unique(trip_data_2$day_of_week) 
```

```
## [1] Saturday  Thursday  Wednesday Sunday    Monday    Tuesday   Friday   
## Levels: Monday Tuesday Wednesday Thursday Friday Saturday Sunday
```

I use the code below to add a month variable to our data frame. I will use this later to check any seasonality in the rides.


```r
trip_data_2 <- trip_data_2 %>% 
  mutate(month = format(as.Date(started_at), "%Y-%m"))

unique(trip_data_2$month)
```

```
##  [1] "2020-10" "2020-11" "2020-12" "2021-01" "2021-02" "2021-03" "2021-04"
##  [8] "2021-05" "2021-06" "2021-07" "2021-08" "2021-09"
```

I also want to create a hour variable, so I can see the hour-distribution of the rides


```r
trip_data_2 <- trip_data_2 %>% 
  mutate(start_hour = hms:: hms(second(started_at), minute(started_at), hour(started_at)))

trip_data_2 %>% 
  summarise(min(start_hour), mean(start_hour), max(start_hour))
```

```
## # A tibble: 1 x 3
##   `min(start_hour)` `mean(start_hour)` `max(start_hour)`
##   <drtn>            <drtn>             <drtn>           
## 1 0 secs            53242.62 secs      86399 secs
```

#### Checking for rows that should be dropped.

There are some stations that in reality are maintenance facilities.[^2]  We should drop all the rides that begin or end at them. First, I will check how many rides fall in this category. 

[^2]:I got this information by using the code  *unique(trip_data_2$start_station_name)* and then visually inspecting the output. As the output was too big, I suppressed it from the final document.


```r
rows_to_drop <- c("WATSON TESTING - DIVVY","Base - 2132 W Hubbard Warehouse","DIVVY CASSETTE REPAIR MOBILE STATION","HUBBARD ST BIKE CHECKING (LBS-WH-TEST)")

sum(trip_data_2$end_station_name %in% rows_to_drop)
```

```
## [1] 1283
```

```r
sum(trip_data_2$start_station_name %in% rows_to_drop)
```

```
## [1] 1064
```
I will drop these rows because they are not genuine rides. Besides that, it does not seem reasonable to assume that it will have any impact.



```r
trip_data_3 <- trip_data_2 %>% 
  filter(!(end_station_name %in% rows_to_drop | start_station_name %in% rows_to_drop))
```

This finishes our data processing efforts. 

I imported the relevant data and combined them in a data frame. I also created new variables that we will use to explore the difference between casual riders and members. Finally, I also dropped some rows that had some wrong data stored.


# Analysis

#### Introduction

In this section we are going to analyze the data for insights. Our data is in long format and already [tidy](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html). Most of our analysis will consist of visualizations. I believe that this is probably the best tool for exploratory analysis. A plot can make clear what a table with numbers may hide.


First, I want to see if the fact we dropped some rows changed the proportion between casual riders and annual members in the data.


```r
n = length(trip_data_2$member_casual)  # Gets the total number of observations

trip_data_3 %>% 
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


No relevant change. What makes sense given how irrelevant the data we dropped seemed to be.

#### Is there any difference between the usage of our bikes in weekdays and weekends?

First, let's inspect the number of rides per day.


```r
table(trip_data_3$day_of_week)
```

```
## 
##    Monday   Tuesday Wednesday  Thursday    Friday  Saturday    Sunday 
##    641781    657820    679456    694608    744041    923355    789826
```
As we can see there are more rides in the weekends than in the weekdays. 

Now, let's use a bar plot to visualize the number of daily rides, but divided between our group of interest.


```r
trip_data_3 %>% 
  ggplot(aes(x = day_of_week)) + 
  geom_bar(aes(fill = member_casual), position = "dodge") +
  scale_y_continuous(labels = comma) +
  ylab("Number of rides") +
  xlab("") +
  ggtitle("Casual riders prefer the weekends", subtitle = "Number of rides per day of week")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

It's clear that, throughout the weekdays there are more annual members riding bikes than casual riders.
In the weekends, there is a jump in the number of casual riders and a small decrease in the number of members riders.

As we now that at least 30% of our users use bikes for commute, it's expected that they are annual members. This would explain why we have more rides by members in the weeekdays 


#### How does the ride lenght differs between members and casual riders?



```r
trip_data_3 %>%
  group_by(day_of_week, member_casual) %>% 
  summarise(mean_ride = mean(ride_lenght)) %>% 
  ggplot(aes(x = day_of_week, y = mean_ride)) +
  geom_col(aes(fill = member_casual), position = "dodge")+
  ylab("minutes") + 
  xlab("") +
  theme(legend.title = element_blank()) +
  ggtitle("Members have shorter rides than casual users", subtitle = "Mean ride lenght, by week day")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

It is clear that casual riders have longer rides. But I want to get an idea of the distribution. 
I will use a box plot for this. 


```r
trip_data_3 %>% 
  ggplot(aes(x = day_of_week, y = ride_lenght)) +
  ylim(0,60) +
  geom_boxplot(aes(color = member_casual))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />
As we can see, the typical ride for members are similar in all the days and shorter than 20 minutes. But, there are many outliers. In general, this seems ok. We are interested in the difference between casual riders and members, and this reinforce our view that the typical member ride is shorter. Possibly because many member are not using our bikes for fun activities, but as a mean for going from point A to point B. 

#### Is there any difference between how members and casual riders use our assisted bicycles ?

Cyclistic offers different type of bicycles, with the goal of making bike-share more inclusive to people with disabilities or that can't use a standard two-wheeled bike. We know that about 8% of riders use these assisted options. Let's see if there is any difference between how members and casual riders use them.

First, let's look at how many rides with each available type of bikes we have.

```r
table(trip_data_3$rideable_type)
```

```
## 
##  classic_bike   docked_bike electric_bike 
##       2750382        674753       1705752
```
As always, let's go back to visualizations of the data.

```r
trip_data_3 %>% 
  ggplot(aes(x = rideable_type)) + 
  geom_bar(aes(fill = member_casual), position = "dodge")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" />
While we can see the member use more classic bikes than casual users. I have my doubts about how mu we could generalize from here. 


#### Seasonality

Let's take a look at how the rides are distributed throughout the year


```r
trip_data_3 %>% 
  group_by(month, member_casual) %>% 
  summarise(number_of_trips = n()/1000) %>% 
  ggplot((aes(x = month, y = number_of_trips, color = member_casual))) +
  geom_line(aes(group= member_casual)) +
  geom_point(size = 2) +
  labs(title = "Number of rides per month", subtitle = "Data from 2020-10 to 2021-09",
       y = "Number of trips in thousands", x = "") +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90), legend.title = element_blank())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />

There is a considerable variation in the number of rides each month. Probably due to weather condition. But as the number of member and casual rides are roughly following the same pattern, we will not further investigate this.


#### Is there any difference between the hour-distribution of rides between the two groups?


```r
trip_data_3 %>% 
  group_by(start_hour, member_casual) %>% 
  summarise(number_of_trips = n()) %>% 
  ggplot((aes(x = start_hour, y = number_of_trips, color = member_casual))) +
  geom_point(aes(group= member_casual)) +
  labs(title = "Rides per hour", subtitle = "24-hour range",
       y = "Number of trips in thousands", x = "") +
  scale_x_time(breaks = hms::as.hms(c('00:00:00',"03:00:00","06:00:00","08:00:00",'09:00:00', '12:00:00','15:00:00','18:00:00','21:00:00'))) +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90), legend.title = element_blank()) +
  facet_grid(member_casual~.) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Hourly distribution of rides-1.png" width="672" />

We can see that there is a small surge in the number of rides in the member group around 8:00 a.m.. The same pattern is not present in the casual group. As we know that about 30% of riders use our bike to commute, it seems that most of our commuters are members.

One possibility is that a considerable amount of member use our bicycles for commute. 

We can further check this if we subset the data on weekdays and weekends. If my hypotheses is true we will see a considerable surge around 08:00 on weekdays and see almost no difference on weekends.


First, let's plot the hourly distribution of rides for weekdays.


```r
trip_data_3 %>% 
  filter(!day_of_week %in% c("Saturday", "Sunday")) %>% 
  group_by(start_hour, member_casual) %>% 
  summarise(number_of_trips = n()) %>% 
  ggplot((aes(x = start_hour, y = number_of_trips, color = member_casual))) +
  geom_point(aes(group= member_casual)) +
  labs(title = "Members rides are bi-modal on weekdays.\nThere is a hump around 08:00 a.m.", subtitle = "Distribution of rides throughout the day - Weekdays only",
       y = "Number of trips in thousands", x = "") +
  scale_x_time(breaks = hms::as.hms(c('00:00:00',"03:00:00","06:00:00","08:00:00",'09:00:00', '12:00:00','15:00:00','18:00:00','21:00:00'))) +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90), legend.title = element_blank(), legend.position = "none") +
  facet_grid(member_casual~.) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Weekday rides-1.png" width="672" />

As we can see in the plot above, the raise around 08:00 is more pronounced when we focus on weekdays. This reinforce the idea that many members are using our bikes for commute, while the same does not seem to true for casual riders.

For concluding our analysis about the hourly distribution of rides, let's plot the data only for weekends.



```r
 trip_data_3 %>% 
  filter(day_of_week %in% c("Saturday", "Sunday")) %>% 
  group_by(start_hour, member_casual) %>% 
  summarise(number_of_trips = n()) %>% 
  ggplot((aes(x = start_hour, y = number_of_trips, color = member_casual))) +
  geom_point(aes(group= member_casual)) +
  labs(title = "On weekends the distribution of rides are similar", subtitle = "Distribution of rides throughout the day - Weekend only",
       y = "Number of trips in thousands", x = "") +
  scale_x_time(breaks = hms::as.hms(c('00:00:00',"03:00:00","06:00:00","08:00:00",'09:00:00', '12:00:00','15:00:00','18:00:00','21:00:00'))) +
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90), legend.title = element_blank(), legend.position = "none") +
  facet_grid(member_casual~.) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/Weekend rides-1.png" width="672" />

On weekends, there is no clear difference in the hourly distribution of rides.

## Summing-up the analysis

Our goal was to search for difference between members and casual rides. The most relevant differences we found were the following:

* Members are the main users of our bikes on weekdays. But on weekends, we have more casual users than members riding our bikes.

* The pattern of use during weekdays differs between the two groups. On weekdays, there is a jump in usage around 08:00 a.m.. The same behavior in not observed in casual riders or on weekends.

* Annual members take shorter rides. 

# Recommendations

1. Expand the time-frame of our analysis. While we analyzed a good amount of data, we have the problem of working with data collected during a pandemic. Give these exceptional circumstances it's possible that our analysis have some biases. So, it would be important to run a similar analysis some months from now, with updated data.

2. We need to better understand our client, while respecting anonymization. Assuming that users must use an app when unlocking the bikes, we could run a digital survey. For example, whenever some user unlock a bike we could question if they are going to use it for commute to work or school, for fun, or other category. We should try to know *why* they are using our bikes, so we can offer better services.

3. In term of products, we should analyze the possibility of creating a weekend-only membership. On weekends there is raise in the numbers of casual rides. It's possible that some of these riders could be interest in such membership.


This is it for now.
