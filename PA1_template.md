---
title: "PA1_template_Rmd"
output: html_document
date: "2024-08-08"
---



## R Markdown

This code loads the data.


```r
activity <- read.csv('S:/CITA/SUSANA, ABRIL/Training and Resources/Data Science Specialization/reproducible_research_project1/activity.csv')
```

This step loads the dplyr and ggplot2 libraries.

```r
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

```r
library(ggplot2)
library(knitr)
```

This step calculates the number of steps taken per day.


```r
daily_steps <- activity %>%
  group_by(date) %>%
  summarize(total_steps = sum(steps, na.rm = TRUE))
```

This step creates a histogram of the total number of steps taken each day.


```r
ggplot(daily_steps, aes(x = total_steps)) +
  geom_histogram(binwidth = 1000, fill = "blue", color = "black") +
  labs(title = "Histogram of Total Steps Per Day",
       x = "Total Steps",
       y = "Frequency") +
  theme_minimal()
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

This step calculates the mean and median of the total number of steps taken per day.


```r
mean_steps <- mean(daily_steps$total_steps, na.rm = TRUE)
median_steps <- median(daily_steps$total_steps, na.rm = TRUE)

cat("Mean of total steps per day:", mean_steps, "\n")
```

```
## Mean of total steps per day: 9354.23
```

```r
cat("Median of total steps per day:", median_steps, "\n")
```

```
## Median of total steps per day: 10395
```

This step makes a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days.


```r
average_steps_interval <- activity %>%
  group_by(interval) %>%
  summarize(mean_steps_interval = mean(steps, na.rm = TRUE))

plot(average_steps_interval$interval, 
     average_steps_interval$mean_steps_interval, 
     type = "l", 
     xlab = "5-Minute Interval", 
     ylab = "Average Number of Steps", 
     main = "Average Number of Steps per 5-Minute Interval")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

This step calculates the 5-minute interval with the maximum number of steps on average.


```r
max_interval <- average_steps_interval %>%
  filter(mean_steps_interval == max(mean_steps_interval)) %>%
  select(interval)

cat("The 5-minute interval with the maximum number of steps on average is:", max_interval$interval, "\n")
```

```
## The 5-minute interval with the maximum number of steps on average is: 835
```

This step calculates the number of missing values in the data.


```r
rows_with_na <- activity %>%
  filter(if_any(everything(), is.na)) %>%
  nrow()

cat("The total number of rows with missing values is:", rows_with_na, "\n")
```

```
## The total number of rows with missing values is: 2304
```

This step creates a new data set with the missing data filled in by using the mean steps per each 5-minute interval.


```r
activity <- activity %>%
  left_join(average_steps_interval, by = "interval")

activity$steps <- ifelse(is.na(activity$steps), 
                     activity$mean_steps_interval, 
                     activity$steps)

activity <- select(activity, -mean_steps_interval)
```

This step creates a histogram of steps taken each day and calculates the mean and median total number of steps taken per day for the new data set without missing values.


```r
activity$date <- as.Date(activity$date)

daily_steps <- activity %>%
  group_by(date) %>%
  summarize(total_steps = sum(steps, na.rm = TRUE))

ggplot(daily_steps, aes(x = total_steps)) +
  geom_histogram(binwidth = 1000, fill = "blue", color = "black") +
  labs(title = "Histogram of Total Steps Per Day",
       x = "Total Steps",
       y = "Frequency") +
  theme_minimal()
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

```r
mean_steps <- mean(daily_steps$total_steps, na.rm = TRUE)
median_steps <- median(daily_steps$total_steps, na.rm = TRUE)


cat("Mean of total steps per day after imputation:", mean_steps, "\n")
```

```
## Mean of total steps per day after imputation: 10766.19
```

```r
cat("Median of total steps per day after imputation:", median_steps, "\n")
```

```
## Median of total steps per day after imputation: 10766.19
```

The mean and median after imputation differ from their previous values. The imputation increases the total count of steps, which leads to higher mean values.

This step creates a new factor variable with two levels: weekday and weekend. 


```r
activity <- activity %>%
  mutate(day_type = ifelse(weekdays(date) %in% c("Saturday", "Sunday"), 
                           "weekend", 
                           "weekday")) %>%
  mutate(day_type = factor(day_type, levels = c("weekday", "weekend")))
```


This step creates a panel plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.


```r
average_steps <- activity %>%
  group_by(interval, day_type) %>%
  summarize(mean_steps = mean(steps, na.rm = TRUE))
```

```
## `summarise()` has grouped output by 'interval'. You can override using the `.groups` argument.
```

```r
ggplot(average_steps, aes(x = interval, y = mean_steps)) +
  geom_line() +
  facet_wrap(~ day_type, ncol = 1, labeller = labeller(day_type = c(weekday = "Weekday", weekend = "Weekend"))) +
  labs(title = "Average Number of Steps per 5-Minute Interval",
       x = "5-Minute Interval",
       y = "Average Number of Steps") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)

```r
knit2html()
```

```
## Error in read_utf8(input): argument "input" is missing, with no default
```

