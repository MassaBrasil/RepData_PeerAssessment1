# Reproducible Research: Peer Assessment 1

## Prepare environment for R markdown (US English)

For non-US English environments (e.g. Portuguese), for better readability (i.e. for weekdays), it is better set it to US English.
As further measures, 'knitr' package was loaded and chunk default options were set as below.



```r
Sys.setlocale("LC_TIME","C")
```

```
## [1] "C"
```

```r
library(knitr)
```

```
## Warning: package 'knitr' was built under R version 3.1.2
```

```r
opts_chunk$set(echo = TRUE, results = 'hold')
```


## Loading and preprocessing the data

Show any code that is needed to:

- Load the data (i.e. read.csv()) ,

```r
rawActivity <- read.csv("C://coursera//activity.csv", stringsAsFactors=FALSE)
```

- Process/transform the data (if necessary) into a format suitable for your analysis.

```r
rawActivity$date <- as.POSIXct(rawActivity$date, format="%Y-%m-%d")

rawActivity <- data.frame(date=rawActivity$date, 
                           weekday=tolower(weekdays(rawActivity$date)), 
                           steps=rawActivity$steps, 
                           interval=rawActivity$interval)

rawActivity <- cbind(rawActivity, 
                      daytype=ifelse(rawActivity$weekday == "saturday" | rawActivity$weekday == "sunday", "weekend", "weekday")
                        )

activity <- data.frame(date=rawActivity$date, 
                       weekday=rawActivity$weekday, 
                       daytype=rawActivity$daytype, 
                       interval=rawActivity$interval,
                       steps=rawActivity$steps)
```

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.


```r
sum_data <- aggregate(
                      activity$steps, 
                      by=list(activity$date), 
                      FUN=sum, 
                      na.rm=TRUE
                      )
```

Make a histogram of the total number of steps taken each day.


```r
names(sum_data) <- c("date", "total")

hist(sum_data$total, 
     breaks=seq(from=0, to=25000, by=2500),
     col="blue", 
     xlab="Tot.num. steps", 
     ylim=c(0, 20), 
     main="Histogram - Tot. num. steps: each day \n (w/o NA's)")
```

![](./PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

Calculate and report the mean and median total number of steps taken per day.


```r
mean(sum_data$total)

median(sum_data$total)

rm(sum_data)
```

```
## [1] 9363.117
## [1] 10417
```

## What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
mean_data <- aggregate(activity$steps, 
                       by=list(activity$interval), 
                       FUN=mean, 
                       na.rm=TRUE)

names(mean_data) <- c("interval", "mean")

plot(mean_data$interval, 
     mean_data$mean, 
     type="l", 
     col="blue", 
     lwd=2, 
     xlab="Interval (min)", 
     ylab="Avg. num. steps", 
     main="Temporal-series: avg. num. steps by interval\n(w/o NA)")
```

![](./PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
max_pos <- which(mean_data$mean == max(mean_data$mean))
max_interval <- mean_data[max_pos, 1]
max_interval

rm(max_pos, max_interval )
```

```
## [1] 835
```

## Inputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.



Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
NA_count <- sum(is.na(activity$steps))
NA_count

rm(NA_count)

na_pos <- which(is.na(activity$steps))
mean_vec <- rep(mean(activity$steps, na.rm=TRUE), times=length(na_pos))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activity[na_pos, "steps"] <- mean_vec
rm(mean_vec, na_pos)


sum_data <- aggregate(activity$steps, by=list(activity$date), FUN=sum)
names(sum_data) <- c("date", "total")
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 


```r
hist(sum_data$total, 
     breaks=seq(from=0, to=25000, by=2500),
     col="blue", 
     xlab="Tot. num. steps", 
     ylim=c(0, 30), 
     main="Histogram: tot. num. steps for each day\n(NA replaced by mean of day)")
```

![](./PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

Do these values differ from the estimates from the first part of the assignment? What is the impact of inputing missing data on the estimates of the total daily number of steps?


```r
mean(sum_data$total)

median(sum_data$total)

rm(sum_data)
```

```
## [1] 10798.61
## [1] 10766.19
```

Values present significative differences from the estimates from the first part of the assignment. The impact from input of missing values (NA) is increase of data, hence it leads to obtain a higher values in mean and median.

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.



```r
library(lattice)

mean_data <- aggregate(activity$steps, 
                       by=list(activity$daytype, 
                               activity$weekday, activity$interval), mean)

names(mean_data) <- c("daytype", "weekday", "interval", "mean")
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using simulated data:


```r
xyplot(mean ~ interval | daytype, mean_data, 
       type="l", 
       lwd=1, 
       xlab="Interval", 
       ylab="Num. steps", 
       layout=c(1,2))
```

![](./PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

The plot shows that activity patterns between weekdays and weekends are slightly different.

Your plot will look different from the one above because you will be using the activity monitor data. Note that the above plot was made using the lattice system but you can make the same version of the plot using any plotting system you choose.




