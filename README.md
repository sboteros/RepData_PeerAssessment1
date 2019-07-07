# Data

We used the 
[Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
with records of steps walked by one anonymous subject in October and November,
2012. The data is described in the [Code book](CodeBook.md).

# Code

The file `PA1_template.Rmd` produces a report in HTML format, which contains the
following information:

1. What is mean total number of steps taken per day?
  + Makes a histogram of the total number of steps taken each day.
  + Calculates and report the **mean** and **median** total number of steps taken
  per day.
2. What is the average daily activity pattern?
  + Makes a time series plot of the 5-minute interval (x-axis) and the average
  number of steps taken, averaged across all days (y-axis).
  + Identifies which 5-minute interval, on average across all the days in the
  dataset, contains the maximum number of steps?
3. Imputing missing values
  + Calculates and reports the total number of missing values in the dataset.
  + Creates a new dataset that is equal to the original dataset but with the
  missing data filled in, imputing averages by time-span.
  + Makes a histogram of the total number of steps taken each day.
  + Calculates and report the **mean** and **median** total number of steps
  taken per day.
4. Are there differences in activity patterns between weekdays and weekends?
  + Creates a new factor variable in the dataset (`weekdays`) with two levels --
  "weekday" and "weekend" indicating whether a given date is a weekday or
  weekend day.
  + Makes a panel plot containing a time series plot (i.e. `type = "l"`) of the
  5-minute interval (x-axis) and the average number of steps taken, averaged
  across all weekday days or weekend days (y-axis).
  
# Authorship
  + **Name**: Santiago Botero S.
  + **E-mail**: sboteros@unal.edu.co
  + **Date**: 2019/07/07
  + **Encoding**: UTF-8
  