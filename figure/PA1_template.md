---
title: "Activity monitoring data analysis (PA1)"
author: "Chengran"
date: "May 17, 2015"
output: html_document
---
# Introduction
This assignment will be described in multiple parts. 
 You will need to write a report that answers the questions detailed below. 
 Ultimately, you will need to complete the entire assignment in a single R markdown document 
 that can be processed by knitr and be transformed into an HTML file.

The variables included in this dataset are:
    steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
    date: The date on which the measurement was taken in YYYY-MM-DD format
    interval: Identifier for the 5-minute interval in which measurement was taken
    The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

# Loading and prepocessing data


```r
setwd("~/USA/JD_Lab/projects/data_science_jhu")
library(dplyr)
library(lattice)
library(knitr)
activityMonitor <- read.csv("data/reproduce_data/activity.csv")
```

# What is mean total number of steps taken per day?
*For this part of the assignment, you can ignore the missing values in the dataset.
1. Calculate the total number of steps taken per day


```r
totalStep_perDay <- activityMonitor %>%
    group_by(date) %>%
    summarise(total = sum(steps))
```

2. If you do not understand the difference between a histogram and a barplot, 
research the difference between them. Make a histogram of the total number of steps taken each day


```r
hist(totalStep_perDay$total)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day


```r
mean(totalStep_perDay$total, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(totalStep_perDay$total, na.rm = TRUE)
```

```
## [1] 10765
```

# What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
meanStep_interval <- activityMonitor %>%
    group_by(interval) %>%
    summarise(averageStep = mean(steps, na.rm = TRUE))
plot(averageStep ~ interval, data = meanStep_interval, type = "l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

```r
# plot(meanStep_interval$interval, meanStep_interval$averageStep, type = "l")
```

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
meanStep_interval[which.max(meanStep_interval$averageStep), ]$interval
```

```
## [1] 835
```

# Imputing missing values
* Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activityMonitor))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
## I used the mean for that 5-minute interval derived from previous question
fillNAstep_mean5minInterval <- function(interval) {
    meanStep_interval[meanStep_interval$interval == interval, ]$averageStep
}
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
new.activityMonitor <- activityMonitor

for(i in 1:nrow(new.activityMonitor)) {
    if(is.na(new.activityMonitor[i, ]$steps) == TRUE) {
        new.activityMonitor[i, ]$steps <- fillNAstep_mean5minInterval(new.activityMonitor[i, ]$interval)
    }
}
## check NAs after filling in missing values
sum(is.na(new.activityMonitor))
```

```
## [1] 0
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 

```r
totalStep_perDay.afterNAfill <- new.activityMonitor %>%
    group_by(date) %>%
    summarise(total = sum(steps))

hist(totalStep_perDay.afterNAfill$total)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

```r
mean(totalStep_perDay.afterNAfill$total, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(totalStep_perDay.afterNAfill$total, na.rm = TRUE)
```

```
## [1] 10766.19
```
+ Do these values differ from the estimates from the first part of the assignment? 
+ What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
    # The mean are both 10766.19 before imputing missing data, 
    # but the median shows a little increase from 10765 to 10766.19.
```

# Are there differences in activity patterns between weekdays and weekends?
* For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
## I decided to use isWeekday function from timeDate package.
library(timeDate)
new.activityMonitor$dayType <- ifelse(isWeekday(new.activityMonitor$date, wday=1:5), 
                                      "weekday", "weekend")

new.activityMonitor$dayType <- factor(new.activityMonitor$dayType, levels = c("weekday", "weekend"))
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis)  and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 
See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
meanStep_perDay.afterNAfill.day <- new.activityMonitor %>%
    group_by(interval, dayType) %>%
    summarise(averageStep = mean(steps))

xyplot(averageStep ~ interval | factor(dayType), data = meanStep_perDay.afterNAfill.day, 
       aspect = 1/2, type = "l", xlab = "Interval", ylab = "Number of steps")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 






