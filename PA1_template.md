# Reproducible Research: Peer Assessment 1

Loaded ggplot2 and lattice packages for graphics generation.     
Loaded plyr package for ddply function.


```r
library(ggplot2)
library(plyr)
library(lattice)
```


```r
dateTime = Sys.time()
```

This analysis was last run on 2014-07-20 16:47:24

## The Dataset used for the assignment

The data for this assignment was obtained from a personal activity monitoring  
device. Over the course of two months, October and November 2012, an anonmymous  
individual used the device to collect personal movement data that included the  
number of steps taken in 5 minute intervals each day. 

The steps data for this assignment can be downloaded from Dr. Roger Peng's   
Reproducible Research course website.

+ **Dataset:** [Activity monitoring data][1] [52K]
[1]: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip  
"Activity monitoring data"

The dataset includes three variables:

1. **steps:**  The number of steps that were taken by the individual in each  
5-minute interval (missing values are represented by NA)

2. **data:**  The date on which the data was acquired in YYYY-MM-DD format
    
3. **interval:**  An identifier for the 5-minute interval in which the data was  
acquired

The dataset is stored in a comma-separated-value (CSV) file. There are a total  
of 17,568 observations.

## Goals of the Assignment

In this assignment, the steps data was used to answer the following three  
questions:

1. What was the mean total number of steps taken per day? (Missing values not  
imputed)

2. What was the average daily activity pattern? (Missing values not imputed)

3. What was the mean total number of steps taken per day? (Missing values   
imputed)

4. Are there differences in activity patterns between weekdays and weekends?  
(Missing values imputed)

## Loading and preprocessing the data

### Download data and read from csv file into data frame

While, the github repository does contain the activity data in a zip file, I  
opted to download the data from  the Course website cited above and stored   
"activity.csv"", in a directory named Steps_Data


```r
if(!file.exists("Steps_Data")) {
    dir.create("Steps_Data")
}

zipURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(zipURL,"./Steps_Data/temp",method = "curl")

unzip("./Steps_Data/temp", exdir = "./Steps_Data/")
unlink("./Steps_Data/temp")
```

Data was read from CSV file into data frame and the head of data frame   
displayed.


```r
stepData <- read.table("./Steps_Data/activity.csv", header = TRUE, sep = ",",
                       na.strings = "NA", 
                       colClasses = c("integer","character","integer"))
head(stepData)
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

### Preprocess the data

Because dates were loaded into the data frame with class "character", it was   
necessary to convert them to Date objects


```r
stepData$date <- as.Date(stepData$date)
```

## What is mean total number of steps taken per day?

Before Generating a  histogram of the total number of steps taken per day,   
ddply() was used to apply the sum function to the data frame subsetted by date. 


```r
totalSteps <- ddply(stepData, c("date"),summarize,  
                    Frequency = sum(steps, na.rm = TRUE))

grob <- ggplot(totalSteps, aes(x = date, y = Frequency))
grob <- grob + geom_bar(stat = "identity")
grob + labs( title = "Total Number of Steps Taken Per Day", 
                     x = "Date", y = "Number of Steps")
```

![plot of chunk totalStepsPerDay](figure/totalStepsPerDay.png) 

The mean total number of steps taken per day was calculated. 


```r
mean(totalSteps$Frequency)
```

```
## [1] 9354
```

The median total number of steps taken pre day was calculated.  


```r
median(totalSteps$Frequency)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

Before Generating a time series of the mean number of steps taken per interval,   
ddply() was used to apply the mean function to the data frame subsetted by  
the 5-minute intervals. 


```r
meanSPI <- ddply(stepData, c("interval"),summarise,  
                 mean = mean(steps, na.rm = TRUE))

timeGrob <- ggplot(meanSPI, aes(x = interval, y = mean)) 
timeGrob <- timeGrob + geom_line() + geom_point()
timeGrob + labs(title = "Mean Number of Steps Taken per Five-Minute Time Interval",
                x = "Interval", y = "Mean Number of Steps")
```

![plot of chunk timeSeries](figure/timeSeries.png) 

The 5-interval across all days in the datataset which contains the maximum   
number of steps taken was determined.


```r
meanSPI$interval[which.max(meanSPI$mean)]
```

```
## [1] 835
```

## Imputing missing values

Before devising a reasonable strategy for imputing the missing data values,     
the distribution of missing values(NAs) in the data set was determined.

The total number of missing values in the data set was calculated.


```r
sum(is.na(stepData$steps))
```

```
## [1] 2304
```

