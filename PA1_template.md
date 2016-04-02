

Download the packages
library(ggplot2)
library(plyr)
Set the directory
activity <- read.csv("Users/Nawjif/Documents/untitled folder/untitled folderactivity.csv")
Process/transform the data (if necessary) into a format suitable for your analysis
## subset data frame to values without na for later use
without_na <- activity[complete.cases(activity),]
1) Make a histogram of the (total number of (steps taken each day))
## (total number of (steps taken per day))
total <- aggregate(steps ~ date, without_na, sum)

## add descriptive variable names
names(total)[2] <- "sum_steps"

## check out new data frame
head(total, 5)
##         date sum_steps
## 1 2012-10-02       126
## 2 2012-10-03     11352
## 3 2012-10-04     12116
## 4 2012-10-05     13294
## 5 2012-10-06     15420
## plot histogram, using breaks purely for better visuals.
hist(
        total$sum_steps,
        col = "green",
        main = "Histogram of the (Total Number of (Steps Taken Each Day))",
        xlab = "(Total Number of (Steps Taken Each Day))",
        breaks = 20
)

2) Calculate and report the mean and median (total number of (steps taken per day))
## mean
mean(total$sum_steps)
## [1] 10766
## median
median(total$sum_steps)
## [1] 10765
What is the average daily activity pattern?
Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
## the average number of steps taken, averaged across all days for each 5-minute
## interval
interval <- aggregate(steps ~ interval, without_na, mean)

## add descriptive variable names
names(interval)[2] <- "mean_steps"

## check out new data frame
head(interval, 5)
##   interval mean_steps
## 1        0    1.71698
## 2        5    0.33962
## 3       10    0.13208
## 4       15    0.15094
## 5       20    0.07547
## format plot margins to accommodate long text labels.
par(mai = c(1.2,1.5,1,1))

## plot time series
plot(
        x = interval$interval,
        y = interval$mean_steps,
        type = "l",
        main = "Time Series Plot of the 5-Minute Interval\n and the Average Number of Steps Taken, Averaged Across All Days",
        xlab = "5-Minute Interval",
        ylab = "Average Number of Steps Taken,\n Averaged Across All Days"
)



2) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
interval[interval$mean_steps==max(interval$mean_steps),]
##     interval mean_steps
## 104      835      206.2
Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
nrow(activity[is.na(activity$steps),])
## [1] 2304
Create a new dataset that is equal to the original dataset but with the missing data filled in.
## merge original activity data frame with interval data frame
newactivity <- merge(activity, interval, by = 'interval', all.y = F)

## merge NA values with averages rounding up for integers
newactivity$steps[is.na(newactivity$steps)] <- as.integer(
        round(newactivity$mean_steps[is.na(newactivity$steps)]))

## drop and reorder columns to match original activity data frame
## http://stackoverflow.com/questions/4605206/drop-columns-r-data-frame
keeps <- names(activity)
newactivity <- newactivity[keeps]
Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.
## (total number of (steps taken per day))
newtotal <- aggregate(steps ~ date, newactivity, sum)

## add descriptive variable names
names(newtotal)[2] <- "sum_steps"

## check out new data frame
head(newtotal, 5)
##         date sum_steps
## 1 2012-10-01     10762
## 2 2012-10-02       126
## 3 2012-10-03     11352
## 4 2012-10-04     12116
## 5 2012-10-05     13294



## plot histogram, using breaks purely for better visuals.
hist(
        newtotal$sum_steps,
        col = "blue",
        main = "Histogram of the (Total Number of (Steps Taken Each Day))\nPart Deux",
        xlab = "(Total Number of (Steps Taken Each Day)) Part Deux",
        breaks = 20
)

## mean
mean(newtotal$sum_steps)
## [1] 10766
## median
median(newtotal$sum_steps)
## [1] 10762
Do these values differ from the estimates from the first part of the assignment?
They do differ, but ever so slightly.
mean(total) = 10766.19, while mean(newtotal) = 10765.64. Rounding produces the same value.
median(total) = 10765, while median(newtotal) = 10762. 3 step difference.
What is the impact of imputing missing data on the estimates of the total daily number of steps?
This seems to highly depend on how you impute the missing data. Since I used the average for a given interval, there was practically no difference because we basically pulled the averages closer to the inserted average value.
Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels - “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
## create new data frame
newnewactivity <- newactivity

## set up logical/test vector
## https://class.coursera.org/repdata-004/forum/thread?thread_id=34#post-472
weekend <- weekdays(as.Date(newnewactivity$date)) %in% c("Saturday", "Sunday")

## Fill in weekday column
## https://class.coursera.org/repdata-004/forum/thread?thread_id=34#post-125
newnewactivity$daytype <- "weekday"

## replace "weekday" with "weekend" where day == Sat/Sun
## https://class.coursera.org/repdata-004/forum/thread?thread_id=34#post-472
newnewactivity$daytype[weekend == TRUE] <- "weekend"

## convert new character column to factor
newnewactivity$daytype <- as.factor(newnewactivity$daytype)

## Check out new data frame
str(newnewactivity)
## 'data.frame':    17568 obs. of  4 variables:
##  $ steps   : int  2 0 0 0 0 0 0 0 0 0 ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 54 28 37 55 46 20 47 38 56 ...
##  $ interval: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ daytype : Factor w/ 2 levels "weekday","weekend": 1 1 2 1 2 1 2 1 1 2 ...
head(newnewactivity, 5)
##   steps       date interval daytype
## 1     2 2012-10-01        0 weekday
## 2     0 2012-11-23        0 weekday
## 3     0 2012-10-28        0 weekend
## 4     0 2012-11-06        0 weekday
## 5     0 2012-11-24        0 weekend
## double check
## https://class.coursera.org/repdata-004/forum/thread?thread_id=34#post-472
weekdays(as.Date(newnewactivity$date[3]))
## [1] "Sunday"
Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).
## the average number of steps taken, averaged across all days for each 5-minute
## interval
newinterval <- aggregate(steps ~ interval + daytype, newnewactivity, mean)

## add descriptive variable names
names(newinterval)[3] <- "mean_steps"

## check out new data frame
head(newinterval, 5)
##   interval daytype mean_steps
## 1        0 weekday    2.28889
## 2        5 weekday    0.40000
## 3       10 weekday    0.15556
## 4       15 weekday    0.17778
## 5       20 weekday    0.08889
## plot time series
library(lattice)
xyplot(
        mean_steps ~ interval | daytype,
        newinterval,
        type = "l",
        layout = c(1,2),
        main = "Time Series Plot of the 5-Minute Interval\nand the Average Number of Steps Taken,\nAveraged Across All Weekday Days or Weekend Days",
        xlab = "5-Minute Interval",
        ylab = "Average Number of Steps Taken"
)


