---
title: "Activity Analysis - Step Monitoring"
author: "GB"
date: "5/11/2020"
output: 
  html_document:
    keep_md: true
---





```r
## Code for reading in the dataset and/or processing the data
##Code for reading in the dataset from zip. If you need to know the file name: get the list of them using unzip(temp, list = T). 
##In this case there is only one file, so add [1].
temp <- tempfile()
download.file("http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
con <- unz(temp, unzip(temp, list = T)[1])
data <- read.csv(con, stringsAsFactors = F)
unlink(temp)

##Process/transform the data into a format suitable for your analysis

data$date <- strptime(data$date, format = "%Y-%m-%d")
data$date <- as.POSIXct(data$date)
```

## Histogram of the total number of steps taken each day


```r
steps_per_day <- NULL
for (i in unique(data$date)) { 
        steps_per_day<- rbind(steps_per_day, c(i, sum(na.omit(data[(data[2]==i),1]))))
}

d <- format(steps_per_day[,1])     # send the date vector to format function
d <-  strptime(d,"%Y-%m-%d")    # strptime(formatted_vector,"%d/%m/%Y)
steps_per_day[,1] <- as.Date(d)    #  pass the above result through as.Date() to remove time zone

hist(as.numeric(steps_per_day[,2]),main="", xlab="Steps per day")
##The mean () and median() of the total number of steps taken per day
abline(v=mean(as.numeric(steps_per_day[,2])), col="blue")
text((mean(as.numeric(steps_per_day[,2]))-280),25,"Mean",srt=90,col="blue")

abline(v=median(as.numeric(steps_per_day[,2])), col="green")
text((median(as.numeric(steps_per_day[,2]))-280),25,"Median",srt=90, col="green")
```

![](PA1_template_files/figure-html/Histogram of the total number of steps taken each day-1.png)<!-- -->

## Mean and median number of steps taken each day
Mean: `9354.23`. Median: `10395`.

## Time series plot of the average number of steps taken

```r
interval_average <- NULL
for (i in unique(data$interval)) { 
        interval_average<- rbind(interval_average, c(i, sum(na.omit(data[(data[3]==i),1]))/61))
}        


plot(interval_average[,1],interval_average[,2] , type="l",xlab="Interval",ylab="Average steps", )

points(interval_average[interval_average[,2]==(max(interval_average[,2])),1],max(interval_average[,2]) , pch="o", col="red")
text(interval_average[interval_average[,2]==(max(interval_average[,2])),1]-60,max(interval_average[,2]), "Max",col="red")
```

![](PA1_template_files/figure-html/Time series plot of the average number of steps taken-1.png)<!-- -->

The 5-minute interval that, on average, contains the maximum number of steps: `835`.



```r
NAs <- na.omit(data$steps) #making index of NAs

dataNONAs <- data
for (i in na.action(NAs)) { #take index of NAs, for each  element impute steps value from the average steps in the same time interval
        dataNONAs[i,1] <- interval_average[(interval_average[,1]==dataNONAs[i,3]),2]
}
```
## Histogram of the total number of steps taken each day, after missing values are imputed

The total number of missing values in the dataset: `2304`. NAs are imputed from the average steps in the same time interval.

```r
steps_per_day_NONAS <- NULL
for (i in unique(dataNONAs$date)) { 
        steps_per_day_NONAS<- rbind(steps_per_day_NONAS, c(i, sum(na.omit(dataNONAs[(dataNONAs[2]==i),1]))))
}

d <- steps_per_day_NONAS[,1]   # send the date vector to format function
d <-  strptime(d,"%Y-%m-%d")    # strptime(formatted_vector,"%d/%m/%Y)
steps_per_day_NONAS[,1] <- as.Date(d)    #  pass the above result through as.Date() to remove time zone
hist(as.numeric(steps_per_day_NONAS[,2]), main="Histogram of the total number of steps per day after NAs imputed", xlab="Steps per day")

##Mean and median number of steps taken each day
##The mean () and median() of the total number of steps taken per day
abline(v=mean(as.numeric(steps_per_day_NONAS[,2])), col="blue")
text((mean(as.numeric(steps_per_day_NONAS[,2]))-250),25,"Mean",srt=90,col="blue")

abline(v=median(as.numeric(steps_per_day_NONAS[,2])), col="green")
text((median(as.numeric(steps_per_day_NONAS[,2]))-250),25,"Median",srt=90, col="green")
```

![](PA1_template_files/figure-html/Histogram of the total number of steps taken each day after missing values are imputed-1.png)<!-- -->

Mean: `10581.01`. Median: `10395`.

Mean have changed after imputation: as NAs have been replaced with average values, mean value rose up. Median was not affected because NAs are uniformly distributed across the intervals as seen in the histogram of NAs. A slight quirk in data is probably attributable to the misinterpretation of where midnight data point belongs.

```r
hist(data[na.action(NAs),3], main="Distribution of NAs",xlab = "Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends


```r
dataNONAs$weekend <- ifelse((weekdays(as.Date(dataNONAs$date))==("Sunday") | weekdays(as.Date(dataNONAs$date))==("Saturday")),1,0)
                         
interval_average <- NULL
for (i in unique(dataNONAs$interval)) { 
        interval_average<- rbind(interval_average, c(i, sum(na.omit(dataNONAs[(dataNONAs[3]==i)&(dataNONAs[4]==0) ,1]))/(length(dataNONAs$weekend)-sum(dataNONAs$weekend))))
}        
interval_average1 <- NULL
for (i in unique(dataNONAs$interval)) { 
        interval_average1<- rbind(interval_average1, c(i, sum(na.omit(dataNONAs[(dataNONAs[3]==i)&(dataNONAs[4]==1) ,1]))/sum(dataNONAs$weekend)))
}        


##Shout out to Sean Anderson for best multipanel explainer https://seananderson.ca/courses/11-multipanel/multipanel.pdf

par(mfrow = c(2, 1))
par(cex = 0.6)
par(mar = c(0, 0, 0, 0), oma = c(4, 4, 0.5, 2))
par(tcl = -0.25)
par(mgp = c(2, 0.6, 0))

for (i in 1:2) {
        if (i %in% c(2)) {
                plot(interval_average[,1],interval_average[,2] , main=NULL, type="l", axes = FALSE)
                axis(1, col = "grey40", col.axis = "grey20")
                axis(2, col = "grey40", col.axis = "grey20",at=seq(0.1,0.8,0.1), las=2)
                axis(4, col = "grey40", col.axis = "grey20", label=F, at=seq(0.1,0.8,0.1)) 
                title("weekday", line = -1) #using negative values to place title inside plot
                }
        if (i %in% c(1)) {
                plot(interval_average1[,1],interval_average1[,2] , main=NULL, type="l", axes = FALSE)
                axis(2, col = "grey40", col.axis = "grey20", label=F, at=seq(0.1,0.5,0.1))
                axis(3, col = "grey40", col.axis = "grey20", label=F)
                axis(4, col = "grey40", col.axis = "grey20", at=seq(0.1,0.5,0.1), las=2)
                title("weekend", line = -1)
        }
        box(col = "grey60")
}
mtext("Interval", side = 1, outer = TRUE, cex = 0.7, line = 2.2, col = "grey20")
mtext("Number of steps", side = 2, outer = TRUE, cex = 0.7, line = 2.2, col = "grey20")
```

![](PA1_template_files/figure-html/Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends-1.png)<!-- -->

