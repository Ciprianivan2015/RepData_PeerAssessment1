# Reproducible Research Project 1
Ciprian Ivan  
21 april 2016  

Below, you will find the *R* code for extracting, transforming and summarising *activity_data*. It was a cool, instructive project.

Code for extracting "activity_data" from the url and reading into a "csv":

```r
library("downloader");
```

```
## Warning: package 'downloader' was built under R version 3.2.5
```

```r
library("dplyr");
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
library("ggplot2");
```

```
## Warning: package 'ggplot2' was built under R version 3.2.3
```

```r
require("gridExtra");
```

```
## Loading required package: gridExtra
```

```
## Warning: package 'gridExtra' was built under R version 3.2.3
```

```r
# .............. download the zip, unzip into csv .................
setwd( dir = "D:/QF/COURSERA/Reproducible Research/Project_1/" );
getwd();
```

```
## [1] "D:/QF/COURSERA/Reproducible Research/Project_1"
```

```r
download(url = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
         , dest = "repdata_Fdata_Factivity.zip"
         , mode="wb"
         );

unzip( zipfile = "repdata_Fdata_Factivity.zip" );
activity_data = read.csv( file = "activity.csv", sep = ",", header = TRUE );
```
*What is mean total number of steps taken per day?*
  * For this part of the assignment, you can ignore the missing values in the dataset.
  * Calculate the total number of steps taken per day
  * Make a histogram of the total number of steps taken each day.
  * Calculate and report the mean and median of the total number of steps taken per day


```r
df_daily = activity_data %>% group_by( date ) %>% summarise( daily_total = sum( steps ) );
# _ _ _ _ _ _ _ _ _ _ _ _  histogram _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
ggplot( data = df_daily, aes( x = daily_total ) ) + 
        geom_histogram( na.rm = TRUE , aes(fill=..count..), col = I("darkblue")  ) +
        geom_vline( aes( xintercept = mean(daily_total, na.rm = T)),    
                color="red", linetype ="dashed", size = 2 ) +
        geom_vline( aes( xintercept = median(daily_total, na.rm = T)),color="green",   size = 1 ) + 
        geom_vline( aes( xintercept = quantile( x = daily_total, probs = c( 0.05), na.rm = T )),color="green",   size = 1 ) +
        geom_vline( aes( xintercept = quantile( x = daily_total, probs = c( 0.95), na.rm = T )),color="green",   size = 1 )  
```

```
## Warning in data.frame(xintercept = structure(2928.2, .Names = "5%"),
## PANEL = c(1L, : row names were found from a short variable and have been
## discarded
```

