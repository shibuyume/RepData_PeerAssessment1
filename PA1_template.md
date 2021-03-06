# Reproducible Research: Peer Assessment 1

```
## environment set up
### Always make code visible
echo = TRUE 
```
* <b>Dataset</b>: <a href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip">Activity Monitoring data</a> [52kb]


* <b>Loading and preprocessing the data</b>
Steps will omit "NA" and force the second column to be of date format. 

```
fullds <- read.csv("activity.csv", header=TRUE, stringsAsFactor=FALSE, na.strings="NA")
omitNAds <- na.omit(fullds)

omitNAds[,2] <- as.Date(omitNAds[,2], format="%Y-%m-%d")
```


* <b>1. What is mean total number of steps taken per day?</b>
<br>The mean total number of steps taken per day: 10766.19
<br>The median total number of steps taken per day: 10765
<br>
<br>Histogram: 

![Total number of steps per day](/figures/total_number_steps.png)

```
steps_per_day <- aggregate(steps ~ date, omitNAds, sum)

###create histogram
hist(steps_per_day$steps, main="Total Steps by Day", col="red", xlab="Number of Steps")

###calculate mean
steps_mean <-mean(steps_per_day$steps)

###calculate median
steps_median <-median(steps_per_day$steps)
```


* <b>2. What is the average daily activity pattern?</b>
<br>The 5-minute interval <u>835</u>, on average across all the days in the dataset, contains the max number of steps. 

<br>Line Graph: 

![Total steps per interval](/figures/steps_per_interval.png)

```
steps_per_interval <- aggregate(steps ~ interval, omitNAds, mean)

#plot line graph
plot(steps_per_interval, type = "l")

#find which interval has max steps
max_interval <- steps_per_interval[which.max(steps_by_interval$steps),1]

```
* <b>3. Imputing missing values</b>
<br> Total number of missing values in dataset: 2304

```
numNAs<-sum(is.na(fullds))
numNAs
```

<br> Devise a strategy for filling in all of the missing values in the dataset. I will use steps by interval to fill in the missing values in the data set because it is previously calculated. Will append this value as an additional column to the inputds. 

```
inputds<- transform(fullds, steps=ifelse(is.na(fullds$steps), steps_per_interval$steps[match(fullds$interval, steps_per_interval$interval)], fullds$steps))

### new total number of steps per day
new_steps_per_day <- aggregate(steps ~ interval, inputds, sum)

```

<br> New histogram that shows old dataset with NA values on the left, and new dataset with filled values on the right. 

![Old vs New Total No. of Steps](/figures/new_steps_per_day.png)

<br>New mean is 2280.339
<br>New median is 2080.906

```
###plot new histogram
par(mfrow=c(1,2)) 
hist(new_steps_per_day$steps, main="New Total No. of Steps", col="red", xlab="No. of Steps")
###plog aginst old steps per day
hist(steps_per_day$steps, main="Old Total No. of Steps (omit NA)", col="blue", xlab="No. of Steps")

###new mean
mean(new_steps_per_day$steps)

###new median
median(new_steps_per_day$steps)

```

<br>The mean and median values are now different. Due to the availability of "more data" when the NA values are replaced, both the mean and median values have increased. Also, the shape of the distribution has changed and shifted to the left, leading to the mean and median values being quite a bit different from each other, as opposed to the original distribution that has the NA values ignored.

* <b>4. Are there differences in activity patterns between weekdays and weekends?</b>
<br>Create a new factor variable in the dataset 

```
### set weekend
weekends <- c("Saturday", "Sunday")

###make new column to hold day of week
inputds[,4] <- as.factor(weekdays(as.Date(inputds$date)))
colnames(inputds)[4] <- "dayofweek"

###make another new column to hold "weekday" or "weekend"
inputds[,5]<- as.factor(ifelse(inputds$dayofweek %in% weekends, "Weekend", "Weekday"))
colnames(inputds)[5] <- "typeofday"

```
<br>Make panel plot that contains a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

![Average No. of Steps (Weekday vs Weekend)](/figures/ave_step_weekdayend.png)

```
###aggregate the weekday and weekend
steps_per_daytype <- aggregate(steps ~ interval+typeofday, inputds, mean)

###load ggplot2 library to plot graph
library(ggplot2)
qplot(interval, steps, data=steps_per_daytype, geom=c("line"), xlab="Interval", 
      ylab="Number of steps", main="") + facet_wrap(~ typeofday, ncol=1)

```

