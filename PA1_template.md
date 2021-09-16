---
title: "Reproducible Research: Peer Assessment 1"
author: "Diyana Nanova"
date: "9/15/2021"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## 1. Loading and preprocessing the data

#chron
#lubridate
#dplyr
#tidyverse
#lattice(for grafics)


### read data
```{r} 

activity_MD <- read.csv("/Users/diyanananova/Desktop/coursera_cours5/cousr5_week2/activity.csv")
head(activity_MD)
```

```{r, message=FALSE}


library(chron)
library(lubridate)
library(dplyr)
library(tidyverse)
library(lattice)
#### 1.1 Total number of steps taken each day

AMD_total <-  activity_MD %>%
  group_by(date)%>%
  summarise(Steps = as.numeric(sum(steps, na.rm=T)))
head(AMD_total)
```


```{r, message=FALSE}
#### 1.2 Histogram of Total number of steps taken each day
steps_total <- AMD_total$Steps
hist(steps_total,
     main="Total number of steps taken each day",
     xlab="Total number of steps",
     col="darkmagenta"
     #freq=FALSE
)
```




## 2. What is mean total number of steps taken per day?
```{r, message=FALSE}
#### 2.1 mean of the total number of steps taken per day

AMD_total_mean <-  activity_MD %>%
  group_by(date)%>%
  summarise(AVG_Steps = as.numeric(mean(steps, na.rm=T)))
head(AMD_total_mean)
```

```{r, message=FALSE}
#### 2.2 median of the total number of steps taken per day
AMD_total_median <-  activity_MD %>%
  group_by(date)%>%
  summarise(Median_Steps = as.numeric(median(steps, na.rm=T)))
head(AMD_total_median)
```


## 3. What is the average daily activity pattern?
```{r, message=FALSE}
#### 3.1 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis) 
AMD_average <- activity_MD %>%
  mutate(Date = ymd(date), Steps = as.numeric(steps), Interval = as.numeric(interval)) %>%
  select(-date, -steps, -interval) %>%
  drop_na() 

plot(AMD_average$Date, AMD_average$Steps, ylab="average number of steps", xlab="days", type="l")
```

```{r, message=FALSE}
#### 3.2 interval with maximum number of steps
AMD_average_max <- AMD_average %>%
  filter(Steps==max(Steps))
head(AMD_average_max)
```
## 4. Imputing missing values
```{r, message=FALSE}
#### 4.1 total number of missing values

sum_NA <- sum(is.na(activity_MD$steps))
```

```{r, message=FALSE}
#### 4.2 filling in all of the missing values
    
AMD <- activity_MD %>%
  mutate(Date = ymd(date), Steps = as.numeric(steps), Interval = as.numeric(interval)) %>%
  select(-date, -steps, -interval)%>%
  group_by(Interval)
```

```{r}  
AMD_mean_by_interval <- AMD %>%
  summarise_at(vars(-Date),list(Steps.x = ~ mean(., na.rm = TRUE))) 
```

```{r}         
mut_mean <- left_join(AMD, AMD_mean_by_interval, by = "Interval")  
```

```{r, message=FALSE}
#### 4.3. create a new data set with the missing values
fill_na <- mut_mean %>%
  mutate(steps_replace  = ifelse(is.na(Steps),Steps.x, Steps))%>%
  select(-Steps, -Steps.x)
```

```{r, message=FALSE}
#### 4.4.  Make a histogram of the total number of steps taken each day and 
#Calculate and report the mean and median total number of steps taken per day
    
AMD_total_new <- fill_na %>%
  group_by(Date) %>%
  summarize(sum = as.numeric(sum(steps_replace)))

hist(AMD_total_new$sum,
     main="Total number of steps taken each day",
     xlab="Total number of steps",
     col="blue"
     #freq=FALSE
)
```    



- In the range of 0 to 5000 steps per day there is a change if the NA values are removed and these are replaced with the average for the respective range


## 5. Are there differences in activity patterns between weekdays and weekends?
```{r, message=FALSE}
#### 5.1. new factor variable in the dataset with two levels – “weekday” and “weekend”

fill_na$weekend = chron::is.weekend(fill_na$Date)

steps_avarege_weekday_weekend <- fill_na %>%
  group_by(Interval,weekend) %>%
  summarise(avrg_steps = mean(steps_replace)) 
  
```  

  


```{r, message=FALSE}
#### 5.2 Make a panel plot containing a time series plot (i.e.  type = "l" of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 
xyplot(steps_replace ~ Interval|weekend , fill_na, type = "l", layout = c(1, 2),
       main = "The average number of steps/weekday days or weekend days/", ylab = "Steps", xlab = "Interval")
```  