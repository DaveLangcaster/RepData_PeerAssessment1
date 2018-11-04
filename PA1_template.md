---
title: "Reproducible Research: Peer Assessment 1"
author: Dave Langcaster
date: "4th November, 2018"
output: 
  html_document:
    keep_md: true
---
## Initial housekeeping tasks

Turning warnings off globally


```r
knitr::opts_chunk$set(warning=FALSE)
```

Displaying all code


```r
echo = TRUE #Display all code
```

## Synopsis

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The purpose of the assessment was to display skills in:

* loading and preprocessing data
* imputing missing values
* interpreting data to answer research questions

## Data

The data for this assignment was downloaded from the course web
site at [this url](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) as a zip file, and then extracted into the working directory.

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


## 1. Loading and preprocessing the data

First, load the data

```r
colclass = c("integer", "character", "integer")
df <- read.csv("activity.csv", head=TRUE, colClasses=colclass, na.strings="NA")
head(df)
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
Next, process the data for analysis. Date columns need to be set to the correct format, and missing values need to be removed. The cleaned data is then saved to a new data frame, retaining the original dataset intact.


```r
df$date <- as.Date(df$date)
df_sub <- subset(df, !is.na(df$steps))
```

## 2. What is mean total number of steps taken per day?

To answer this question, we plot a histogram showing the daily total number of steps.


```r
dailysteps <- tapply(df_sub$steps, df_sub$date, sum, na.rm=TRUE, simplify=T)
dailysteps <- dailysteps[!is.na(dailysteps)]

hist(x=dailysteps,
     col="blue",
     breaks=20,
     xlab="Total steps per day",
     ylab="Frequency",
     main="Distribution of total daily steps (missing data values excluded)")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Finally, we need to calculate the mean total number of steps per day.


```r
mean(dailysteps)
```

```
## [1] 10766.19
```
We should also calculate the median to account for any skewness of the data.


```r
median(dailysteps)
```

```
## [1] 10765
```

The mean is therefore 10766 steps, and the median is 10765 steps. The data is therefore close to a normal distribution.

## 3. What is the average daily activity pattern?

Average daily activity pattern is best visualised using a time series plot. Activity is measured in 5-minute intervals (x-axis) and the plot shows the average across all days of the number of steps taken each day (y-axis).


```r
steps_avg <- tapply(df_sub$steps, df_sub$interval, mean, na.rm=TRUE, simplify=T)
df_int_avg <- data.frame(interval=as.integer(names(steps_avg)), avg=steps_avg)

with(df_int_avg,
     plot(interval,
          avg,
          type="l",
          xlab="Daily 5-minute intervals",
          ylab="Average steps per interval (all days)"))
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

The next step is to find which 5-minute interval across all the days had the maximum number of steps.


```r
max_steps <- max(df_int_avg$avg)
df_int_avg[df_int_avg$avg == max_steps, ]
```

```
##     interval      avg
## 835      835 206.1698
```

This turns out to be interval 835, with 206 steps.

## 4. Imputing missing values

There are a number of days/intervals where there are missing values (coded as \color{red}{\verb|NA|}NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

We therefore have to calculate the number of missing values in the dataset.


```r
sum(is.na(df$steps))
```

```
## [1] 2304
```

This shows that there are 2304 rows with missing data. A simple strategy to fill in the mssing values would be to use the mean for any 5-minute interval that has missing data.

We need to create a new data frame that is completed using this strategy.


```r
df_impute <- df
ndx <- is.na(df_impute$steps)
int_avg <- tapply(df_sub$steps, df_sub$interval, mean, na.rm=TRUE, simplify=T)
df_impute$steps[ndx] <- int_avg[as.character(df_impute$interval[ndx])]
```

Next, we need to create a new histogram, similar to that in 2. above, showing the total number of steps per day, that includes the imputed data.


```r
new_dailysteps <- tapply(df_impute$steps, df_impute$date, sum, na.rm=TRUE, simplify=T)

hist(x=new_dailysteps,
     col="blue",
     breaks=20,
     xlab="Total steps per day",
     ylab="Frequency",
     main="Distribution of total daily steps (missing data values imputed)")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->
Finally, we calculated the mean and median of the imputed data.


```r
mean(new_dailysteps)
```

```
## [1] 10766.19
```

```r
median(new_dailysteps)
```

```
## [1] 10766.19
```

This shows a slight increase in the median, but the mean remains exactly the same. The missing values have little to no effect on the data.

## 5. Are there differences in activity patterns between weekdays and weekends?

To answer this question, we need to create a new factor variable 'wkday' in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

We first create a helper function 'is_wkday()' to decide if a day is a weekday or not.

```r
is_wkday <- function(d) {
    wd <- weekdays(d)
    ifelse (wd == "Saturday" | wd == "Sunday", "weekend", "weekday")
}
```
Then we use this to iterate between all the columns of the dataset to sort the weekdays from weekends.


```r
wd2 <- sapply(df_impute$date, is_wkday)
df_impute$wkday <- as.factor(wd2)
head(df_impute)
```

```
##       steps       date interval   wkday
## 1 1.7169811 2012-10-01        0 weekday
## 2 0.3396226 2012-10-01        5 weekday
## 3 0.1320755 2012-10-01       10 weekday
## 4 0.1509434 2012-10-01       15 weekday
## 5 0.0754717 2012-10-01       20 weekday
## 6 2.0943396 2012-10-01       25 weekday
```
Finally, we create a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
wd_df <- aggregate(steps ~ wkday+interval, data=df_impute, FUN=mean)

library(lattice)
xyplot(steps ~ interval | factor(wkday),
       layout = c(1, 2),
       xlab="Interval",
       ylab="Number of steps",
       type="l",
       lty=1,
       data=wd_df)
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

We can see that weekday activity seems to start earlier than weekend activity, and there is a peak of activity in the early morning on weekdays. Following this, the activity levels remain low throughout the day compared with weekends. On weekends activity starts later, but remains relatively high throughout the day.
