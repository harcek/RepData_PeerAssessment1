#### reproducible research PA

---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data
####1. Load the data (i.e. read.csv())

```r
measured_activity <- read.csv("activity.csv",header=T, na.string="NA")
```

####2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
total_steps_per_day <- aggregate(measured_activity$steps, by=list(measured_activity$date), FUN=sum, na.rm=TRUE)
colnames(total_steps_per_day) <- c("Day", "Total_Steps")
```

## What is mean total number of steps taken per day?
####1. Make a histogram of the total number of steps taken each day

```r
hist(total_steps_per_day$Total_Steps, main="Number of steps per day", xlab="Number of steps", ylab="Frequency", breaks = 10)
```

![plot of chunk hist_total_steps_per_day](figure/hist_total_steps_per_day.png) 

####2. Calculate and report the mean and median total number of steps taken per day

```r
mean(total_steps_per_day$Total_Steps, na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
median(total_steps_per_day$Total_Steps, na.rm=TRUE)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
####1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
avg_steps_per_interval <- aggregate(measured_activity$steps, by=list(measured_activity$interval), FUN=mean, na.rm=TRUE)
colnames(avg_steps_per_interval) <- c("Interval", "Average_Steps")
with(avg_steps_per_interval, plot(Interval, Average_Steps, type="l", main = "Average number of steps taken (across all days)", xlab = "5-minute interval", ylab="Average number of steps"))
```

![plot of chunk plot_average_activity_pattern](figure/plot_average_activity_pattern.png) 

####2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avg_steps_per_interval[which.max(avg_steps_per_interval$Average_Steps),1]
```

```
## [1] 835
```

## Imputing missing values
####1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
nrow(measured_activity) - sum(complete.cases(measured_activity))
```

```
## [1] 2304
```

####2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
Since the only NA's are in number of steps, it is most logical to me to use mean of that 5-minute interval.

####3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
library(plyr)
measured_activity_imputed <- ddply(measured_activity, 
      .(interval), 
      transform, 
      steps=ifelse(is.na(steps), as.integer(mean(steps, na.rm=TRUE)), steps))
nrow(measured_activity) - sum(complete.cases(measured_activity))
```

```
## [1] 2304
```

####4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
total_steps_per_day_imputed <- aggregate(measured_activity_imputed$steps, by=list(measured_activity_imputed$date), FUN=sum, na.rm=TRUE)
colnames(total_steps_per_day_imputed) <- c("Day", "Total_Steps")
total_steps_per_day_imputed$Total_Steps <- as.numeric(as.character(total_steps_per_day_imputed$Total_Steps))
hist(total_steps_per_day_imputed$Total_Steps, main="Number of steps per day", xlab="Number of steps", ylab="Frequency", breaks = 10)
```

![plot of chunk hist_total_steps_per_day_imputed](figure/hist_total_steps_per_day_imputed.png) 


```r
mean(total_steps_per_day_imputed$Total_Steps, na.rm=TRUE)
```

```
## [1] 10749.77
```


```r
median(total_steps_per_day_imputed$Total_Steps, na.rm=TRUE)
```

```
## [1] 10641
```

## Are there differences in activity patterns between weekdays and weekends?
####1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
measured_activity_imputed$weekend <- factor(ifelse(weekdays(as.Date(measured_activity_imputed$date)) 
                                                   %in% c('Saturday','Sunday'), 'Weekend', 'Weekday'))
avg_steps_per_interval_imputed <- aggregate(measured_activity_imputed$steps, by=list(measured_activity_imputed$interval, measured_activity_imputed$weekend), FUN=mean, na.rm=TRUE)
colnames(avg_steps_per_interval_imputed) <- c("Interval", "Weekend", "Average_Steps")
```

####2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

```r
library(ggplot2)
qplot(data = avg_steps_per_interval_imputed, x = Interval, y = Average_Steps, colour = Weekend, geom = "line", main="Activity patterns weekday vs. weekend", ylab = "Total Steps")
```

![plot of chunk plot_average_daily_activity_pattern_weekday_vs_weekend](figure/plot_average_daily_activity_pattern_weekday_vs_weekend.png) 