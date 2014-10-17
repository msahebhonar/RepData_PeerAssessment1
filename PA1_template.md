    ## Use suppressPackageStartupMessages to eliminate package startup messages.

Loading and preprocessing the data
----------------------------------

    data <- read.csv("~/activity.csv")
    data <- transform(data, date = as.Date(date, format = "%Y-%m-%d"))
    str(data)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

What is mean total number of steps taken per day?
-------------------------------------------------

### 2.1 Histogram

Total number of steps taken per day was calculated.

    totalNSteps <- aggregate(data$step, list(Day = data$date), sum, na.rm = T )

Figure 1 shows distribution of total number of steps taken per day for
this person.

    ggplot(totalNSteps, aes(x)) + 
      geom_histogram(breaks = seq(0, 25000, 5000), binwidth = 3000, fill = "blue", color = "light blue") +
      xlab("Total number of steps per day") +
      ggtitle("Figure 1. Distribution of total number of steps per day") 

![plot of chunk
figure(1)](./PA1_template_files/figure-markdown_strict/figure(1).png)

Based on figure 1, distribution of total number of steps per day seems
**skewed to the right**.

### 2.2 Mean & Median

    meanTotalNSteps <- mean(totalNSteps$x)
    medianTotalNSteps <- median(totalNSteps$x)

**Mean** and **median** total number of steps taken per day were
**9354.2295** and **10395**, respectively.

What is the average daily activity pattern?
-------------------------------------------

### 3.1 Trend of daily activity

In this part, mean number of steps across all days in 5-minute intervals
was calculated.

    dailyAct <- aggregate(data$steps, list(Interval = data$interval), mean, na.rm = T)

Figure 2 shows the trend of mean number of steps for 5-minute intervals
across all days.

    ggplot(dailyAct, aes(Interval, x)) +
      geom_line(color = "blue") +
      xlab("5-minute intervals") +
      ylab("mean") +
      ggtitle("Figure 2. Trend of mean number of steps for 5-minute intervals \n across 61 days") 

![plot of chunk
figure(2)](./PA1_template_files/figure-markdown_strict/figure(2).png)

### 3.2 Maximum activity time

In order to figure out the busiest time of day for this person, maximum
number of steps across intervals was calculated.

    d <- dailyAct[which.max(dailyAct$x),]

Result shows that the subject was mostly activated at interval with
identifier **835** (8 to 9 AM) during these two months.

Imputing missing values
-----------------------

### 4.1 Number of missing values

    na <- is.na(data$steps)
    nas <- sum(na)

Number of missing values is **2304**.

### 4.2 Missing values imputation

To increase accuracy of results, missing values were imputed with using
mean for that 5-minute interval.

    step <- numeric()
    for(i in 1:nrow(data)){
      if(na[i]){
        step[i] = dailyAct$x[dailyAct$Interval == data$interval[i]] 
      }else{
        step[i] = data$steps[i]
      }
    }

### 4.3 New dataset

New dataset was created with *date* and *interval* variables from
original dataset and *steps* from imputed vector.

    newData <- data.frame(steps = step, date = data$date, interval = data$interval)

### 4.4 Summary of new dataset

Histogram (figure 3) as well as mean and median of the total number of
steps taken each day were evaluated with using imputed dataset.

    #calculate total number of steps per each day with using imputed dataset
    totalNSteps <- aggregate(newData$step, list(Day = newData$date), sum)

    #Histogram of total number of steps per each day with using imputed dataset
    ggplot(totalNSteps, aes(x)) + 
      geom_histogram(breaks = seq(0, 25000, 5000), binwidth = 3000, fill = "blue", color = "light blue") + 
      xlab("Total number of steps per day") + 
      ggtitle("Figure 3. Distribution of total number of steps per day \n with using imputed dataset") 

![plot of chunk
figure(3)](./PA1_template_files/figure-markdown_strict/figure(3).png)

Comparison of figure 1 and 3 confirmed the impact of missing values on
statistical inference. Distribution of total number of steps per day
with using original dataset was nearly skewed (figure 1), whereas this
was almost bell shaped with using imputed dataset (figure 3)

    #Mean and Median of total number of steps per day with using imputed dataset
    newMeanTotalNSteps <- mean(totalNSteps$x)
    newMedianTotalNSteps <- median(totalNSteps$x)

**Mean** and **median** total number of steps taken per day were
**1.0766 × 10<sup>4</sup>** and **1.0766 × 10<sup>4</sup>**,
respectively. Comparison of new value of **mean** and **median** with
previous results (mean = **9354.2295** and median = **10395**) shows
that **median** is more robust statistics than **mean**.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

For this section, a new factor variable was created to distinguish
weekdays from weekends.

    newData$weekId <- format(newData$date, "%u")
    newData[newData$weekId %in% as.character(c(1:5)), ]$weekId = "weekday"
    newData[newData$weekId %in% as.character(c(6, 7)), ]$weekId = "weekend"
    newData$weekId <- as.factor(newData$weekId)

Activity patterns between weekdays and weekends were shown in figure 4

    #mean number of steps across all days in 5-minute intervals
    dailyAct <- aggregate(newData$steps, list(Interval = newData$interval, Week = newData$weekId), mean, na.rm = T)

    #Plot mean number of steps across all days in 5-minute intervals between weekday and weekend
    ggplot(dailyAct, aes(Interval, x)) +
      geom_line(aes(color = Week)) +
      facet_grid(Week~.) +
      xlab("5-minute intervals") +
      ylab("mean") +
      ggtitle("Figure 4. Trend of mean number of steps for 5-minute intervals \n across 61 days between weekday and weekend") +
      theme(legend.position = "none")

![plot of chunk
figure(4)](./PA1_template_files/figure-markdown_strict/figure(4).png)

As figure 4 shows the subject was more activated in weekday mornings
(goes to work) than weekend mornings (surprise!). In contrast, the
subject more active in weekend afternoons (shopping, ...) than weekday
afternoons. In conclusion, the subject's gender might be **female**.
