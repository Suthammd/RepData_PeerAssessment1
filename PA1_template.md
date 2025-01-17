---
title: "Reproducible Research: Peer Assessment 1"
author: "By Jirapanakorn Sutham"
output: 
  html_document:
    keep_md: true
---
1. Reading the dataset downlod from "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"

```r
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", 
              "activity.zip")
unzip("activity.zip")
Activity_mornitoring_data <- read.csv("activity.csv", header = TRUE , na.strings = "NA")
```
Change date column class from Character to Date

```r
Activity_mornitoring_data$date <- as.Date(as.character(Activity_mornitoring_data$date), 
                                          "%Y-%m-%d")
```
Rearrange dataset column in date,interval,steps order.

```r
library(dplyr)
Activity_mornitoring_data<-relocate(Activity_mornitoring_data, steps, .after = interval)
head(Activity_mornitoring_data)
```

```
##         date interval steps
## 1 2012-10-01        0    NA
## 2 2012-10-01        5    NA
## 3 2012-10-01       10    NA
## 4 2012-10-01       15    NA
## 5 2012-10-01       20    NA
## 6 2012-10-01       25    NA
```
2. Plot histogram of the total number of steps taken each day

```r
step_by_date<-Activity_mornitoring_data %>%
    group_by(date)%>%
       summarise(total_steps = sum(steps))

par(mar = c(4,4,1,1))
barplot(total_steps~date, data = step_by_date , col =rgb(1,.1,.5, alpha = 0.3),
           xlab = "Date", ylab="steps", main = "The total steps taken each day", 
              space = 0)
```

![](PA1_template_files/figure-html/plot histogram-1.png)<!-- -->
  
3.Mean and median number of steps taken each day

```r
x<- Activity_mornitoring_data %>%
    group_by(date) %>%
      select(date,steps) %>%
        summarise(mean_steps = mean(steps, na.rm = TRUE))
y <-  Activity_mornitoring_data %>%
      group_by(date) %>%
        select(date,steps) %>%
           summarise(median_steps = median(steps, na.rm = TRUE))

result<-data.frame(mean = x, median =y)
result <- result[,-3]
head(result)
```

```
##    mean.date mean.mean_steps median.median_steps
## 1 2012-10-01             NaN                  NA
## 2 2012-10-02         0.43750                   0
## 3 2012-10-03        39.41667                   0
## 4 2012-10-04        42.06944                   0
## 5 2012-10-05        46.15972                   0
## 6 2012-10-06        53.54167                   0
```
4.Time series plot of the average number of steps taken.

```r
Avgstep_by_date<-Activity_mornitoring_data %>%
    group_by(date)%>%
     mutate(Avg_steps = mean(steps, na.rm = TRUE))%>%
      ungroup()%>%
          mutate(interval_index = 1:nrow(Activity_mornitoring_data))
    

Avgstep_by_date$dateinterval <-paste(Avgstep_by_date$date,Avgstep_by_date$interval)

par(mar=c(4,4,1,1))
plot(Avg_steps ~ interval_index , data = Avgstep_by_date, type ="l", 
     col =rgb(1,0,0, alpha = 0.7), xaxt = "n",xlab = "5 minutes interval",
        ylab="Average steps in a day", 
            main = "The average steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

5.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_steps <- max(Avgstep_by_date$steps, na.rm = TRUE)
interest_interval <- Avgstep_by_date%>%
                        select(date,interval,steps)%>%
                            filter(steps == max_steps)
which_interval<-paste("date",interest_interval$date,
                      "interval",interest_interval$interval)
```
The answer is date 2012-11-27 interval 615  

6.Code to describe and show a strategy for imputing missing data
Calculate and report the total number of missing values in the dataset

```r
rows<-sum(is.na(Activity_mornitoring_data$steps))
percent<-mean(is.na(Activity_mornitoring_data$steps))*100
```
There are 2304 rows with NAs, which around 13.1147541 %  

Imputing missing values by all steps mean  

```r
m<- mean(Activity_mornitoring_data$steps, na.rm = TRUE)
Activity_mornitoring_data[is.na(Activity_mornitoring_data)] = m
```
7.Histogram of the total number of steps taken each day after missing values are imputed

```r
step_by_date<-Activity_mornitoring_data%>%
    group_by(date)%>%
       summarise(total_steps = sum(steps))

par(mar = c(4,4,1,1))
barplot(total_steps~date, data = step_by_date , col =rgb(1,.5,.5, alpha = 0.3),
           xlab = "Date", ylab="steps", 
             main = "The total steps taken each day, missing value are imputed"
                , space = 0)
```

![](PA1_template_files/figure-html/use imputed_data now-1.png)<!-- -->

8.Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends.

```r
Activity_mornitoring_data <-  Activity_mornitoring_data%>%
                    mutate(day_abb = weekdays(date, abbreviate = TRUE))

weekday_data <- Activity_mornitoring_data %>%
                    filter(day_abb == c("Mon","Tue","Wed","Thu","Fri"))%>%
                        group_by(date)%>%
                            mutate(avg_steps = mean(steps))%>%
                                ungroup()
weekday_data<- weekday_data%>%
                    mutate(interval_index = 1:nrow(weekday_data))

weekend_data <- Activity_mornitoring_data %>%
                    filter(day_abb == c("Sat","Sun"))%>%
                        group_by(date)%>%
                            mutate(avg_steps = mean(steps))%>%
                                ungroup()
weekend_data <- weekend_data%>%
                    mutate(interval_index = 1:nrow(weekend_data))
par(mfrow =c(1,2))
plot(weekday_data$interval_index, weekday_data$avg_steps, type = "l", col="red",
        main ="Weekday average steps taken",ylab ="steps", xlab="5 minutes interval",
           xaxt ="n", ylim = c(0,80))
plot(weekend_data$interval_index, weekend_data$avg_steps, type = "l", col="blue",
        main ="Weekend average steps taken",ylab ="steps", xlab="5 minutes interval",
           xaxt ="n",ylim = c(0,80))
```

![](PA1_template_files/figure-html/identify weekdays and weekends first-1.png)<!-- -->



