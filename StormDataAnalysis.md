# Reproducible Research - Peer Assessment 2
Kevin E. D'Elia  
09/18/2015  



# Title
Your document should have a title that briefly summarizes your data analysis

# Synopsis
describes and summarizes your analysis in at most 10 complete sentences.

# Data Processing


```r
stormData <- read.csv(bzfile("./data/repdata_data_StormData.csv.bz2"), stringsAsFactors = FALSE)
```



```r
str(stormData)
```

```
## 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : chr  "4/18/1950 0:00:00" "4/18/1950 0:00:00" "2/20/1951 0:00:00" "6/8/1951 0:00:00" ...
##  $ BGN_TIME  : chr  "0130" "0145" "1600" "0900" ...
##  $ TIME_ZONE : chr  "CST" "CST" "CST" "CST" ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: chr  "MOBILE" "BALDWIN" "FAYETTE" "MADISON" ...
##  $ STATE     : chr  "AL" "AL" "AL" "AL" ...
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : chr  "" "" "" "" ...
##  $ BGN_LOCATI: chr  "" "" "" "" ...
##  $ END_DATE  : chr  "" "" "" "" ...
##  $ END_TIME  : chr  "" "" "" "" ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : chr  "" "" "" "" ...
##  $ END_LOCATI: chr  "" "" "" "" ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: chr  "" "" "" "" ...
##  $ WFO       : chr  "" "" "" "" ...
##  $ STATEOFFIC: chr  "" "" "" "" ...
##  $ ZONENAMES : chr  "" "" "" "" ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : chr  "" "" "" "" ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```



```r
suppressMessages(library(dplyr))
library("dplyr")
```
Describe how you came up with the excluded columns

then use dplyr to select only columns of interest


```r
stormData  <- stormData  %>% select(c(EVTYPE, FATALITIES, INJURIES, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP))
```

Since we are interested in events that caused fatalities and injuries, let's filter out any that didn't cause any:
This is what you want, them map this to the NWS STUFF


```r
populationData <-   stormData  %>% 
                    select(c(EVTYPE, FATALITIES, INJURIES)) %>% 
                    filter(FATALITIES > 0.00 & INJURIES > 0.00)
```

While we are at it, let's also create a data frame with information just about property and crop damage.


```r
economicData <-   stormData  %>% 
                  select(c(EVTYPE, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP)) %>% 
                  filter(PROPDMG > 0.00 & CROPDMG > 0.00)
```

Can the datasets be reduced further?  Use summary to look at mean data for each group


```r
summary(populationData$FATALITIES)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   1.000   1.000   1.000   2.862   2.000 158.000
```

```r
summary(populationData$INJURIES)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    1.00    2.00    5.00   29.82   20.00 1700.00
```

```r
summary(economicData$PROPDMG)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.01    5.00   15.00   66.23   50.00  975.00
```

```r
summary(economicData$CROPDMG)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.01    5.00   10.00   64.76   50.00  978.00
```



```r
populationData  <- populationData  %>% filter(FATALITIES >= 2 & INJURIES >= 20)
economicData  <- economicData  %>% filter(PROPDMG >= 50 & CROPDMG >= 50)
```



Now map names from the 48 Event Types defined by the NWS to our remaining data

```r
cold  <- grepl("cold", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[cold]  <- "Cold/Wind Chill"

rain  <- grepl("rain", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[rain]  <- "Heavy Rain"

fog  <- grepl("fog", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[fog]  <- "Dense Fog"

heat  <- grepl("heat wave", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[heat]  <- "Excessive Heat"

wind  <- grepl("high wind", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[wind]  <- "High Wind"

tropical  <- grepl("tropical storm", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[tropical]  <- "Tropical Storm"

thunder  <- grepl("tstm", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[thunder]  <- "THUNDERSTORM WIND"

stream  <- grepl("urban", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[stream]  <- "Flash Flood"

fire  <- grepl("fire", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[fire]  <- "Wildfire"

hurricane  <- grepl("hurricane", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[hurricane]  <- "Hurricane (Typhoon)"

spout  <- grepl("waterspout", populationData$EVTYPE, ignore.case = T)
populationData$EVTYPE[spout]  <- "Waterspout"
```


```r
populationData$EVTYPE  <- toupper(trimws(populationData$EVTYPE))
populationData$EVTYPE  <- as.factor(populationData$EVTYPE)
```



```r
y <- populationData  %>% group_by(EVTYPE) %>% summarise(fatalities = sum(FATALITIES), injuries = sum(INJURIES))
```



