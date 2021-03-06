---
output:
  pdf_document: default
  html_document: default
---
#Reproducible Research: Peer Assessment 1
##Loading and preprocessing the data

Show any code that is needed to

1.Load data

2.Process/transform the data (if necessary) into a format suitable for your analysis

start by loading dplyr package
```{r load dplyr package, echo=FALSE, results='hide'}
library(dplyr)
```


```{r}
activity <- read.csv("activity.csv")
activity <- tibble::as_tibble(activity)
activityCompleteObs <- activity[which(complete.cases(activity)),]
```

A view of activity

```{r}
activity
```

view activityCompleteObs

```{r}
activityCompleteObs
```

##What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

3. Calculate and report the mean and median of the total number of steps taken per day

```{r}
# Calculate the total number of steps taken per day
steps_per_day <- aggregate(steps ~ date, activityCompleteObs, sum)

# Create a histogram of no of steps per day
hist(steps_per_day$steps, main = "Histogram of total number of steps per day", xlab = "Steps per day")
abline(v=mean(steps_per_day$steps), col='blue')
abline(v=median(steps_per_day$steps), col='red')

```
```{r}
print(paste("The mean and median of the total number of steps taken per day are:",
      mean(steps_per_day$steps, na.rm = TRUE), "and", median(steps_per_day$steps, na.rm = TRUE)))
```

##What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}
# Calculate average steps per interval for all days 
avg_steps_per_interval <- aggregate(steps ~ interval, activityCompleteObs, mean)
avg_steps_per_interval <- tibble::as_tibble(avg_steps_per_interval)
avg_steps_per_interval
```


Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r}
# Plot the time series with appropriate labels and heading
plot(avg_steps_per_interval$interval, avg_steps_per_interval$steps, type='l', col="blue", main="Average number of steps by Interval", xlab="Time Intervals", ylab="Average number of steps")

```


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}
which.max(avg_steps_per_interval$steps)
```

```{r}
avg_steps_per_interval[104,]
```

*So the 5-minute interval, on average across all the days in the dataset, containing the maximum number of steps is 835 with an average of 206 steps*

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```{r}
table(complete.cases(activity))
```

*So there is 2304 rows with missing values in our dataset*


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

We will replace the missing NA values with the average steps in that interval across all the days

To nail it we will add a variable Indicator in our dataset. This variable will indicate us whether one row contain a missing values


```{r}
Ind <- function(t){
  x <- dim(length(t))
  x[which(!is.na(t))] <- 1
  x[which(is.na(t))] <- 0
  return(x)
}
```

```{r}
activity$indicator <- Ind(activity$steps)
activity
```


Let's now write a code to impute missing values by

- Loop through all the rows of activity, find the one with NA for steps.

- For each identify the interval for that row

- Then identify the avg steps for that interval in avg_steps_per_interval

- Substitute the NA value with that value


```{r}
for (i in 1:nrow(activity)){
  if (activity$indicator[i]==0){
    value_to_impute <- avg_steps_per_interval$steps[which(avg_steps_per_interval$interval == activity$interval[i])]
    activity$steps[i] <- value_to_impute
  }
}
```


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

let's take a view to our filled dataset

```{r}
activity
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```{r}
# Aggregate the steps per day with the imputed values
steps_per_day_impute <- aggregate(steps ~ date, activity, sum)

# Draw a histogram of the value 
hist(steps_per_day_impute$steps, main = "Histogram of total number of steps per day (IMPUTED)", xlab = "Steps per day")
abline(v=mean(steps_per_day$steps), col='blue')

```


```{r}
# Compute the mean and median of the imputed value
# Calculate the mean and median of the total number of steps taken per day
round(mean(steps_per_day_impute$steps))
```

```{r}
median(steps_per_day_impute$steps)
```

After imputing missing values, mean and median don't really change!

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```{r}
activity$day <- weekdays(as.Date(as.character(activity$date),'%Y-%m-%d'))
```



```{r}
table(activity$day)
```


```{r}
activity$weekdays <- as.factor(ifelse(weekdays(as.Date(activity$date)) %in% c("samedi", "dimanche"), "weekend", "weekday"))
```


```{r}
table(activity$weekdays)
```




We load the ggplot2 package

```{r load ggplot2}
library(ggplot2)
```

```{r}
# Create the aggregated data frame by intervals and day_type
steps_per_day_type <- aggregate(steps ~ interval+weekdays, activity, mean)
```


```{r}
g <- ggplot(steps_per_day_type, aes(interval, steps))
g+geom_line(aes(colour = weekdays)) +
    facet_grid(weekdays ~ .) +
    labs(x="Interval", y=expression("Number of steps")) +
    ggtitle("Average number of steps taken, averaged across all weekday days or weekend days")
```

We do see some subtle differences between the average number of steps between weekdays and weekends. For instance, it appears that the user started a bit later on weekend mornings