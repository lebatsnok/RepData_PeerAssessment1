# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data


```r
X <- read.csv("activity.csv", as.is = TRUE)
# convert date to Date
X$date <- as.Date(X$date)
# calculate start date/time for each interval - just in case we later need
# it
X$start <- with(X, trunc(as.POSIXct(date), "days") + interval * 60)
```


## What is mean total number of steps taken per day?

Ignoring the missing values in the dataset, I first make a histogram of the total number of steps taken each day. I chose to use 20 breaks so that the lowest category (very few or 0 daily steps) can be clearly seen on the histogram.


```r
# ignoring the missing values in the dataset: - make a histogram of the
# total number of steps taken each day
dailysteps <- with(X, tapply(steps, date, sum, na.rm = TRUE))
hist(dailysteps, breaks = 20, xlab = "Number of steps per day", main = "Histogram of daily steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 



```r
# - calculate the mean and median total number of steps taken per day
mean.steps <- format(mean(dailysteps), digits = 1)
median.steps <- format(median(dailysteps), digits = 1)
```


Ok now the mean of total number of steps taken per day was 9354 and the median was 10395. 

## What is the average daily activity pattern?

Here is the plot for average daily activity pattern - that is, for each 5-minute interval, the mean number of steps (across all days in the data set) at this interval is shown.


```r
intervalmeans <- with(X, tapply(steps, interval, mean, na.rm = TRUE))
labs <- format(X$start, "%H")[1:(1440/5)]
plot(intervalmeans, axes = FALSE, type = "l", xlab = "Time of day", ylab = "Mean steps", 
    main = "Average daily activity pattern")
axis(1, at = seq(1, 289, 36), labels = seq(0, 24, 3))
axis(2)
box()
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


## Imputing missing values


```r
number.of.missing.values <- sum(is.na(X$steps))
```


There are 2304 missing values in the data set. This is way too much! Something must be done about it! Here we have devised a simple strategy: in each 5-minute period where the number of steps is missing, we will use the average value from other days. The steps data with imputed missing values will be stored in the data frame called X2.



```r
folded <- matrix(X$steps, nrow = 288)
intervalmeans <- apply(folded, 1, mean, na.rm = TRUE)
steps2 <- c(apply(folded, 2, function(x) ifelse(is.na(x), intervalmeans, x)))
X2 <- X
X2$steps <- steps2
```


Nice. Now let's make the histogram.

```r
dailysteps2 <- with(X2, tapply(steps, date, sum, na.rm = TRUE))
hist(dailysteps2, breaks = 20, xlab = "Number of steps per day", main = "Histogram of daily steps (missing values imputed)")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 



```r
# - calculate the mean and median total number of steps taken per day
mean.steps2 <- format(mean(dailysteps2), digits = 1)
median.steps2 <- format(median(dailysteps2), digits = 1)
diff.mean <- format(mean(dailysteps2) - mean(dailysteps), digits = 1)
diff.median <- format(median(dailysteps2) - median(dailysteps), digits = 1)
```


After imputing the missing data, the mean of total number of steps taken per day was 10766 and the median was 10766. So imputing the missing values makes a difference of 1412 steps for the mean, and 371 steps for the median.

## Are there differences in activity patterns between weekdays and weekends?

Let's finally see if the activity patterns are different for weekdays and weekends.


```r
X2$wkdy <- factor(ifelse(format(X2$date, "%u") %in% 1:5, "weekday", "weekend"))
imWE <- with(subset(X2, wkdy %in% "weekend"), tapply(steps, interval, mean, 
    na.rm = TRUE))
imWD <- with(subset(X2, wkdy %in% "weekday"), tapply(steps, interval, mean, 
    na.rm = TRUE))
```



```r
par(mfrow = 2:1, mar = c(0.125, 4, 4, 0.5))
plot(imWD, axes = FALSE, type = "l", xlab = "Time of day", ylab = "Mean steps", 
    main = "Average daily activity pattern", ylim = c(0, 250))
axis(2)
text(15 * 12, 240, "Weekdays")
box()

par(mar = c(4, 4, 0.125, 0.5))
plot(imWE, axes = FALSE, type = "l", xlab = "Time of day", ylab = "Mean steps", 
    ylim = c(0, 250))

axis(1, at = seq(1, 289, 36), labels = seq(0, 24, 3))
axis(2)
box()
axis(1, at = seq(1, 289, 36), labels = seq(0, 24, 3))
axis(2)
text(15 * 12, 240, "Weekends")
box()
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 

```