```
## Warning in data.frame(xintercept = structure(16204.8, .Names = "95%"),
## PANEL = c(1L, : row names were found from a short variable and have been
## discarded
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)\
 
 

```r
# _ _ _ _ _ _ _ _ _ _ _ _  mean, median _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
mean( df_daily$daily_total, na.rm = T )
```

```
## [1] 10766.19
```

```r
median( df_daily$daily_total, na.rm = T )
```

```
## [1] 10765
```


*What is the average daily activity pattern?*
  * Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) 
and the average number of steps taken, averaged across all days (y-axis)
  * Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
# _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
df_no_NA = activity_data %>% filter( !is.na( steps ) );
df_interval = activity_data %>% filter( !is.na( steps ) ) %>% group_by( interval ) %>% summarise( interval_avg = mean( steps ) );

df_max = df_interval [ df_interval$interval_avg == max( df_interval$interval_avg ), ];
# _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
plot1 = ggplot( data = df_interval, aes( x = interval, y = interval_avg )) +
        geom_line(  ) +
        geom_point(  ) +
        #scale_y_continuous(limits = c(0, 800)) +
        geom_vline( aes( xintercept = df_max$interval ), color="green", size = 1, linetype = "dashed" ) + 
        geom_hline( aes( yintercept = df_max$interval_avg ), color="green", size = 1, linetype = "dashed" ) +
        geom_text( data = NULL, x = df_max$interval * 1.5, y = df_max$interval_avg * 0.85, 
                   label = paste0( "Maximum average per interval for ( interval ==",  
                                   df_max$interval, "; value == " , df_max$interval_avg , " ) "
                                   )  
                   )
# .................. geom_boxplot ...........................................
plot2 = ggplot( data = df_no_NA, aes( x = factor( interval ), y = steps ) ) + 
        geom_boxplot(outlier.shape = NA) +
        theme(axis.title.x=element_blank(),
                axis.text.x=element_blank(),
                axis.ticks.x=element_blank()
              ) +
        stat_summary( fun.y = mean, geom = "line", aes( group = 1 ), colour = "red" )  + 
        stat_summary( fun.y = mean, geom = "point", colour = "red" )
grid.arrange( plot1, plot2, nrow = 2 ) 
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)\


*Imputing missing values*
 
 Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
 
   * Calculate and report the total number of missing values in the dataset 
 (i.e. the total number of rows with NAs)   
   * Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
   * Create a new dataset that is equal to the original dataset but with the missing data filled in.
   * Make a histogram of the total number of steps taken each day   
   * Calculate and report the mean and median total number of steps taken per day. 
   * Do these values differ from the estimates from the first part of the assignment? 
   * What is the impact of imputing missing data on the estimates of the total daily number of steps?
   

```r
df_NA = activity_data %>% filter( is.na( steps ) );
nrow( df_NA );
```

```
## [1] 2304
```

```r
df_NA_dist_dates = df_NA %>% distinct( date ) %>% select( date );
df_with_imputed_NAs = df_no_NA;
```


```r
#   - Create a new dataset that is equal to the original dataset but with the missing data filled in.
for (iter in 1:nrow(df_NA_dist_dates) ){
        rep_date = data.frame( rep( df_NA_dist_dates$date[ iter ], times = 288 ) );
        names( rep_date ) = "date";
        input_df = cbind.data.frame( df_interval$interval_avg,  rep_date$date,  df_interval$interval );
        names( input_df ) = c("steps", "date", "interval");
        df_with_imputed_NAs = rbind( df_with_imputed_NAs, input_df);
}
rm( list = c("rep_date", "input_df") );
df_with_imputed_NAs = df_with_imputed_NAs %>% arrange( date, interval )
```


```r
#   - Make a histogram of the total number of steps taken each day 
df_daily_with_imputed_NAs = df_with_imputed_NAs %>% group_by( date ) %>% summarise( daily_total = sum( steps ) );
df_daily_compare = rbind( cbind( df_daily, type = "NAs present"), cbind( df_daily_with_imputed_NAs, type = "with imputed NAs" ) );

ggplot( data = df_daily_compare, aes( x = daily_total ) ) + 
        facet_grid( type ~. ) +
        geom_histogram( na.rm = TRUE , aes(fill=..count..), col = I("darkblue")  ) 
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)\


```r
# Calculate and report the mean and median total number of steps taken per day. 
library(knitr);
```

```
## Warning: package 'knitr' was built under R version 3.2.3
```

```r
df_synthesis_est = data.frame( mean = mean( df_daily_with_imputed_NAs$daily_total ), 
                               median =median( df_daily_with_imputed_NAs$daily_total ),
                               type = "with imputed NAs"); 

df_synthesis_est = rbind( df_synthesis_est, data.frame( mean = mean( df_daily$daily_total, na.rm = TRUE), 
                                                        median =median( df_daily$daily_total, na.rm = TRUE ),
                                                        type = "NAs present") );

kable( df_synthesis_est, format = "markdown");
```



|     mean|   median|type             |
|--------:|--------:|:----------------|
| 10766.19| 10766.19|with imputed NAs |
| 10766.19| 10765.00|NAs present      |
