#Introduction

This document presents the findings of **Peer Assessment 1** of the course **[Reproducible Research](https://www.coursera.org/learn/reproducible-research/home/welcome)** on **[Coursera](htttps://www.coursera.org)**. 
This assignment makes use of data from a personal activity monitoring device (eg. fitness band). This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.
**Note**: To find out more about the assignment instructions visit [link](https://github.com/smy73/Reproducible-Research-Peer-Assesment-1/blob/master/README.md)

This document presents the results of the Reproducible Research's Peer Assessment 1 in a report using a single R markdown document that can be processed by knitr and be transformed into an HTML file.

## Loading and Pre-processing the Data

1. Loading Knitr for report processing

We set echo equal to **TRUE** and results equal to **'hold'** as global options for this document.
```{r, set_options}
library(knitr)
opts_chunk$set(echo = TRUE, results = 'hold')
```
2. Loading required libraries
```{r call_libraries}
library(data.table)
library(ggplot2) 
```
3. Reading data from activity.csv
```{r read_data}
rdata <- read.csv('activity.csv', header = TRUE, sep = ",",
                  colClasses=c("numeric", "character", "numeric"))
```
###Tidying the dataset
'**date**field is converted to 'Date' class
'**interval**' field is converted to 'factor' class
```{r tidy_data}
rdata$date <- as.Date(rdata$date, format = "%Y-%m-%d")
rdata$interval <- as.factor(rdata$interval)
```

4. Rechecking the data using 'str()'
```{r check_data}
str(rdata)
```
##What is mean total of the number of steps taken per day?
Our dataset has certain missing values.We ignore the missing values in the dataset and calculate total steps per day as follows:
```{r pre_calc_stepsperday}
steps_per_day<-aggregate(steps~ date, rdata, sum)
colnames(steps_per_day) <- c("date","steps")
head(steps_per_day)
```
**1. Plotting histogram of Total number of steps taken per day**
```{rhisto}
ggplot(steps_per_day, aes(x= steps))+
       geom_histogram(fill= "blue", binwidth = 1000) +
        labs(title="Histogram of Steps Taken Per Day",
             x= "Number of Steps per Day", y= "Number of times in a day (count)")+
  theme_classic()
```
**2. Calculating mean and median of steps_per_day**
```{r meanmedian}
steps_mean <- mean(steps_per_day$steps, na.rm = TRUE)
steps_median <- median(steps_per_day$steps, na.rm= TRUE)
```

The mean is **`r format(steps_mean,digits = 8)`** and median is **`r format(steps_median,digits = 8)`**.

##What is the average daily activity pattern?

We calculate the aggregation of steps by intervals of 5-minutes and convert the intervals into integer values and save them onto a single data frame called 'steps_per_interval'

```{r steps_interval}
steps_per_interval<-aggregate(rdata$steps, 
        by=list(interval = rdata$interval),
        FUN=mean, na.rm=TRUE)
####Converting to integers

steps_per_interval$interval <- 
  as.integer(levels(steps_per_interval$interval)[steps_per_interval$interval])
colnames(steps_per_interval) <- c("interval", "steps")
```

##Time Series Plot

We now construct the time series plot of the average number of steps taken (averaged across all days) versus the 5-minute intervals:

```{r plot_time_series}
ggplot(steps_per_interval, aes(x=interval, y=steps)) +   
        geom_line(color="orange", size=1) +  
        labs(title="Average Daily Activity Pattern", x="Interval", y="Number of steps") +  
        theme_bw()
```


Now we find the 5 minute interval with the maximum number of steps


```{r max_interval}
max_interval<-steps_per_interval[which.max(steps_per_interval$steps),]
```

The **`r max_interval$interval`<sup>th</sup>** interval has a maximum **`r round(max_interval$steps)`** steps.

##Imputing the missing values
**1. Finding the number of missing values**
Before we can fill in the missing values in the dataset, we need to find out how many such values are missing.
The total number of missing values in 'steps' can be calculated using `is.na()` method to check whether the value is mising or not and then summing the logical vector.
```{r tot_na_value}

missing_vals <- sum(is.na(rdata$steps))

```

The total number of ***missing values*** are **`r missing_vals`**.

**2. Strategy for filling in all of the missing values in the dataset**
To populate missing values, we choose to replace them with the mean value at the same interval across days. In most of the cases the median is a better centrality measure than mean, but in our case the total median is not much far from total mean.

We create a function `na_fill(data, pervalue)` which the `data` arguement is the `rdata` data frame and `pervalue` arguement is the `steps_per_interval` data frame.

```{r fill_na}
na_fill <- function(data, pervalue) {
        na_index <- which(is.na(data$steps))
        na_replace <- unlist(lapply(na_index, FUN=function(idx){
                interval = data[idx,]$interval
                pervalue[pervalue$interval == interval,]$steps
        }))
        fill_steps <- data$steps
        fill_steps[na_index] <- na_replace
        fill_steps
}

rdata_fill <- data.frame(  
        steps = na_fill(rdata, steps_per_interval),  
        date = rdata$date,  
        interval = rdata$interval)
str(rdata_fill)
```

Checking if all values are filled

```{r check_empty}
sum(is.na(rdata_fill$steps))
```

Zero output shows that there are ***NO MISSING VALUES***.


**3. A histogram of the total number of steps taken each day**

Now let us plot a histogram of the daily total number of steps taken, plotted with a bin interval of 1000 steps, after filling missing values.


```{r histo_fill}
fill_steps_per_day <- aggregate(steps ~ date, rdata_fill, sum)
colnames(fill_steps_per_day) <- c("date","steps")

##plotting the histogram
ggplot(fill_steps_per_day, aes(x = steps)) + 
       geom_histogram(fill = "blue", binwidth = 1000) + 
        labs(title="Histogram of Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times in a day(Count)") + theme_bw() 

```

### Calculate and report the **mean** and **median** total number of steps taken per day.

```{r meanmedian_fill}
steps_mean_fill   <- mean(fill_steps_per_day$steps, na.rm=TRUE)
steps_median_fill <- median(fill_steps_per_day$steps, na.rm=TRUE)
```

The mean is **`r format(steps_mean_fill,digits = 8)`** and median is **`r format(steps_median_fill,digits = 8)`**.

### Do these values differ from the estimates from the first part of the assignment?

Yes, mean and median values differ slightly.

- **Before filling the data**
    1. Mean  : **`r format(steps_mean,digits = 8)`**
    2. Median: **`r format(steps_median,digits = 8)`**
    
    
- **After filling the data**
    1. Mean  : **`r format(steps_mean_fill,digits = 8)`**
    2. Median: **`r format(steps_median_fill,digits = 8)`**

We see that the values after filling the data mean and median are equal.

### What is the impact of imputing missing data on the estimates of the total daily number of steps?

As we can see, comparing with the calculations done in the first section of this document, we observe that while the mean value remains unchanged, the median value has changed and virtually matches to the mean after imputing the missing values.  

Since our data has shown a t-student distribution (refer histograms), it seems that the impact of imputing missing values has increased our peak.


## Are there differences in activity patterns between weekdays and weekends?

We do this comparison using the table with filled-in missing values.  
1. Augment the table with a column that indicates the day of the week  
2. Subset the table into two parts - weekends (Saturday and Sunday) and weekdays (Monday through Friday).  
3. Tabulate the average steps per interval for each data set.  
4. Plot the two data sets side by side for comparison.  

```{r weekdays}
weekdays_steps <- function(data) {
    weekdays_steps <- aggregate(data$steps, by=list(interval = data$interval),
                          FUN=mean, na.rm=T)
    # convert to integers for plotting
    weekdays_steps$interval <- 
            as.integer(levels(weekdays_steps$interval)[weekdays_steps$interval])
    colnames(weekdays_steps) <- c("interval", "steps")
    weekdays_steps
}

data_by_weekdays <- function(data) {
    data$weekday <- 
            as.factor(weekdays(data$date)) # weekdays
    weekend_data <- subset(data, weekday %in% c("Saturday","Sunday"))
    weekday_data <- subset(data, !weekday %in% c("Saturday","Sunday"))
    
    weekend_steps <- weekdays_steps(weekend_data)
    weekday_steps <- weekdays_steps(weekday_data)
    
    weekend_steps$dayofweek <- rep("weekend", nrow(weekend_steps))
    weekday_steps$dayofweek <- rep("weekday", nrow(weekday_steps))
    
    data_by_weekdays <- rbind(weekend_steps, weekday_steps)
    data_by_weekdays$dayofweek <- as.factor(data_by_weekdays$dayofweek)
    data_by_weekdays
}

data_weekdays <- data_by_weekdays(rdata_fill)
```

Below we can see the panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends:
```{r plot_weekdays}
ggplot(data_weekdays, aes(x=interval, y=steps)) + 
        geom_line(color="violet") + 
        facet_wrap(~ dayofweek, nrow=2, ncol=1) +
        labs(x="Interval", y="Number of steps") +
        theme_bw()

```

**Conclusions**

From the comparitive time series plots, we see that during the weekdays,the interval of time that sees peak physical activity is much higher than all the other smaller peaks.This means that during the weekdays, only at a certain time of the day physical activity is at its daily peak. The person in question seems to have some sort of fitness regimen as we find intense activity at a certain time daily.We can also see that the weekend plot has more peaks than weekdays. 
This could be due to the fact that weekends don't follow a work-centric schedule.

***