```r
plot(y$fatalities, type="l", col="blue", axes=F, ann=T, xlab="Event Types", ylab="Totals", cex.lab=0.8, lwd=2, ylim=c(0, 4000))
axis(1, at=1:20, labels = F)
axis(2, las=1, cex.axis=0.8)
 text(1:20, par("usr")[3] - 2, srt=45, adj=1, labels=y$EVTYPE, xpd=T, cex=0.6)
lines(y$injuries, col="green", lwd=2)
box()
```

![](./figures/plot_population-1.png) 

You can plot fatalities/injuries
you can plot two-panel across states

x  <- storm.data %>% group_by(EVTYPE) %>% summarise(total_fatalities = sum(FATALITIES))
x  <- storm.data %>% group_by(EVTYPE) %>% summarise(total_injuries = sum(INJURIES))




# no data for this
Astronomical Low Tide

# data for this
Avalanche
Blizzard

Coastal Flood
coastal  <- grepl("coastal", stormData$EVTYPE, ignore.case = T)
stormData$EVTYPE[coastal]  <- "COASTAL FLOOD"

Cold/Wind Chill
Debris Flow
Dense Fog
Dense Smoke
Drought
Dust Devil
Dust Storm
Excessive Heat

# Combine records with extreme under this
Extreme Cold/Wind Chill
extreme  <- grepl("extreme", stormData$EVTYPE, ignore.case = T)
stormData$EVTYPE[extreme]  <- "Extreme Cold/Wind Chill"


Flash Flood
Flood
Frost/Freeze
Funnel Cloud
Freezing Fog
Hail

hail  <- grepl("hail", stormData$EVTYPE, ignore.case = T)
stormData$EVTYPE[hail]  <- "HAIL"

Heat
Heavy Rain
Heavy Snow
High Surf
High Wind

Hurricane (Typhoon)
Ice Storm
Lake-Effect Snow
Lakeshore Flood
Lightning
Marine Hail
Marine High Wind
Marine Strong Wind
Marine Thunderstorm Wind
Rip Current

Seiche
nothing to do

Sleet
Storm Surge/Tide
Strong Wind

This link will give the definition for TSTM, i.e., thunderstorm
http://forecast.weather.gov/glossary.php?word=tstm

Thunderstorm Wind
thunder  <- grepl("thu.*s.*", stormData$EVTYPE, ignore.case = T)
stormData$EVTYPE[thunder]  <- "THUNDERSTORM WIND"

Tornado
Tropical Depression
Tropical Storm
Tsunami
Volcanic Ash
Waterspout
Wildfire
Winter Storm
Winter Weather


```r
knitr::kable(head(mtcars), digits = 2, align = c(rep("l", 4), rep("c", 4), rep("r", 4)))
```

                    mpg    cyl   disp   hp     drat     wt     qsec     vs    am   gear   carb
------------------  -----  ----  -----  ----  ------  ------  -------  ----  ---  -----  -----
Mazda RX4           21.0   6     160    110    3.90    2.62    16.46    0      1      4      4
Mazda RX4 Wag       21.0   6     160    110    3.90    2.88    17.02    0      1      4      4
Datsun 710          22.8   4     108    93     3.85    2.32    18.61    1      1      4      1
Hornet 4 Drive      21.4   6     258    110    3.08    3.21    19.44    1      0      3      1
Hornet Sportabout   18.7   8     360    175    3.15    3.44    17.02    0      0      3      2
Valiant             18.1   6     225    105    2.76    3.46    20.22    1      0      3      1




describes (in words and code) how the data were loaded into R and processed for analysis. In particular, your analysis must start from the raw CSV file containing the data. You cannot do any preprocessing outside the document. If preprocessing is time-consuming you may consider using the cache = TRUE option for certain code chunks.

Does the analysis include description and justification for any data transformations? 

Your data analysis must address the following questions:

    Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?

    Across the United States, which types of events have the greatest economic consequences?

Consider writing your report as if it were to be read by a government or municipal manager who might be responsible for preparing for severe weather events and will need to prioritize resources for different types of events. However, there is no need to make any specific recommendations in your report.

# Results
in which your results are presented.

You may have other sections in your analysis, but Data Processing and Results are required.

The analysis document must have at least one figure containing a plot.

Your analyis must have no more than three figures. Figures may have multiple plots in them (i.e. panel plots), but there cannot be more than three figures total.

Do the figure(s) have descriptive captions (i.e. there is a description near the figure of what is happening in the figure)?

You must show all your code for the work in your analysis document. This may make the document a bit verbose, but that is okay. In general, you should ensure that echo = TRUE for every code chunk (this is the default setting in knitr).
