First Homework
==============
--- 
title: "First Homework" 
output: 
  html_document: 
    keep_md: true 
---

## 1. Code for reading in the dataset and/or processing the data

```r
library(readr)
library(dtplyr)
library(tidyr)
library(ggplot2)
library(plyr)
library(lattice)
library(knitr)
```

Downloading data

```r
setwd("/Volumes/Backup/Coursera/00 Data Science Specialization/05 Reproducible Research")
filePath <- "/Volumes/Backup/Coursera/00 Data Science Specialization/05 Reproducible Research"
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl,destfile="Dataset.zip",method="curl")
unzip(zipfile="Dataset.zip")
data <- read.csv("activity.csv")
summary(data)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

The variables included in this dataset are:

* steps: Number of steps taking in a 5-minute interval (missing values are coded as 𝙽𝙰)

* date: The date on which the measurement was taken in YYYY-MM-DD format

* interval: Identifier for the 5-minute interval in which measurement was taken

## 2. Histogram of the total number of steps taken each day

```r
data$date <- as.Date(data$date,"%Y-%m-%d")
data$day <- weekdays(as.Date(data$date))
data$DateTime<- as.POSIXct(data$date, format="%Y-%m-%d")
sumData <- aggregate(data$steps ~ data$date, data, FUN="sum")
hist(sumData[,2],breaks=20, 
     xlab="Steps",
     main = "Steps Taken per Day",
     col="gray",
     xlim = c(0,25000),ylim = c(0,12))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## 3. Mean and median number of steps taken each day

```r
meanSteps  <- as.character(round(mean(sumData[,2]),2))
medianSteps <- median(sumData[,2])
```

The mean of steps is 10766.19. The median of steps is 10765.

## 4. Time series plot of the average number of steps taken

```r
myCleanData <- data[!is.na(data$steps),]
interData <- ddply(myCleanData, .(interval), summarize, Avg = mean(steps))
p <- ggplot(interData, 
            aes(x = interval, y = Avg), 
            xlab = "Interval", 
            ylab = "Avg Number of Steps")
p + geom_line() + xlab("Interval") + ylab("Avg Number of Steps") + ggtitle("Avg Number of Steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

## 5. The 5-minute interval that, on average, contains the maximum number of steps

```r
maxStep <- max(interData$Avg)
interval <- interData[interData$Avg == maxStep, 1]
```
The interval is 835.

## 6. Code to describe and show a strategy for imputing missing data

```r
# Number of NAs in original data set
nas <- nrow(data[is.na(data$steps),])
# Average number of steps per weekday and interval
avgData <- ddply(myCleanData, .(interval, day), summarize, Avg = mean(steps))
# Dataset with all NAs for substitution
naData <- data[is.na(data$steps),]
# Merge NA data with average weekday interval for substitution
newData <- merge(naData, avgData, by = c("interval", "day"))
## Reorder the new substituded data in the same format as clean data set
newData2 <- newData[,c(6,4,1,2,5)]
colnames(newData2) <- c("steps", "date", "interval", "day", "DateTime")
# merge the NA averages and non NA data
mergedData <- rbind(myCleanData, newData2)
# Compare NA with non NA data
sumData <- aggregate(data$steps ~ data$date, data, FUN="sum")
sumData2 <- aggregate(mergedData$steps ~ mergedData$date, FUN = sum)
colnames(sumData2)<- c("Date", "Steps")
# average of Steps
avg1 <- as.character(round(mean(sumData[,2]),2))
avg2 <- as.integer(mean(sumData2$Steps))
# median of Steps
median1 <- median(sumData[,2])
median2 <- as.integer(median(sumData2$Steps))
```

The total number of missing values is 2304
The average and median of the original data are 10766.19 and 10765;
The average and median of the cleaned data are 10821 and 11015.
There are differences between both data.

## 7. Histogram of the total number of steps taken each day after missing values are imputed

```r
hist(sumData2[,2],breaks=20, 
     xlab="Steps",
     main = "Steps Taken per Day with NAs fixed",
     col="gray",
     xlim = c(0,25000),ylim = c(0,12))
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

## 8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

```r
# new category based on days of the week
mergedData$DayCategory <- ifelse(mergedData$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")
# summary data by interval and type of day
interData2 <- ddply(mergedData, .(interval, DayCategory), summarize, Avg = mean(steps))
# Panel plot
xyplot(Avg ~ interval|DayCategory, data = interData2, type = "l",  layout = c(1,2),
       main="Average Steps per Interval Based on Type of Day", 
       ylab="Average Number of Steps", xlab = "Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

knit2html("PA1_template.Rmd", force_v1 = TRUE)