The percentage of missing values in the data set was calculated.


```r
sum(is.na(stepData$steps))/length(stepData$steps)
```

```
## [1] 0.1311
```

The percentage of NA values for each date was calculated.


```r
dailyNAs <- ddply(stepData, c("date"),summarize,
                  percent_NA = 100 * sum(is.na(steps))/length(steps))

day_of_week <- weekdays(dailyNAs$date)
dailyNAs <- cbind(day_of_week, dailyNAs)
print(dailyNAs)
```

```
##    day_of_week       date percent_NA
## 1       Monday 2012-10-01        100
## 2      Tuesday 2012-10-02          0
## 3    Wednesday 2012-10-03          0
## 4     Thursday 2012-10-04          0
## 5       Friday 2012-10-05          0
## 6     Saturday 2012-10-06          0
## 7       Sunday 2012-10-07          0
## 8       Monday 2012-10-08        100
## 9      Tuesday 2012-10-09          0
## 10   Wednesday 2012-10-10          0
## 11    Thursday 2012-10-11          0
## 12      Friday 2012-10-12          0
## 13    Saturday 2012-10-13          0
## 14      Sunday 2012-10-14          0
## 15      Monday 2012-10-15          0
## 16     Tuesday 2012-10-16          0
## 17   Wednesday 2012-10-17          0
## 18    Thursday 2012-10-18          0
## 19      Friday 2012-10-19          0
## 20    Saturday 2012-10-20          0
## 21      Sunday 2012-10-21          0
## 22      Monday 2012-10-22          0
## 23     Tuesday 2012-10-23          0
## 24   Wednesday 2012-10-24          0
## 25    Thursday 2012-10-25          0
## 26      Friday 2012-10-26          0
## 27    Saturday 2012-10-27          0
## 28      Sunday 2012-10-28          0
## 29      Monday 2012-10-29          0
## 30     Tuesday 2012-10-30          0
## 31   Wednesday 2012-10-31          0
## 32    Thursday 2012-11-01        100
## 33      Friday 2012-11-02          0
## 34    Saturday 2012-11-03          0
## 35      Sunday 2012-11-04        100
## 36      Monday 2012-11-05          0
## 37     Tuesday 2012-11-06          0
## 38   Wednesday 2012-11-07          0
## 39    Thursday 2012-11-08          0
## 40      Friday 2012-11-09        100
## 41    Saturday 2012-11-10        100
## 42      Sunday 2012-11-11          0
## 43      Monday 2012-11-12          0
## 44     Tuesday 2012-11-13          0
## 45   Wednesday 2012-11-14        100
## 46    Thursday 2012-11-15          0
## 47      Friday 2012-11-16          0
## 48    Saturday 2012-11-17          0
## 49      Sunday 2012-11-18          0
## 50      Monday 2012-11-19          0
## 51     Tuesday 2012-11-20          0
## 52   Wednesday 2012-11-21          0
## 53    Thursday 2012-11-22          0
## 54      Friday 2012-11-23          0
## 55    Saturday 2012-11-24          0
## 56      Sunday 2012-11-25          0
## 57      Monday 2012-11-26          0
## 58     Tuesday 2012-11-27          0
## 59   Wednesday 2012-11-28          0
## 60    Thursday 2012-11-29          0
## 61      Friday 2012-11-30        100
```

A brief examination of the the dailyNAs data frame (above) revealed that each   
date has either all missing values (100%) or no missing values (0%).   
Out of the 61 dates, there are 8 dates for which all of the values are missing.   
Two of these dates fall on a Monday, two on a Friday, and one each on Wednesday,   
Thursday,Saturday, and Sunday. Based on these results, the following imputation   
strategy was devised:

