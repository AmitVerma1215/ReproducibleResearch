---
output: html_document
---


Reproducible Research - Course Project 1
============================================

##Reading the Dataset

```{r}

library(knitr)
library(ggplot2)
library(dplyr)
library(lattice)
setwd("E:\\Old_Doc\\sas doc\\Stat\\JHU\\Reproducible Research\\Week2\\Factivity")
activity <- read.csv("activity.csv")
summary(activity)
str(activity)
activity$date <- as.Date(activity$date, "%Y-%m-%d")
str(activity)

## Total number of steps taken per day

```{r, echo=TRUE}

colMeans(is.na(activity))
steps <- aggregate(steps ~ date, activity, sum, na.rm = TRUE)
steps

```
## Histogram of the total number of steps taken each day

```{r, echo=TRUE}

act <- aggregate(steps ~ date, activity, sum, na.rm = TRUE)
hist(act$steps, main = "Numberof Steps Taken each Day", xlab = "Steps", border = "blue", col = "green", breaks = 25)


```

##Mean and median number of steps taken each day

```{r, echo=TRUE}

Stepsmean <- mean(act$steps, na.rm = TRUE)
Stepsmean

Stepsmedian <- median(act$steps, na.rm = TRUE)
Stepsmedian

```

##Time series plot of the average number of steps taken

```{r, echo=TRUE}

NotNA <- activity[!is.na(activity$steps),]
Meantable <- aggregate(steps ~ interval, NotNA, mean)
ggplot(Meantable, aes(x = interval, y = steps), xlab = "Interval", ylab = "Average Steps") + 
  geom_line() + ggtitle("Average number of  Steps taken")

```

##The 5-minute interval that, on average, contains the maximum number of step

```{r, echo=TRUE}

Maxsteps <- NotNA[which.max(NotNA$steps),]
Maxsteps

```

##Code to describe and show a strategy for imputing missing data

```{r, echo=TRUE}

###  Total number of missing values in the datase
sum(is.na(activity$steps))

### New dataset that is equal to the original dataset for the missing data filled in
activity2 <- activity
Missdata <- is.na(activity2$steps)

####  Filling in all of the missing values in the dataset.
avginterval <- tapply(activity2$steps, activity2$interval, mean, na.rm = TRUE, simplify = TRUE)
activity2$steps[Missdata] <- avginterval[as.character(activity2$interval[Missdata])]

sum(is.na(activity2$steps))

### Histogram of the total number of steps taken each day before and after missing values are imputed
par(mfrow = c(1,2))

stepwithNA <- aggregate(steps ~ date, activity, sum)

ggplot(stepwithNA, aes(x = steps), xlab = "Date", ylab = "TotalSteps") + geom_histogram(fill = "red") +
  ggtitle("Total Number of Steps taken on each Day, with NA")


stepseachday <- aggregate(steps ~ date, activity2, sum)
stepseachday

ggplot(stepseachday, aes(x = steps), xlab = "Date", ylab = "TotalSteps") + geom_histogram(fill = "blue") +
       ggtitle("Total Number of Steps taken on each Day, No NA") 

```

```{r, echo=TRUE}

## New factor variable in the dataset with two levels - "weekday" and "weekend" 

activity2 <- mutate(activity2, weektype = ifelse(weekdays(activity2$date) == "Saturday" | weekdays(activity2$date) == "Sunday", "weekend", "weekday"))
activity2$weektype <- as.factor(activity2$weektype)
head(activity2)
tail(activity2)

```


##Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

```{r, echo=TRUE}

par(mfrow = c(2, 1))

intervaldata <- activity2 %>% group_by(interval, weektype) %>% summarise(steps = mean(steps))

ggplot(intervaldata, aes(x = interval, y = steps, color = weektype)) + 
  geom_line() + labs(title = "Average number of steps taken across Weekdays and Weekends", 
x = "interval", y = "Steps") + 
  facet_wrap(~weektype, ncol = 1, nrow = 2)

```



