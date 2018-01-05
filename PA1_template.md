R Markdown
----------

1.  Set working directory
2.  Clone the github repository to local from R-Studio, using the shell
    command - git clone
3.  Unzip the input file and read (tried reading compressed file
    directly using readr, but it didn't help)
4.  Include dplyr package for data manipulations
5.  Load the data into a tbl\_df and arrange columns, group by and
    summarize
6.  Now, get answer for Q1 - What is mean total number of steps taken
    per day?

Loading and preprocessing the data

    mywd <- setwd("~/R/03. R Projects/Coursera projects/Reproducible Research/Course Project 1")
    mySrcData <- "/home/balman/R/03. R Projects/Coursera projects/Reproducible Research/Course Project 1/RepData_PeerAssessment1"
    setwd(mySrcData)
    myRawData <- read.csv("activity.csv")
    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    my_tib1 <- tbl_df(myRawData)
    my_tib2 <- select(.data = my_tib1, 2:3, 1)
    by_date <- group_by(my_tib2, date)
    by_date_sum <- summarise(by_date, steps_tot = sum(steps,na.rm = TRUE))
    summary(by_date_sum)

    ##          date      steps_tot    
    ##  2012-10-01: 1   Min.   :    0  
    ##  2012-10-02: 1   1st Qu.: 6778  
    ##  2012-10-03: 1   Median :10395  
    ##  2012-10-04: 1   Mean   : 9354  
    ##  2012-10-05: 1   3rd Qu.:12811  
    ##  2012-10-06: 1   Max.   :21194  
    ##  (Other)   :55

2. Plot the histogram on Daily steps count
==========================================

    hist(x = by_date_sum$steps_tot, main = "Daily Avg steps trend", xlab = "Daily steps Count",breaks = 61)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    #png("plot1.png", width=480, height=480)
    #with(by_date_sum, {
    #  hist(x = by_date_sum$steps_tot, main = "Daily Avg steps trend", xlab = "Daily steps Count",breaks = 61)
    #})
    #dev.off()

3. Code to calculate Mean and median number of steps taken each day
===================================================================

    mean_steps_per_day <- mean(by_date_sum$steps_tot)
    median_steps_per_day <- median(by_date_sum$steps_tot)
     mean_steps_per_day

    ## [1] 9354.23

     median_steps_per_day

    ## [1] 10395

4. Time series plot of the average number of steps taken
========================================================

    by_date_avg <- summarise(by_date, steps_avg = mean(steps,na.rm = TRUE))
        plot(aggregate(steps ~ interval, data = by_date, FUN = mean), type = "l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

    #png("plot2.png", width=480, height=480)
    #with(by_date, {
    #    plot(aggregate(steps ~ interval, data = by_date, FUN = mean), type = "l")
    #})
    #dev.off()

The 5-minute interval that, on average, contains the maximum number of steps
============================================================================

    by_interval <- group_by(my_tib2, interval)
    by_interval_sum <- summarise(by_interval, max_steps = max(steps, na.rm = TRUE))
    arrange(by_interval_sum, desc(max_steps))

    ## # A tibble: 288 x 2
    ##    interval max_steps
    ##       <int>     <dbl>
    ##  1      615       806
    ##  2      900       802
    ##  3      550       794
    ##  4      720       789
    ##  5      835       786
    ##  6      925       785
    ##  7     1600       785
    ##  8     1635       785
    ##  9     1140       783
    ## 10      850       781
    ## # ... with 278 more rows

    max_steps <- max(by_interval_sum$max_steps)
    filter(by_interval_sum, max_steps == 806)

    ## # A tibble: 1 x 2
    ##   interval max_steps
    ##      <int>     <dbl>
    ## 1      615       806

The interval that on a daily average had the max number of steps = 615

Code to describe and show a strategy for imputing missing data Replaced
NAs with mean value of steps

    by_date_cleaned <- by_date
    by_date_cleaned$steps[is.na(by_date_cleaned$steps)] <- mean(na.omit(by_date_cleaned$steps))

Histogram of the total number of steps taken each day after missing
values are imputed Make a histogram of the total number of steps taken
each day and Calculate and report the mean and median total number of
steps taken per day. Do these values differ from the estimates from the
first part of the assignment? What is the impact of imputing missing
data on the estimates of the total daily number of steps?

    by_date_sum <- aggregate(steps ~ date, rm.na = TRUE, data = by_date, FUN = sum)
    by_date_sum2 <- aggregate(steps ~ date, rm.na = TRUE, data = by_date_cleaned, FUN = sum)

    with(by_date_sum, {
    par(mfrow = c(1, 2))
    plot(by_date_sum, type = "h", lwd = 5,lend = "square", main = "With NAs")
    abline(h = seq(0, 20000, 2500), lty = "dashed")
    plot(by_date_sum2, type = "h", lwd = 5, lend = "square", main = "NAs replaced")
    abline(h = seq(0, 20000, 2500), lty = "dashed")
    })

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

    #png("plot3.png", width=480, height=480)
    #with(by_date_sum, {
    #par(mfrow = c(1, 2))
    #plot(by_date_sum, type = "h", lwd = 5,lend = "square", main = "With NAs")
    #abline(h = seq(0, 20000, 2500), lty = "dashed")
    #plot(by_date_sum2, type = "h", lwd = 5, lend = "square", main = "NAs replaced")
    #abline(h = seq(0, 20000, 2500), lty = "dashed")
    #})
    #dev.off()

    mean_by_date <- aggregate(steps ~ date, data = by_date, FUN = mean)
    mean_by_date_cleaned <- aggregate(steps ~ date, data = by_date_cleaned, FUN = mean)
    median_by_date <- aggregate(steps ~ date, data = by_date, FUN = median)
    median_by_date_cleaned <- aggregate(steps ~ date, data = by_date_cleaned, FUN = median)

Are there differences in activity patterns between weekdays and
weekends?

    by_date_cleaned$date <- as.POSIXct(by_date_cleaned$date)
    by_date_cleaned$daytype <-  ifelse(weekdays(by_date_cleaned$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")
    by_date_cleaned$daytype <- as.factor(by_date_cleaned$daytype)

Plotting the timeseries plots

    q <- by_date_cleaned %>% group_by(daytype, interval) %>% summarize(daily_steps_tot = sum(steps))
    library(lattice)
    with(q, {
          xyplot(daily_steps_tot ~ interval | daytype, 
          type = "l",      
          main = "Total Steps Interval-wise",
          xlab = "Daily Intervals",
          ylab = "Total Steps")
    })

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

    #png("plot4.png", width=480, height=480)
    #with(q, {
    #      xyplot(daily_steps_tot ~ interval | daytype, 
    #      type = "l",      
    #      main = "Total Steps Interval-wise",
    #      xlab = "Daily Intervals",
    #      ylab = "Total Steps")
    #})
    #dev.off()
