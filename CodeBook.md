# `walking` database

We used the walking dataset included with the [Reproducible Resear Course][1],
available [here][2]. We loaded it in a database called `walking`, modified the
variable `interval`, which is now a sequence over each day; so, the database has
the following variables:

+ *steps* (integer): Number of steps taking in a 5-minute interval (missing
values are coded as `NA`).
+ *date* (date): The date on which the measurement was taken in YYYY-MM-DD
format.
+ *interval* (integer): Identifier for the 5-minute interval in which
measurement was taken, by each day.
+ *time* (Period): Identifies the hour and minute of the day when the reported
interval began.
+ *daytime* (POSIXct): Identifies the day, hour and minute when the reported
interval began.

# `walking2` database

This database has the same variables as `walking`, but the missing values of 
the variable `steps` has been imputed with the averages of each `interval`
across the period.

