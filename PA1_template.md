---
title: "Reproducible Research: Peer Assessment 1"
subtitle: "Walking datasets"
author: "Santiago Botero Sierra"
date: 2019/07/06
output: 
  html_document:
    keep_md: true
bibliography: bibliography.bib
---


```r
  knitr::opts_chunk$set(message = FALSE, warning = FALSE)
```


# Introduction

This report uses data obtained from an anonymous individual whose daily activity
has been tracked in Otober and November, 2017. The dataset includes the steps
performed by this individual in 5 minutes time-spans.

We are interested in analysing the total steps performed by the individual, 
visualizing if there was any activity pattern in those months, and looking for
differences on activity between week-days and weekend-days.

# Software



We used the statistical program R version 3.5.2 (2018-12-20) [@r]. We used the following
packages loaded in `R`:

+ `knitr` [@knitr1; @knitr2; @knitr3].
+ `dplyr` [@dplyr].
+ `tidyr` [@tidyr].
+ `ggplot2` [@ggplot2].
+ `lubridate` [@lubridate].
+ `here` [@here].

# Data

We used the walking dataset included with the [Reproducible Resear Course][1],
available [here][2]. In the cleaning phase we noted that each reported date has
288 observations, which correponds exactly with the 288 five minutes
time-spans in a given day, so we transformed the interval identificator in 
minutes of the day; and we generated a variable called `time` which unified the
information available in the variables `date`and `interval`, reflecting the 
initial time of each time-span.


```r
  # Unziping the dataset
  if (!dir.exists(here(".", "data"))) {dir.create(here(".", "data"))}
  unzip(here(".", "activity.zip"), exdir = here(".", "data"))
  
  # Loading and tidying the dataset
  walking <- read.csv(here(".", "data", "activity.csv")) %>%
    tbl_df %>%
    group_by(date) %>%
    mutate(interval = row_number(interval) - 1) %>%
    ungroup() %>%
    mutate(date = ymd(date),
           time = seconds_to_period(interval * 60 * 5),
           daytime = ymd_hms(date + time)) %>%
    group_by(date)
```


# Analysis

## What is mean total number of steps taken per day?


```r
  total_steps <- walking %>%
    summarize(total = sum(steps, na.rm = TRUE))
```

In the following plot is depicted the number of days in which the individual
walked each number of steps. It's interesting to note that the mean value 
(vertical black line) is in a bin just observed in one day, and corresponds to
9354.2295082. Each day, this individual walked
between 0 and 
21194 steps.


```r
  # Create figure/ directory
  if (!dir.exists(here(".", "figure"))) {dir.create(here(".", "figure"))}

  # Export the graph
  png(here(".", "figure", "mean_steps_histo.png"))
  ggplot(total_steps, aes(total)) + 
    geom_histogram() + 
    geom_vline(aes(xintercept = mean(total, na.rm = TRUE))) +
    labs(title = "Total number of steps taken per day",
         x = "Number of steps", y = "Number of days") + 
    theme_bw()
  dev.off()
```

```
## png 
##   2
```

```r
  # Include the graph
  knitr::include_graphics(here(".", "figure", "mean_steps_histo.png"))
```

<img src="C:/Users/sbote/OneDrive/Documentos/DataScienceSpecialization/5 Reproducible research/Week2/RepData_PeerAssessment1/./figure/mean_steps_histo.png" width="480" />

## What is the average daily activity pattern?


```r
  statistics <- total_steps %>% 
    group_by(date) %>%
    summarize(mean = mean(total, na.rm = TRUE),
              median = median(total, na.rm = TRUE))
```

We take into account different summary statistics on the total daily walking
steps, shown in the next table. We noted that the *median* and the *mean* steps
were the same in each day of the months observed.



In the following graph we ploted the daily mean of total steps during the
period.


```r
  # Export graph
  png(here(".", "figure", "mean_steps_line.png"))
  ggplot(statistics, aes(x = date, y = mean)) +
    geom_line() +
    labs(title = "Mean steps taken each day",
         x = "Date", y = "Mean steps",
         caption = "Note: Mean and median steps taken each day are the same.") +
    theme_bw()
  dev.off()
```

```
## png 
##   2
```

```r
  # Include the graph
  knitr::include_graphics(here(".", "figure", "mean_steps_line.png"))
```

<img src="C:/Users/sbote/OneDrive/Documentos/DataScienceSpecialization/5 Reproducible research/Week2/RepData_PeerAssessment1/./figure/mean_steps_line.png" width="480" />

In the following graph we observe the average number of steps taken in each 
5-minute time-span by the individual. Here, we can observe that, on average, 
from 00:00 to 05:00 this individual do not use to walk (or data is missing), and
that the most intensive walking activity take place aproximately at 08:00. 


```r
  # Compute maximum steps
  max_steps <- walking %>%
    ungroup() %>%
    group_by(interval) %>% 
    summarize(mean = mean(steps, na.rm = TRUE)) %>%
    mutate(time = seconds_to_period(interval * 5 * 60))
  # Export graph
  png(here(".", "figure", "max_steps_line.png"))
  ggplot(max_steps, aes(x = time, y = mean)) + 
    geom_line() + 
    theme_bw() +
    scale_x_time(breaks = seconds_to_period(seq(1, 24, by = 4) * 60^2)) + 
    labs(title = "Average steps in a 5-minutes time-span",
         x = "Time of the day", y = "Average steps")
  dev.off()
```

```
## png 
##   2
```

```r
  # Include the graph
  knitr::include_graphics(here(".", "figure", "max_steps_line.png"))
```

<img src="C:/Users/sbote/OneDrive/Documentos/DataScienceSpecialization/5 Reproducible research/Week2/RepData_PeerAssessment1/./figure/max_steps_line.png" width="480" />

