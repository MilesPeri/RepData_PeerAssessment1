Loading and preprocessing the data
----------------------------------

    library(dplyr)
    library(lubridate)
    library(ggplot2)

    setwd("~/Curso/Reproducible research")
    activity <- read.csv(unzip("activity.zip"))
    str(activity)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

    activity <- activity %>% mutate(time=interval/100) 
    activity <- activity %>% mutate(date=ymd(date))

What is mean total number of steps taken per day?
-------------------------------------------------

Calculate the total number of steps taken per day

    steps_per_day <- activity %>% group_by(date) %>% 
      summarise(steps=sum(steps, na.rm=T))

Make a histogram of the total number of steps taken each day

    plot1=hist(steps_per_day$steps, breaks = 30, xlab = "Total steps by day", col = "pink", main = "Steps")

![](project-course-1_files/figure-markdown_strict/unnamed-chunk-3-1.png)

Calculate and report the mean and median of the total number of steps
taken per day

    mean(steps_per_day$steps)

    ## [1] 9354.23

    median(steps_per_day$steps)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

Time series plo tof the 5-minute interval (x-axis) and the average
number of steps taken, averaged across all days (y-axis)

    activity %>% group_by(time) %>% 
      summarise(mean=mean(steps, na.rm=T)) %>% ggplot()+
      geom_line(aes(x=time, y=mean), color="green")+
      labs(title = "mean steps per 5 minutes interval", x="time of the day", y="mean steps")

![](project-course-1_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

    activity %>% group_by(time) %>% 
      summarise(mean=mean(steps, na.rm=T)) %>% 
      slice_max(mean)

    ## # A tibble: 1 x 2
    ##    time  mean
    ##   <dbl> <dbl>
    ## 1  8.35  206.

Imputing missing values
-----------------------

Calculate and report the total number of missing values in the dataset

    activity %>% summarise(nas=sum(is.na(steps)))

    ##    nas
    ## 1 2304

Devise a strategy for filling in all of the missing values in the
dataset. I use de 5 minutes mean for filling missing values.

Create a new dataset that is equal to the original dataset but with the
missing data filled in.

    activity_filled <- activity %>% group_by(time) %>% 
      mutate(imput=round(mean(steps, na.rm=T))) %>% 
      mutate(imput=as.integer(imput)) %>% 
      mutate(steps=case_when(is.na(steps)~imput,
                             TRUE~steps)) %>% 
      select(c(1:4))

Make a histogram of the total number of steps taken each day and
Calculate and report the mean and median total number of steps taken per
day. Do these values differ from the estimates from the first part of
the assignment? What is the impact of imputing missing data on the
estimates of the total daily number of steps?

    steps_per_day2 <- activity_filled %>% group_by(date) %>% 
      summarise(steps=sum(steps))
    plot2=hist(steps_per_day2$steps, breaks = 30, xlab = "Total steps by day", col = "turquoise", main = "Steps")

![](project-course-1_files/figure-markdown_strict/unnamed-chunk-9-1.png)

    mean(steps_per_day2$steps)

    ## [1] 10765.64

    median(steps_per_day2$steps)

    ## [1] 10762

There is an increase in the mean and the median of daily steps when
accounting for missing data, that is because the number of days with
very few steps is reduced.

Also the center value of 10000 steps is much more frequent

    par(mfrow=1:2)
    hist(steps_per_day$steps, breaks = 30, xlab = "Total steps by day", col = "pink", main = "Steps with NAs", ylim=c(0,15))
    hist(steps_per_day2$steps, breaks = 30, xlab = "Total steps by day", col = "turquoise", main = "Steps without NAs")

![](project-course-1_files/figure-markdown_strict/unnamed-chunk-10-1.png)

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create a new factor variable in the dataset with two levels – “weekday”
and “weekend” indicating whether a given date is a weekday or weekend
day.

    Sys.setlocale("LC_TIME", "English")

    ## [1] "English_United States.1252"

    activity_filled=activity_filled %>% mutate(week_days=weekdays(date)) %>% 
      mutate(weekend=case_when(week_days=="Saturday"|week_days=="Sunday"~"weekend",
                               TRUE~"weekdays")) %>% 
      mutate(weekend=as.factor(weekend))

Make a panel plot containing a time series plot (i.e. type = "l") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis)

    activity_filled %>% group_by(weekend,time) %>% 
      summarise(mean=mean(steps)) %>% ggplot()+
      facet_grid(weekend~.)+
      geom_line(aes(x=time, y=mean), col="violet", lwd=1.2)+
      labs(title = "mean steps per 5 minutes interval", x="time of the day", y="mean steps")

![](project-course-1_files/figure-markdown_strict/unnamed-chunk-12-1.png)