If the number of steps for a five-minute interval is missing, the imputed value         
will be the mean of the five-minute interval for that particular day (e.g. if     
the missing value occurs at interval 4 on a Sunday,the imputed value will be the    
mean number of steps for interval 4's that occurred on Sundays).

The first step in the imputation process was to generate values that could be   
used for imputation. It was noted that for each day of the week, the steps were  
counted for 288 five-minute intervals. It was therefore necessary to generate  
means for the 288 intervals that occured on Sundays, the 288 intervals that    
occured on Mondays, etc. Therefore 288 * 7 = 2016 means were generated. Note,      
although there are no missing Tuesday values, mean values for Tuesday intervals   
were still calculated. 


```r
day <- factor(
    weekdays(stepData$date),
    c("Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"))
stepData <- cbind(day,stepData)
imputationFrame <- ddply(stepData, c("day","interval"), summarize,
                          mean = mean(steps, na.rm = TRUE))
```

Data collection began on 2012-10-01, a Monday, and ended on 2012-11-30, a   
Friday. Data was collected for 61 days with the steps recorded for 288  
five-minute intervals each day for a total of 17568 observations. The imputation   
table contains the means of the 288 intervals for each of the 7 days   
(Monday- Sunday) resulting in 2016 rows. To vectorize the missing value  
replacement operation, a vector of length 17568 from the imputationFrame's    
mean number of steps column was created.


```r
impVec <- c(rep(imputationFrame$mean, times = 8),imputationFrame$mean[1:1440])
```

Missing value in the step data were replaceds with imputed values.


```r
stepData$steps[is.na(stepData$steps)] <- impVec[is.na(stepData$steps)]
```

Before Generating a  histogram of the total number of steps taken per day,   
ddply() was used to apply the sum function to the data frame subsetted by date. 


```r
totalSteps <- ddply(stepData, c("date"),summarize,  
                    Frequency = sum(steps, na.rm = TRUE))

grob <- ggplot(totalSteps, aes(x = date, y = Frequency))
grob <- grob + geom_bar(stat = "identity")
grob + labs( title = "Total Number of Steps Taken Per Day", 
                     x = "Date", y = "Number of Steps")
```

![plot of chunk totalStepsPerDayImputed](figure/totalStepsPerDayImputed.png) 

The mean total number of steps taken per day was calclated.


```r
mean(totalSteps$Frequency)
```

```
## [1] 10821
```

The median total number of steps taken per day was calculated.   


```r
median(totalSteps$Frequency)
```

```
## [1] 11015
```

To calculate the mean, the total number of steps taken each day are summed and   
divided by the number of days (61). Prior to imputation, the 8 days composed    
entirely of NAs contributed nothing to the numerator. After imputation, each of     
these 8 days contributed positvely to the numerator. Therefore, the mean    
estimated after imputation is greater than the mean estimated before imputation.

The median total number of steps can be determined by arranging the total number   
of steps taken each day in ascending order and selecting the middle value. In   
this case there were 61 vaules so the median corresponds to the 31st value.   
Prior to imputation, the 8 days composed entirely of NAs were assigned values of   
0 placing beloow the median. After imputation, it was possible that the now all  
positive values for the 8 days would still fall below the originally-calculated   
median. If this had been the case, the median would have remained the same.    
Since the median was greater after imputation, it can be inferred    
after imputation, it can be inferred that at least one of the days now has a    
value greater than the originally calculated median.  

## Are there differences in activity patterns between weekdays and weekends?

A new factor variable, "day", with two levels - "weekday" and "weekend" was   
created. 


```r
stepData$day <- as.character(stepData$day)
stepData$day <- ifelse((stepData$day == "Saturday") | (stepData$day == "Sunday")
                       ,"weekend","weekday")
stepData$day <- factor(stepData$day)
```

Before genearting a time series plot of the five-minute interval and the average    
number of steps taken (averaged across all weekday days or weekend days),    
ddply() was used to apply the mean function to the data subsetted by type of day    
(weekend or weekday), and time interval. 


```r
meanSPI_weekend_vs_weekday <- ddply(stepData, c("day","interval"),summarise,  
                 mean = mean(steps))

xyplot(mean ~ interval | day, data = meanSPI_weekend_vs_weekday, type = "b",
       main = "Mean Number of Steps Taken per Five-Minute Time Interval\nOn Weekdays vs Weekends", 
       ylab = "Number of Steps", layout = c(1,2))
```

![plot of chunk timeSeries_weekend_vs_weekday](figure/timeSeries_weekend_vs_weekday.png) 

## Software Environment 


```r
sessionInfo()
```

```
## R version 3.0.3 (2014-03-06)
## Platform: x86_64-apple-darwin10.8.0 (64-bit)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] lattice_0.20-29 plyr_1.8.1      ggplot2_1.0.0   knitr_1.6      
## 
## loaded via a namespace (and not attached):
##  [1] colorspace_1.2-4 digest_0.6.4     evaluate_0.5.5   formatR_0.10    
##  [5] grid_3.0.3       gtable_0.1.2     labeling_0.2     MASS_7.3-33     
##  [9] munsell_0.4.2    proto_0.3-10     Rcpp_0.11.2      reshape2_1.4    
## [13] scales_0.2.4     stringr_0.6.2    tools_3.0.3
```