As a matter of fact, the most active 5-minute time-span, on average, is
the one shown in the followig table.


```r
  max_steps[which.max(max_steps$mean), 2:3] %>%
    knitr::kable(col.names = c("Steps", "Time"),
                 caption = "Maximum average number of steps.")
```



Table: Maximum average number of steps.

    Steps        Time
---------  ----------
 206.1698   8H 35M 0S


## Imputing missing values

In the following plot, we observed that the missing values are not related with
the time of meassurment, since the missing values represent 
13.1147541%, regardless of the time or the level of 
activity.


```r
  # Compute missings
  missing <- walking %>%
    ungroup() %>%
    group_by(interval) %>% 
    summarize(mean_steps = mean(steps, na.rm = TRUE),
              prop_missing = mean(is.na(steps)) * 100) %>%
    gather(key = "measure", value = "value", mean_steps, prop_missing) %>%
    mutate(time = seconds_to_period(interval * 5 * 60))
  # Export graph
  png(here(".", "figure", "missing_line.png"))
  ggplot(missing, aes(x = time, y = value)) + 
    geom_line() + 
    facet_grid(measure ~ ., scales = "free") +
    theme_bw() +
    scale_x_time(breaks = seconds_to_period(seq(1, 24, by = 4) * 60^2)) + 
    labs(title = "Average steps in a 5-minutes time-span and missing proportion",
         x = "Time of the day")
  dev.off()
```

```
## png 
##   2
```

```r
  # Include the graph
  knitr::include_graphics(here(".", "figure", "missing_line.png"))
```

<img src="C:/Users/sbote/OneDrive/Documentos/DataScienceSpecialization/5 Reproducible research/Week2/RepData_PeerAssessment1/./figure/missing_line.png" width="480" />

We continued analysing why the proportion of missing values were the same
across all time-spans in the period, and noted that all missing values were in
specific days, and that all the registries of these days were missing. This
information is shown in the following table.


```r
  walking %>% filter(is.na(steps)) %>% select(date) %>% table
```

```
## .
## 2012-10-01 2012-10-08 2012-11-01 2012-11-04 2012-11-09 2012-11-10 
##        288        288        288        288        288        288 
## 2012-11-14 2012-11-30 
##        288        288
```

Therefore, our imputing strategy is replacing the missing values with the
time-span average.


```r
  replace <- walking %>%
    ungroup %>%
    group_by(interval) %>%
    summarize(steps2 = mean(steps, na.rm = TRUE))
  walking2 <- merge(walking, replace) %>%
    tbl_df %>%
    mutate(steps = ifelse(is.na(steps), steps2, steps)) %>% 
    select(-steps2) %>%
    arrange(daytime)
```

Bellow, is shown a histogram with the steps after imputing missing values.


```r
  # Export graph
  png(here(".", "figure", "mean_steps_histo2.png"))
  walking2 %>%
    group_by(date) %>%
    summarize(total = sum(steps)) %>%
    ggplot(aes(total)) + 
    geom_histogram() + 
    geom_vline(aes(xintercept = mean(total, na.rm = TRUE))) +
    labs(title = "Total number of steps taken per day",
         x = "Number of steps", y = "Number of days",
         caption = "Note: Missing values imputed as time-spans averages") + 
    theme_bw()
  dev.off()
```

```
## png 
##   2
```

```r
  # Include the graph
  knitr::include_graphics(here(".", "figure", "mean_steps_histo2.png"))
```

<img src="C:/Users/sbote/OneDrive/Documentos/DataScienceSpecialization/5 Reproducible research/Week2/RepData_PeerAssessment1/./figure/mean_steps_histo2.png" width="480" />

## Are there differences in activity patterns between weekdays and weekends?

In the following plot we noted that, on average, weekday and weekend walking
intensity is different. We observed that the range of steps is lower in weekends
than in weekdays, but the weekend activity has more spikes than the weekday one.
One different approach for imputing data, would be imputing by average 
time-spans, by controlling for differences in the days of the week.


```r
  # Set locale to a common language
  Sys.setlocale(locale = "English")
```

```
## [1] "LC_COLLATE=English_United States.1252;LC_CTYPE=English_United States.1252;LC_MONETARY=English_United States.1252;LC_NUMERIC=C;LC_TIME=English_United States.1252"
```

```r
  # Identifying weekdays and weekends
  weekly <- walking2 %>%
    mutate(weekdays = weekdays(date),
           weekdays = ifelse(weekdays %in% c("Saturday", "Sunday"),
                             "Weekend", "Weekday")) %>% 
    group_by(weekdays, interval) %>%
    summarize(mean = mean(steps)) %>% 
    ungroup %>%
    mutate(time = seconds_to_period(interval * 60 * 5))
  
  # Export graph
  png(here(".", "figure", "weekdays.png"))
  ggplot(weekly, aes(x = time, y = mean)) + 
    geom_line() + 
    facet_grid(. ~ weekdays) +
    theme_bw() +
    scale_x_time(breaks = seconds_to_period(seq(1, 24, by = 8) * 60^2)) + 
    labs(title = "Average steps in a 5-minutes time-span, by weekends and others",
         x = "Time of the day")
  dev.off()
```

```
## png 
##   2
```

```r
  # Include the graph
  knitr::include_graphics(here(".", "figure", "weekdays.png"))
```

<img src="C:/Users/sbote/OneDrive/Documentos/DataScienceSpecialization/5 Reproducible research/Week2/RepData_PeerAssessment1/./figure/weekdays.png" width="480" />

## References

[1]: https://www.coursera.org/learn/reproducible-research?
[2]: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip
