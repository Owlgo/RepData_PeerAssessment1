# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
echo=TRUE
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.0.3
```

```r
#
data<-read.csv("..\\activity.csv")
dates<-unique(data$date)
ok<-complete.cases(data$steps, data$date)
data_ok<-data[ok,]
```


## What is mean total number of steps taken per day?


```r
step <- 1:length(dates)
for(i in 1:length(dates)){
        step[i]<-sum(data_ok$steps[data_ok$date == dates[i]])
}
#    
```
a histogram of the total number of steps taken each day

```r
ggplot(NULL, aes(x=step))+
        geom_histogram() +xlab("Total Number of Steps per Day")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

# mean  of the total number of steps taken per day


```r
mean(step)
```

```
## [1] 9354.23
```

# median of the total number of steps taken per day


```r
median(step)
```

```
## [1] 10395
```

## What is the average daily activity pattern?


```r
intr<-unique(data_ok$interval)  # the 5 minute interval values used
#
# Calculate the means for each interval
#
intrvl_mean<-1:length(intr)
for(i in 1:length(intr)){
        intrvl_mean[i]<-mean(data_ok$steps[data_ok$interval == intr[i]])
}
#
#
ggplot(NULL,aes(x=intr, y=intrvl_mean))+
        geom_line() + xlab("Interval") + ylab("Mean Number of Steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

## Interval with the maximum average number of steps

```r
max_ave_steps<-intr[which(intrvl_mean==max(intrvl_mean))]  
max_ave_steps
```

```
## [1] 835
```

## Total number of missing values in the dataset


```r
num_missing_values<-sum(!ok)
num_missing_values
```

```
## [1] 2304
```

## Inputing missing values with the median values for that time interval


```r
# Calculate the median for each interval
#
intrvl_median<-1:length(intr)
for(i in 1:length(intr)){
        intrvl_median[i]<-median(data_ok$steps[data_ok$interval == intr[i]])
}
#
#
ggplot(NULL,aes(x=intr, y=intrvl_median)) + geom_line() + xlab("Interval") + ylab("Interval Median")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 


```r
empty_indx <- which(!ok)
#
data_p_mssng <- data
#
for(i in 1:length(empty_indx)){
        value <- empty_indx[i] %% 288
        if(value == 0) value <- 288
        data_p_mssng$steps[empty_indx[i]] <- intrvl_mean[value]
}     
#
#
step_mssng <- 1:length(dates)
for(i in 1:length(dates)){
        step_mssng[i]<-sum(data_p_mssng$steps[data_p_mssng$date == dates[i]])
}
#        
ggplot(NULL, aes(x=step_mssng))+
        geom_histogram() + xlab("Interval") + ylab("Count (missing data replaced with median)")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

## The mean and median total number of steps taken per day


```r
mean(step_mssng)
```

```
## [1] 10766.19
```

```r
median(step_mssng)
```

```
## [1] 10766.19
```
There are significant differences - 13% - between the means of the original dataset (9354) and the data set filled in with medians (10766). Thee was no significant change - 3% - in he medians (original:10395; filled:10766).  

## Are there differences in activity patterns between weekdays and weekends?

```r
data_p_mssng$date<-as.Date(data_p_mssng$date)
day<-weekdays(data_p_mssng$date)
endday<-day == "Sunday" | day == "Saturday"
day[endday] <- "weekend"
day[!endday] <- "weekday"
#
data_p_mssng$day <- day
data_p_mssng$day<-factor(data_p_mssng$day)
#
daywk<-1:length(intr)
intrvl_mean<-1:length(intr)
for(i in 1:length(intr)){
        intrvl_mean[i]<-mean(data_p_mssng$steps[data_p_mssng$interval == intr[i]&data_p_mssng$day=="weekend"])
        daywk[i] <-"weekend"
}
tend<-data.frame(intr)
tend$intrvl_mean<-intrvl_mean
tend$daywk<-daywk
#
for(i in 1:length(intr)){
        intrvl_mean[i]<-mean(data_p_mssng$steps[data_p_mssng$interval == intr[i]&data_p_mssng$day=="weekday"])
        daywk[i]<-"weekday"
}
tday<-data.frame(intr)
tday$intrvl_mean<-intrvl_mean
tday$daywk<-daywk
#
##intr
#
dfmssng<-rbind(tend, tday)
#
dfmssng$daywk<-factor(dfmssng$daywk)
#
plt<-ggplot(dfmssng, aes(x=intr, y=intrvl_mean)) + geom_line()
plt + facet_grid(daywk ~ .) + xlab("Interval") + ylab("Interval Mean")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 
