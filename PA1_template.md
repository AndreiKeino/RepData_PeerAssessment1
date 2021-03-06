---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
setwd("D:\\Coursera\\Data_Science_Specialization\\05 - Reproducible Research\\Course Project 1")

# load the data
df = read.csv("activity.csv", stringsAsFactors = F)
```

## What is mean total number of steps taken per day?

```r
# What is mean total number of steps taken per day?
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
# calculate the step sum by day
steps <- df %>%
    group_by(date) %>%
    summarise_at(vars(steps), funs(sum(., na.rm=TRUE)))

# plot the histogram

hist(steps$steps, main="Histogram of the total number of steps taken each day.", xlab='Steps')
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
# calculate the mean steps
meanSteps <- mean(steps$steps)
meanSteps
```

```
## [1] 9354.23
```

```r
# and the median steps
medSteps <- median(steps$steps, na.rm = T)
medSteps
```

```
## [1] 10395
```


## What is the average daily activity pattern?

```r
#What is the average daily activity pattern?

#calculate the average number of steps taken, averaged across all days (y-axis)

avrgSteps <- df %>%
    group_by(interval) %>%
    summarise_at(vars(steps), funs(mean(., na.rm=TRUE)))

# Make a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number 
# of steps taken, averaged across all days (y-axis)

plot(avrgSteps$interval, avrgSteps$steps, type='l', main='Average number of steps taken in interval', 
     ylab='Average steps', xlab='Interval')
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
#Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

# index of interval which contains the maximum number of steps
maxIdx = which.max(avrgSteps$steps)

# value of interval  which contains the maximum number of steps
maxInterval <- avrgSteps$interval[[maxIdx]]

# Calculate and report the total number of missing values in the 
# dataset (i.e. the total number of rows with NAs)

naCnt <- sum(complete.cases(df))
naCnt
```

```
## [1] 15264
```


## Imputing missing values

```r
# the strategy to fill missing NA's is the mean for that 5-minute interval
# Create a new dataset that is equal to the original dataset but with the missing data filled in.

ds <- df
# filling the missing values

for(i in 1:length(ds$steps))
{
    if(is.na(ds$steps[[i]]))
    {
        interval <- ds$interval[[i]]
        idx <- match(interval, avrgSteps$interval)
        ds$steps[[i]] <- avrgSteps$steps[[idx]]
    }
}

# Make a histogram of the total number of steps taken each day and Calculate 
# and report the mean and median total number of steps taken per day. 
# Do these values differ from the estimates from the first part of the assignment? 
# What is the impact of imputing missing data on the estimates of the total daily number of steps?


totStepsByDay <- ds %>%
    group_by(date) %>%
    summarise_at(vars(steps), funs(sum(., na.rm=TRUE)))

#a histogram of the total number of steps taken each day and Calculate 

hist(totStepsByDay$steps, main="Histogram of the total number of steps taken each day after replacing NAs", xlab='Steps')
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
# calculate the mean steps
meanSteps1 <- mean(totStepsByDay$steps,  na.rm = T)

meanSteps1
```

```
## [1] 10766.19
```

```r
# and the median steps
medSteps1 <- median(totStepsByDay$steps, na.rm = T)

medSteps1
```

```
## [1] 10766.19
```
## Are there differences in activity patterns between weekdays and weekends?

```r
# Create a new factor variable in the dataset with two levels – “weekday” and “weekend” 
# indicating whether a given date is a weekday or weekend day.
library(chron)

ds$weekend = factor(as.integer(is.weekend(ds$date)), labels=c("weekday", "weekend"))


# Make a panel plot containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis) 
# and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 
# See the README file in the GitHub repository to see an example of what this plot should look like 
# using simulated data.

# create the weekday and weekend dataframes
dsWeekday <- ds[ds$weekend == "weekday", ]
dsWeekend <- ds[ds$weekend == "weekend", ]

avrgStepsWd <- dsWeekday %>%
    group_by(interval) %>%
    summarise_at(vars(steps), funs(mean(., na.rm=TRUE)))

avrgStepsWe <- dsWeekend %>%
    group_by(interval) %>%
    summarise_at(vars(steps), funs(mean(., na.rm=TRUE)))

par(mfrow=c(2, 1))

plot(avrgStepsWd$interval, avrgStepsWd$steps, type='l', main='Average number of steps taken in interval on weekday', 
     ylab='Average steps', xlab='Interval')

plot(avrgStepsWe$interval, avrgStepsWe$steps, type='l', main='Average number of steps taken in interval on weekend', 
     ylab='Average steps', xlab='Interval')
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
