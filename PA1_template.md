# PA1_template.Rmd
Rosina Plomp  
October 26, 2017  


## Reproducible Research - Assignment week 2##

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

#Loading and Preprocessing the data
First, I downloaded the data from the cloudfront website.


```r
## Loading and preprocessing the data
  #download file
    setwd("F:/Coursera - Data Science/Course 5/week 2")
    ifelse(file.exists("activity_monitoring_data.zip"),
           print("file exists"), 
           download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile="activity_monitoring_data.zip")
    )
```

```
## [1] "file exists"
```

```
## [1] "file exists"
```

```r
    ifelse(file.exists("activity"),
           print("file unzipped"),
           unzip("activity_monitoring_data.zip")
    )
```

```
## [1] "./activity.csv"
```

```r
  #read in data
    data <- read.csv("activity.csv", header=TRUE)
  #load libraries
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

#What is mean total number of steps taken per day?
I investigated the total number of steps taken per day, and generated the histogram seen below, which shows that the number of steps are clustered around a mean of 10766.19 (median: 10765) and range from 41 to 21194. 


```r
  #calculate the total nr of steps taken per day
    data_grouped <- group_by(data, date)
    nrsteps <- summarize(data_grouped, steps_tot=sum(steps))
      
  #histogram of nr of steps per day
    hist(nrsteps$steps_tot, col="red",breaks=25,xlab="number of steps per day",main="number of steps per day") 
```

![](PA1_template_files/figure-html/total steps per day-1.png)<!-- -->

```r
  #mean and median
    mean_steps <- mean(nrsteps$steps_tot,na.rm=TRUE)
    median_steps <- median(nrsteps$steps_tot,na.rm=TRUE)
```
#What is the average daily activity pattern?
Upon plotting the average nr of steps per 5-minute interval against the time on an average day, It was clear that minimal activity took place during the first 8 hours, and activity peaked around 14 hours after the start of measurement (at 835 minutes). 

```r
#time-series plot of the 5-minute interval and average nr of steps taken
    data_grouped<-group_by(data, interval)
    interval_data<- summarize(data_grouped, steps_mean=mean(steps,na.rm=TRUE))
    plot(interval_data$interval,interval_data$steps_mean,type="l",xlab="time(min)",ylab="average nr of steps", main="daily activity pattern")
```

![](PA1_template_files/figure-html/daily activity-1.png)<!-- -->

```r
  #find 5-minute interval with the maximum nr of steps
    max_steps<- interval_data[which(interval_data$steps_mean==max(interval_data$steps_mean)),1]
```

#Imputing missing values
There were 2304 missing values in the dataset. I replaced these values with the mean number of steps per interval, which did not significantly alter the mean and median of the values (mean: 1076.19, median: 10766.19).


```r
#calculate and report the total number of missing values in the dataset
    nrNAs <- sum(is.na(data$steps))

    #Create a new dataset that is equal to the original dataset but with the missing data filled in.
    NArows <-which(is.na(data$steps))
    newdata <-data
    newdata[NArows,1]<-mean(data$steps,na.rm=TRUE)
    
    #Make a histogram of the total number of steps taken each day and 
    #Calculate and report the mean and median total number of steps taken per day. 
    #Do these values differ from the estimates from the first part of the assignment? 
    #What is the impact of imputing missing data on the estimates of the total daily number of steps?
    newdata_grouped <- group_by(newdata, date)
    nrstepsnew <- summarize(newdata_grouped, steps_tot=sum(steps))
    hist(nrstepsnew$steps_tot, col="green",breaks=25,xlab="number of steps per day",main="number of steps per day") 
```

![](PA1_template_files/figure-html/imputing NAs-1.png)<!-- -->

```r
    newmean_steps <- mean(nrstepsnew$steps_tot,na.rm=TRUE)
    newmedian_steps <- median(nrstepsnew$steps_tot,na.rm=TRUE)
```

##Are there differences in activity patterns between weekdays and weekends?
Finally, I plotted the activity patterns (i.e. average number of steps per five minute interval over time) separately for weekdays and weekends. This showed that weekdays typically show an earlier and higher peak in activity, while weekends have a slower start and overall lower level of activity, which would be consistent with the hypothesis that people like to sleep in and relax during the weekend.


```r
    #Create a new factor variable in the dataset with two levels - "weekday" and "weekend" 
    #indicating whether a given date is a weekday or weekend day.
    newdata$day <- "weekday" #create factor variable and set it to weekday
    #convert dates to class date
    newdata$date <- strptime(as.Date(newdata$date), "%Y-%m-%d", tz="UTC")
    for(i in 1:length(newdata$day)){
      if(weekdays(newdata$date[i]) %in% c("Sunday","Saturday")){
        newdata$day[i]<- "weekend"
      }
    }
    
    #Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) 
    #and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 
    #See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.    
    newdata_weekend <- subset(newdata[,c(1,3,4)],newdata$day=="weekend")
    newdata_weekday <- subset(newdata[,c(1,3,4)],newdata$day=="weekday")
    newdata_weekend_grouped<-group_by(newdata_weekend, interval)
    newdata_weekday_grouped<-group_by(newdata_weekday, interval)
    interval_data_weekend<- summarize(newdata_weekend_grouped, steps_mean=mean(steps,na.rm=TRUE))
    interval_data_weekday<- summarize(newdata_weekday_grouped, steps_mean=mean(steps,na.rm=TRUE))
    par(mfrow =c(1,2), mar=c(4,4,2,1)) 
    plot(interval_data_weekday$interval,interval_data_weekday$steps_mean,type="l",main="Weekday",xlab="time(min)",ylab="average nr of steps")
    plot(interval_data_weekend$interval,interval_data_weekend$steps_mean,type="l",main="Weekend",xlab="time(min)",ylab="average nr of steps")
```

![](PA1_template_files/figure-html/weekdays and weekends-1.png)<!-- -->


