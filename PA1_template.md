---
  title: "Reproducible Research: Peer Assessment 1"
html_fragment: default
output: 
  html_document: keep_md = true
---
  
  
  
  Dear reviewer,

thank you for devoting your time to my work, focused on the data about personal movement using activity monitoring devices, collected in October and November in 2012. Measured movements were operationalized by the number of steps. The entire procedure of obtaining and analysing data is presented in the following sections of this brief report.



## Downloading, unzipping and loading the data



After opening the relevant library in order to apply codes, the file was downloaded and unzipped in the current working directory.

```{r, echo = T}
library(data.table)
fileurl = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileurl,"./repdata_data_activity.zip", mode = "wb")
unzip("repdata_data_activity.zip", exdir = getwd())
```

After preparing the file, it was read into the active R session, and in accordance to upcoming tasks, additional libraries were activated in order to proccess the data.

```{r, echo = T, message=FALSE, warning=FALSE}
rda <- read.csv("activity.csv", header = T)
library(dplyr)
library(ggplot2)
```

After finishing all of the mentioned preparations, the prepared data were used to answer the following four questions.

##Q1  What is mean total number of steps taken per day?

Firstly, a new data frame was formed from the original, with help of dplyr commands, in order to obtain the total number of steps per day. NA results were omitted from this analysis. To take a glimpse at the obtained results, the data for first 10 days were shown.

```{r, spd, echo = TRUE}
steps_per_day <- rda %>% group_by(date) %>% summarise(total_steps_per_day = sum(steps, na.rm = T))
head(steps_per_day, 10)
```

Afterwards, the new data frame was used to create a plot.

```{r, gspd, echo = TRUE}
ggplot(steps_per_day, aes(x = date, y = total_steps_per_day)) + geom_histogram(fill = "steelblue", col = "dark blue", stat = "identity") + ylab("total steps per day") + theme(axis.text.x = element_text(angle = 90, size = 8))
```

In order to calculate the mean and the median of the total steps per day, the following formula was applied:
  
  ```{r, m_mspd, echo = TRUE}
mean_median <- steps_per_day %>% summarise(mean_total_steps_per_day = mean(total_steps_per_day), median_total_steps_per_day = median(total_steps_per_day))
mean_median
```

The results suggest that the mean of the total steps per day is equal to 9354.23, while the median is equal to 10395, indicating an asymmetric distribution.



##Q2 What is the average daily activity pattern?

In order to calculate the average daily activity pattern, dplyr functions were used again, with data being grouped and summarised with respect to intervals.

```{r, int, echo = TRUE}
activity_per_interval <- rda %>% group_by(interval) %>% summarise(mean_steps_per_interval = mean(steps, na.rm = T))
head(activity_per_interval, 10)
```

Afterwards, the following code was used to find the interval with highest activity.

```{r, int_find, echo = TRUE}
activity_per_interval[which.max(activity_per_interval$mean_steps_per_interval),]
``` 

As the results suggest, the interval with highest average number of steps is the interval 835, with slightly more than 206 steps on average. This can also be noticed from the following graph.

```{r, int_graph, echo = TRUE}
ggplot(activity_per_interval, aes(x = interval, y = mean_steps_per_interval)) + geom_line(col = "dark blue") + ylab("average activity per interval")
``` 



##Q3 Imputing missing values



Firstly, the following formula was applied in order to count the missing data in the original data frame.

```{r, miss1, echo = TRUE}
colSums(is.na(rda))
``` 

In order to keep things simple, missing values were replaced with the grand mean (mean across all of the intervals and days), which was achieved by the following codes:
  
  ```{r, missrep, echo = TRUE}
new_rda <- rda
new_rda$steps[is.na(new_rda$steps)] <- mean(new_rda$steps, na.rm = T)
View(new_rda)
``` 

Afterwards, new plots were formed and mean and median were recalculated.

```{r, missnewp, echo = TRUE}
steps_per_day2 <- new_rda %>% group_by(date) %>% summarise(total_steps_per_day = sum(steps, na.rm = T))
ggplot(steps_per_day2, aes(x = date, y = total_steps_per_day)) + geom_histogram(fill = "steelblue", col = "dark blue", stat = "identity") + ylab("total steps per day") + theme(axis.text.x = element_text(angle = 90, size = 8))
mean_median2 <- steps_per_day2 %>% summarise(mean_total_steps_per_day = mean(total_steps_per_day), median_total_steps_per_day = median(total_steps_per_day))
mean_median2
```

As it can be noticed from the table above, mean and median now have the same value (10766.19), indicating that the distribution should be symmetric now.



##Q4 Are there differences in activity patterns between weekdays and weekends?

In order to manipulate with dates, firstly they were recoded in the date format.

```{r, diff1, echo = TRUE}
new_rda$date <- as.Date(new_rda$date)
``` 

Afterwards, the timezone was set as follows in order to obtain more reproducible results.

```{r, diff2, echo = TRUE}
Sys.setlocale("LC_TIME", "English")
``` 

Then, a new variable with names of weekdays for specific dates was formed and recoded (into weekXend) in order to discriminate between weekdays and weekends.

```{r, diff3, echo = TRUE}
new_rda$weekdays <- weekdays(new_rda$date)
new_rda$weekXend <- ifelse(c(new_rda$weekdays == "Saturday" | new_rda$weekdays == "Sunday"), "weekend", "weekdays")
``` 

Finally, new calculations were conducted in order to estimate if the pattern of activity per interval is different on weekdays and weekends, results of which are shown in the graph below.

```{r, diff4, echo = TRUE}
new_averages <- new_rda %>% group_by(interval, weekXend) %>% summarise(average_n_of_steps = mean(steps))
ggplot(new_averages, aes(x = interval, y = average_n_of_steps)) + geom_line() + facet_grid(weekXend~.) + ylab("average number of steps")
```

As the graph suggests, the pattern of activity is different on weekends and weekdays.



##Thank you for reviewing my project! :)