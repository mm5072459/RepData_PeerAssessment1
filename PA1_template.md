---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

```r
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
library(ggplot2)
```


## Loading and preprocessing the data

1.Load the data (i.e. read.csv())

Here i load the data using the read.csv() command.

```r
activity <- read.csv("activity.csv")
```
Below, i use the str(), summary() and head() commands to see how the data looks.

```r
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```



```r
summary(activity)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0  
##  NA's   :2304
```



```r
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

2.Process/transform the data (if necessary) into a format suitable for your analysis

i removed the missing values from the data here.

```r
act.complete <- na.omit(activity)
```


## What is mean total number of steps taken per day?

1.Calculate the total number of steps taken per day

Here i use dplyr to separate the data into days so i can calculate the total number of steps.

```r
library(dplyr)
act.day <- group_by(act.complete, date)
act.day <- summarize(act.day, steps=sum(steps))
```



```r
summary(act.day)
```

```
##      date               steps      
##  Length:53          Min.   :   41  
##  Class :character   1st Qu.: 8841  
##  Mode  :character   Median :10765  
##                     Mean   :10766  
##                     3rd Qu.:13294  
##                     Max.   :21194
```

2.If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
library(ggplot2)
qplot(steps, data=act.day)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

3.Calculate and report the mean and median of the total number of steps taken per day

Here i use the mean() and median() functions.

```r
mean(act.day$steps)
```

```
## [1] 10766.19
```



```r
median(act.day$steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

1.Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

Here i make a data frame where i aggregate the steps into averages in 5-minute intervals:

```r
act.int <- group_by(act.complete, interval)
act.int <- summarize(act.int, steps=mean(steps))
```

Here is the plot of average steps daily in comparison to the intervals:

```r
ggplot(act.int, aes(interval, steps)) + geom_line()
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

We find the row in the interval data frame for which steps is equal to the maximum number of steps, then we look at the interval of that row:

```r
act.int[act.int$steps==max(act.int$steps),]
```

```
## # A tibble: 1 x 2
##   interval steps
##      <int> <dbl>
## 1      835  206.
```



## Imputing missing values

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

The total number of rows with NAs is equal to the difference between the number of rows in the raw data and the number of rows in the data with only complete cases:

```r
nrow(activity)-nrow(act.complete)
```

```
## [1] 2304
```

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

I replace missing values with the mean number of steps for each interval across all of the days. The act.int data frame contains these means. I start by merging the act.int data with the raw data:


```r
names(act.int)[2] <- "mean.steps"
act.impute <- merge(activity, act.int)
```

3.Create a new dataset that is equal to the original dataset but with the missing data filled in

If steps is NA, I replace the value with the mean number of steps for the interval:

```r
act.impute$steps[is.na(act.impute$steps)] <- act.impute$mean.steps[is.na(act.impute$steps)]
```

4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Heres a dataset with the total number of steps per day using the imputed data:

```r
act.day.imp <- group_by(act.impute, date)
act.day.imp <- summarize(act.day.imp, steps=sum(steps))
```

The histogram and summary statistics are below:

```r
qplot(steps, data=act.day.imp)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-20-1.png)<!-- -->



```r
mean(act.day.imp$steps)
```

```
## [1] 10766.19
```



```r
median(act.day.imp$steps)
```

```
## [1] 10766.19
```
The mean appears to be unaffected by this simple data imputation. The median is smaller.


## Are there differences in activity patterns between weekdays and weekends?

1.Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

I convert the date variable to the date class, then use the weekdays() function to generate the day of the week of each date. I create a binary factor to indicate the two weekend days:

```r
act.impute$dayofweek <- weekdays(as.Date(act.impute$date))
act.impute$weekend <-as.factor(act.impute$dayofweek=="Saturday"|act.impute$dayofweek=="Sunday")
levels(act.impute$weekend) <- c("Weekday", "Weekend")
```

2.Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

Here I create separate data frames for weekends and weekdays:

```r
act.weekday <- act.impute[act.impute$weekend=="Weekday",]
act.weekend <- act.impute[act.impute$weekend=="Weekend",]
```

Here I find the mean number of steps across days for each 5 minute interval:

```r
act.int.weekday <- group_by(act.weekday, interval)
act.int.weekday <- summarize(act.int.weekday, steps=mean(steps))
act.int.weekday$weekend <- "Weekday"
act.int.weekend <- group_by(act.weekend, interval)
act.int.weekend <- summarize(act.int.weekend, steps=mean(steps))
act.int.weekend$weekend <- "Weekend"
```

I append the two data frames together and make 2 time series plots:

```r
act.int <- rbind(act.int.weekday, act.int.weekend)
act.int$weekend <- as.factor(act.int$weekend)
ggplot(act.int, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-26-1.png)<!-- -->

