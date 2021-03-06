## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement -- a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Data
The data for this assignment can be downloaded from the course web site:

- Dataset: Activity monitoring data [52K]

The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


## Loading and preprocessing the data

The data is loaded and processed into a data frame "Activity".
```{r, echo=TRUE}
library(ggplot2)
library(plyr)

activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?
Basic statistical analysis is performed for the total number of steps taken each day
```{r, echo=TRUE, fig.align='center'}
steps_day <- aggregate(steps ~ date, activity, sum)
hist(steps_day$steps, main = paste("Total Steps Per Day"), col = "green", 
     xlab = "Number of steps")

total_mean <- mean(steps_day$steps)
total_median <- median(steps_day$steps)
```

## What is the average daily activity pattern?
- Avtivity pattern is analyzed by calcuating the average steps per interval and plotting histograms.
- Finding maximum average steps.

```{r, echo=TRUE, fig.align='center'}
step_interval <- aggregate(steps ~ interval, activity, mean)
plot(step_interval$interval, step_interval$steps, type= "l",
     xlab = "Interval", ylab = "Number of Steps",
     main = "Average Number of Steps per Day by Interval")

maximumsteps <- step_interval[which.max(step_interval$steps), 1]
```

## Imputing missing values
The missing data is imputed and the 'NA' Values are replaced by average steps value of that day.

```{r}
missing_data <- nrow(activity[!is.na(activity$steps), ])

Complete_data <- transform(activity, 
                           steps = ifelse(is.na(activity$steps),
                                          step_interval$steps[match(activity$interval, step_interval$interval)],
                                          activity$steps))

Complete_data[as.character(Complete_data$date) == "2012-10-01", 1] <- 0

step_com_day <- aggregate(steps ~ date, Complete_data, sum)
```

Histograms are plotted for the complete dataset and a comparison is made between Imputed and Non - Imputed datasets

```{r}
hist(step_com_day$steps, main = paste("Total Steps Each Day"), col = "red",
     xlab = "Number of steps")
hist(steps_day$steps, main = paste("Total Steps Per Day - Missing values"),
     col = "green", xlab = "Number of Steps", add = T)
legend("topright", c("Complete", "Incomplete"), col = c("red", "green"), lwd = 10)

Com_mean <- mean(step_com_day$steps)
com_median <- median(step_com_day$steps)
```

The mean and median for the complete datasets are also determined.

## Are there differences in activity patterns between weekdays and weekends?

The dates given in the datasets are distinguished into weekdays and weekends. A plot is used to compare the average steps walked during the weekdays and weekends.

```{r}
day <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")

Complete_data$Type <- as.factor(ifelse(is.element(weekdays(as.Date(Complete_data$date)), day), "Weekday", "Weekend"))

step_variation <- aggregate(steps ~ interval + Type, Complete_data, mean)

ggplot(step_variation, aes(x = interval, y = steps, color = Type)) +
    geom_line() + labs(title = "Average Daily steps by day type", x = "Interval", y = "No of steps") +
    facet_wrap(~Type, ncol = 1, nrow = 2)
```
