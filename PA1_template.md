# Reproducible Research: Peer Assessment 1



## Loading and preprocessing the data
Since the assingment repository contains the "activity.zip" file, it can directly be unzipped. But when the report is knited outside of the repository it should download the "activity.zip" file.

```r
if (!file.exists("activity.zip")){
  download.file(url='https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip', destfile='activity.zip')
}
unzip('activity.zip')
```
To read the activities the `read.csv()` function is used and the `stringsAsFactors` parameter is set to false because the date doesn't need to be treated as factor.

```r
activity <- read.csv("activity.csv", stringsAsFactors=FALSE)
```
During the further chapters the following libraries are used.

```r
library("dplyr")
library("lubridate")
library("ggplot2")
```

## What is mean total number of steps taken per day?
At first the total number of steps taken each day has to be calculated. This is done wit the help of the `summarize` function from the `dplyr` package. To calculate mean and median either the `NA` values have to be removed or `the na.rm` parameter of the mean and the median functions is set to `TRUE`. 

```r
stepsPerDay <- summarise(group_by(activity, date), sum(steps))
names(stepsPerDay)[2] <- "totalSteps"

meanStepsPerDay   <- mean(stepsPerDay$totalSteps, na.rm = TRUE)
medianStepsPerDay <- median(stepsPerDay$totalSteps, na.rm = TRUE)
hist(stepsPerDay$totalSteps, xlab = "Total steps per day", main = "Histogram of steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
<br>
The mean of total steps per day is **10766.19** and the median is **10765.00.**

## What is the average daily activity pattern?


```r
stepsPerInterval <- summarise(group_by(activity, interval), mean(steps, na.rm = TRUE))
names(stepsPerInterval)[2] <- "averageSteps"

plot(stepsPerInterval$interval,stepsPerInterval$averageSteps, type = "l")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
max <- max(stepsPerInterval$averageSteps)
interval <- stepsPerInterval[stepsPerInterval$averageSteps==max,c('interval')]
```
The interval with the most average steps across all the days in the dataset is interval **835**.

## Imputing missing values

```r
nas <- is.na(activity$steps)

missingValues <- sum(nas)

plot(nas)
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
<br>
In the dataset are **2304** values missing.

Based on the assumtion that many activities depend on the weekday, the missing values shold be filled with the average value of the same day and interval.

For that reason the data is grouped by weekday and interval and the mean value is calculated. Then the data frames are merged.

```r
# new dataframe with wekkday attached (1 = sunday)
fill <-cbind(activity,wday(activity$date))
names(fill)[4] = "wday";

# average values by weekday and interval
fillgroup <- summarise(group_by(fill, interval, wday), mean(steps, na.rm = TRUE))
names(fillgroup) <- c("interval","wday","averageSteps")

# merged dataset with average steps
fillMerge <- merge(fill[nas,], fillgroup,by=c("interval","wday"))
fillMerge <- fillMerge[order(fillMerge$date,fillMerge$interval),]

#Independent filled dataset
filledActivity <- activity
filledActivity[is.na(activity$steps),'steps'] <- fillMerge$averageSteps


filledStepsPerDay <- summarise(group_by(filledActivity, date), sum(steps))

hist(filledStepsPerDay$`sum(steps)`)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
meanStepsPerDay <- mean(filledStepsPerDay$`sum(steps)`)
medianStepsPerDay <- median(filledStepsPerDay$`sum(steps)`)
```
The mean of total steps per day of the filled dataset is **10821.21** and the median is **11015.00.**
Both mean and median are have higher values after the data has benn filled in. This is because the mean value of the inserted values is higher then the original mean value.

## Are there differences in activity patterns between weekdays and weekends?

```r
#filledActivity <- filledActivity[["Day"]] 

days <- wday(filledActivity$date)
weekdays <- days > 1 & days < 7
weekends <- !weekdays

filledActivity[weekdays,"day"] <- "weekday"
filledActivity[!weekdays,"day"] <- "weekend"

par(mfrow=c(2,1))
activityPattern<- summarise(group_by(filledActivity[weekdays,], interval,day), mean(steps))
plot(activityPattern$interval,activityPattern$`mean(steps)`, type = "l", main = "Activity pattern weekdayas")
activityPatternWeekEnds <- summarise(group_by(filledActivity[weekends,], interval), mean(steps))
plot(activityPatternWeekEnds$interval,activityPatternWeekEnds$`mean(steps)`, type = "l", main = "Activity pattern weekends")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->
<br>
The main difference between weekdays and weekends is that on weekends there are fewer steps in the morning and more steps in the afternoon.
