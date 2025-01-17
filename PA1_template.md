---
title: "Reproducible Research Project 1"
author: "Ummara Rasheed"
date: "2024-02-10"
output: html_document
---



## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:
#load the necessary library

```r
library(ggplot2)
```
#load the data

```r
fullData <- read.csv("activity.csv")
```
#transform the data

```r
fullData$date <- as.Date(fullData$date, "%Y-%m-%d")
```
# A. What is mean total number of steps taken per day?
1. Total steps per day calculation

```r
stepsPerDay <- aggregate(steps ~ date, fullData, FUN = sum)
```

2. create histogram, Histogram of the total number of steps taken each day

```r
g <- ggplot (stepsPerDay, aes (x = steps))
g + geom_histogram(fill = "blue", binwidth = 1000) +
    labs(title = " Histogram of Steps Taken Each Day ", x = "Steps", y = "Frequency")
```

![plot of chunk unnamed-chunk-35](figure/unnamed-chunk-35-1.png)
3. Mean and median of the total number of steps taken per day
#Mean of steps

```r
stepsMean <- mean(stepsPerDay$steps, na.rm=TRUE)
stepsMean
```

```
## [1] 10766.19
```
#median of steps

```r
stepsMedian <- median(stepsPerDay$steps, na.rm=TRUE)
stepsMedian
```

```
## [1] 10765
```
The mean and median of the total number of steps taken per day are 1.0766189^{4} and 10765 respectively

B. What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
#create average number of steps by 5-min interval

```r
stepsPerInterval <- aggregate(steps ~ interval, fullData, mean)
```
#Create a time series plot of average number of steps per interval
#annotate the plot

```r
h <- ggplot (stepsPerInterval, aes(x=interval, y=steps))
h + geom_line()+ labs(title = " Time Series Plot of Average Steps per Interval", x = "Interval", y = "Average Steps across All Days")
```

![plot of chunk unnamed-chunk-39](figure/unnamed-chunk-39-1.png)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
#Max steps by interval

```r
maxInterval <- stepsPerInterval[which.max(stepsPerInterval$steps), ] 
maxInterval
```

```
##     interval    steps
## 104      835 206.1698
```
C. Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
noMissingValue <- nrow(fullData[is.na(fullData$steps),])
noMissingValue
```

```
## [1] 2304
```
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
fullData1 <- read.csv("activity.csv", header=TRUE,sep=",")
```
#variable/column with weekdays name

```r
fullData1$day <- weekdays(as.Date(fullData1$date))
```

# create average number of steps per 5-min interval and day

```r
stepsAvg1 <- aggregate(steps ~ interval + day, fullData1, mean)
```

#Dataset with all NAs for substitution

```r
nadata <- fullData1 [is.na(fullData1$steps),]
```

#Merge NAs dataset with the average steps based on 5-min interval+weekday

```r
newdata1 <- merge(nadata, stepsAvg1, by=c("interval", "day"))
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
#Data without NAs

```r
cleanData <- fullData1 [!is.na(fullData1$steps),]
```


```r
#Reorder the new substituted data in the same format as the clean data set (Leave out the NAs column which will be substituted by the average steps based on 5-min interval + day) 
newdata2 <- newdata1[,c(5,4,1,2)]
colnames(newdata2) <- c("steps", "date", "interval", "day")
```



```r
# Merge the new average data (NAs) with the dataset without NAs
mergeData <- rbind (cleanData, newdata2)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
#Total steps per day on the merged data

```r
stepsPerDayFill <- aggregate(steps ~ date, mergeData, FUN = sum)
```

# Create the histogram

```r
g1 <- ggplot (stepsPerDayFill, aes (x = steps))
g1 + geom_histogram(fill = "yellow", binwidth = 1000) +
    labs(title = " Histogram of Steps Taken Each Day ", x = "Steps", y = "Frequency")
```

![plot of chunk unnamed-chunk-51](figure/unnamed-chunk-51-1.png)
#Total steps with imputed data

```r
stepsMeanFill <- mean(stepsPerDayFill$steps, na.rm=TRUE)
stepsMeanFill
```

```
## [1] 10821.21
```
#Median 

```r
stepsMedianFill <- median(stepsPerDayFill$steps, na.rm=TRUE)
stepsMedianFill
```

```
## [1] 11015
```
Conclusion: The new mean of the imputed data is 1.082121^{4} steps compared to the old mean of 1.0766189^{4} steps. Which creates the difference of 55.0209226 steps on average per day.The new median of the imputed data is 1.1015^{4} steps as compared to the old median of 10765 steps. That creates a difference of 250 steps for the median.So, the overall shape of the distribution has not changed.

D. Are there differences in activity patterns between weekdays and weekends?
#new variable creating 

```r
mergeData$DayType <- ifelse(mergeData$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")
```
2. Make a panel plot containing a time series plot (i.e. 
type = "l"
type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
stepsPerIntervalDT <- aggregate(steps ~ interval+DayType, mergeData, FUN = mean)
```


```r
j <- ggplot (stepsPerIntervalDT, aes(x=interval, y=steps))
j + geom_line()+ labs(title = " Time Series Plot of Average Steps per Interval: weekdays vs. weekends", x = "Interval", y = "Average Number of Steps") + facet_grid(DayType ~ .)
```

![plot of chunk unnamed-chunk-56](figure/unnamed-chunk-56-1.png)

