---title: "Reproducible Research - Peer Assessment Project 1"author: "TLK"date: "October 2, 2017"output:  html_document: default  word_document: default---
###Read csv file```{r, echo = TRUE}activity <- read.csv("activity.csv")head(activity)```
```{r, echo = TRUE}library(ggplot2)```
```{r, echo = TRUE}library(plyr)```Processing the Data
```{r, echo = TRUE}activity$day <- weekdays(as.Date(activity$date))activity$DateTime<- as.POSIXct(activity$date, format="%Y-%m-%d")
##pulling data without nasclean <- activity[!is.na(activity$steps),]```

Calculate the total number of steps taken per day```{r, echo = TRUE}## summarizing total steps per datesumTable <- aggregate(activity$steps ~ activity$date, FUN=sum, )colnames(sumTable)<- c("Date", "Steps")```
Make a histogram of the total number of steps taken each day```{r, echo = TRUE}## Creating the histogram of total steps per dayhist(sumTable$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day")dev.copy(png,'histogram1.png')dev.off()```
Calculate and report the mean and median of the total number of steps taken per day```{r, echo = TRUE}## Mean of Stepsas.integer(mean(sumTable$Steps))```
```{r, echo = TRUE}## Median of Stepsas.integer(median(sumTable$Steps))```The average number of steps taken each day was 10766 steps.
The median number of steps taken each day was 10765 steps
Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)```{r, echo = TRUE}library(plyr)library(ggplot2)##pulling data without nasclean <- activity[!is.na(activity$steps),]
##create average number of steps per intervalintervalTable <- ddply(clean, .(interval), summarize, Avg = mean(steps))
##Create line plot of average number of steps per intervalp <- ggplot(intervalTable, aes(x=interval, y=Avg), xlab = "Interval", ylab="Average Number of Steps")p + geom_line()+xlab("Interval")+ylab("Average Number of Steps")+ggtitle("Average Number of Steps per Interval")dev.copy(png,'lineplot1.png')dev.off()```
Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r, echo = TRUE}##Maximum steps by intervalmaxSteps <- max(intervalTable$Avg)##Which interval contains the maximum average number of stepsintervalTable[intervalTable$Avg==maxSteps,1]```
The maximum number of steps for a 5-minute interval was 206 steps.
The 5-minute interval which had the maximum number of steps was the 835 interval.
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r, echo = TRUE}##Number of NAs in original data setnrow(activity[is.na(activity$steps),])```
My strategy for filling in NAs will be to substitute the missing steps with the average 5-minute interval based on the day of the week.
```{r, echo = TRUE}## Create the average number of steps per weekday and intervalavgTable <- ddply(clean, .(interval, day), summarize, Avg = mean(steps))
## Create dataset with all NAs for substitutionnadata<- activity[is.na(activity$steps),]## Merge NA data with average weekday interval for substitutionnewdata<-merge(nadata, avgTable, by=c("interval", "day"))```
Create a new dataset that is equal to the original dataset but with the missing data filled in.
```{r, echo = TRUE}## Reorder the new substituded data in the same format as clean data setnewdata2<- newdata[,c(6,4,1,2,5)]colnames(newdata2)<- c("steps", "date", "interval", "day", "DateTime")
##Merge the NA averages and non NA data togethermergeData <- rbind(clean, newdata2)```
Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 
```{r, echo = TRUE}##Create sum of steps per date to compare with step 1sumTable2 <- aggregate(mergeData$steps ~ mergeData$date, FUN=sum, )colnames(sumTable2)<- c("Date", "Steps")dev.copy(png,'histogram2.png')dev.off()```
```{r, echo = TRUE}## Median of Steps with NA data taken care ofas.integer(median(sumTable2$Steps))```
```{r, echo = TRUE}## Creating the histogram of total steps per day, categorized by data set to show impacthist(sumTable2$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with NAs Fixed", col="Black")hist(sumTable$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with NAs Fixed", col="Grey", add=T)legend("topright", c("Imputed Data", "Non-NA Data"), fill=c("black", "grey") )dev.copy(png,'histogram3.png')dev.off()```
The new mean of the imputed data is 10821 steps compared to the old mean of 10766 steps. That creates a difference of 55 steps on average per day.
The new median of the imputed data is 11015 steps compared to the old median of 10765 steps. That creates a difference of 250 steps for the median.
However, the overall shape of the distribution has not changed.
Create a new factor variable in the dataset with two levels - “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
```{r, echo = TRUE}## Create new category based on the days of the weekmergeData$DayCategory <- ifelse(mergeData$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")```
Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).
```{r, echo = TRUE}library(lattice) ```
```{r, echo = TRUE}## Summarize data by interval and type of dayintervalTable2 <- ddply(mergeData, .(interval, DayCategory), summarize, Avg = mean(steps))
##Plot data in a panel plotxyplot(Avg~interval|DayCategory, data=intervalTable2, type="l",  layout = c(1,2),       main="Average Steps per Interval Based on Type of Day",        ylab="Average Number of Steps", xlab="Interval")dev.copy(png,'panelplot1.png')dev.off()```Yes, the step activity trends are different based on whether the day occurs on a weekend or not.
