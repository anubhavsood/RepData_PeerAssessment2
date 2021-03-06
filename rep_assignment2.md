---
title: "Reproducible_PA2"
author: "Anubhav"
date: "Sunday, March 22, 2015"
output: html_document
---

Reproducible Research: Peer Assessment 2
==========================================
Created by Anubhav Sood on March 22, 2015

##Impact of Severe Weather Events on Public Health and Economy in the United States
In this report, we aim to analyze the impact of different weather events on public health and economy based on the storm database collected from the U.S. National Oceanic and Atmospheric Administration's (NOAA) from 1950 - 2011. 
We will use the estimates of fatalities, injuries, property and crop damage to decide which types of event are most harmful to the population health and economy.

### Basic settings

```r
echo=TRUE
library(ggplot2)
library(dplyr, quietly=TRUE)
library(gridExtra, quietly=TRUE)
library(R.utils, quietly=TRUE)
```

* Loading and processing the data
If repdata-data-StormData.csv file is not found, download and unzip the repdata-data-StormData.csv.bz2 folder else do nothing. 

```r
if (!"repdata-data-StormData.csv" %in% dir())
{
download.file("http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", destfile="repdata-data-StormData.csv.bz2")
bunzip2("repdata-data-StormData.csv.bz2", overwrite=T, remove=F)
}
data<-read.csv(file="repdata-data-StormData.csv",header=TRUE,stringsAsFactor=FALSE)
```

### Question 1: Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?

* Sum all data from columns FATALITIES and INJURIES on the basis of column EVTYPE. 

```r
options(scipen = 999)
data_health<-aggregate(data[,c("FATALITIES","INJURIES")],list(data$EVTYPE),sum)
names(data_health)[1]<-"EVTYPE"
```

* Extract 10 most fatality causing weather events

```r
health_1<-select(data_health,EVTYPE,FATALITIES)
health_1<-arrange(health_1,desc(FATALITIES))
health_1<-head(health_1,n=10L)
```

* Extract 10 most injury causing weather events

```r
health_2<-select(data_health,EVTYPE,INJURIES)
health_2<-arrange(health_2,desc(INJURIES))
health_2<-head(health_2,n=10L)
```

* Plot that comparison graphs between the 10 most fatal and injurious weather events and then answer Question 1

```r
fig1<-ggplot(health_1,aes(x=EVTYPE,weight=FATALITIES)) + xlab("Weather Event Type") + ylab("Number of Fatalities") + 
  geom_bar(aes(x=EVTYPE,y=FATALITIES),stat="identity",fill="steelblue") + theme(axis.text.x = element_text(angle = 45,hjust = 1)) + 
  ggtitle("Total Fatalities by Severe Weather")
fig2<-ggplot(health_2,aes(x=EVTYPE,weight=INJURIES)) + xlab("Weather Event Type") + ylab("Number of Injuries") + 
	geom_bar(aes(x=EVTYPE,y=INJURIES),stat="identity",fill="steelblue") + theme(axis.text.x = element_text(angle = 45,hjust = 1)) + 
	ggtitle("Total Injuries by Severe Weather")
grid.arrange(fig1, fig2, ncol = 2)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

### Conclusion 1:
TORNADO cause maximum number of fatalities and injuries.

### Question 2: Across the United States, which types of events have the greatest economic consequences?

* Loading and processing the data. Only load EVTYPE, PROPDMG, PROPDMGEXP, PROPDMG and PROPDMGEXP

```r
options(scipen = 999) 
data_economy<-select(data, EVTYPE, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP)
```

* Extract the data for property damage and crop damage into two different datasets.Both PROPDMGEXP and CROPDMGEXP columns record a multiplier for each observation where we have Hundred (H), Thousand (K), Million (M) and Billion (B). Please refer the Storm data document at location https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf

```r
economy_1<-data_economy[toupper(data_economy$PROPDMGEXP)=="H" | toupper(data_economy$PROPDMGEXP)=="K" | 
			 toupper(data_economy$PROPDMGEXP)=="M" |toupper(data_economy$PROPDMGEXP)=="B" |
			 toupper(data_economy$PROPDMGEXP)=="" | is.na(data_economy$PROPDMGEXP),c("EVTYPE", "PROPDMG", "PROPDMGEXP")]
economy_2<-data_economy[toupper(data_economy$CROPDMGEXP)=="H" | toupper(data_economy$CROPDMGEXP)=="K" | 
			 toupper(data_economy$CROPDMGEXP)=="M" |toupper(data_economy$CROPDMGEXP)=="B" |
			 toupper(data_economy$CROPDMGEXP)=="" | is.na(data_economy$CROPDMGEXP),c("EVTYPE", "CROPDMG", "CROPDMGEXP")]
```

* Extract the 10 most property damaging weather events in the U.S.

```r
economy_1[toupper(economy_1$PROPDMGEXP)=="H","PROPDMGEXP"]<-2
economy_1[toupper(economy_1$PROPDMGEXP)=="K","PROPDMGEXP"]<-3
economy_1[toupper(economy_1$PROPDMGEXP)=="M","PROPDMGEXP"]<-6
economy_1[toupper(economy_1$PROPDMGEXP)=="B","PROPDMGEXP"]<-9
economy_1[toupper(economy_1$PROPDMGEXP)=="" | is.na(economy_1$PROPDMGEXP),"PROPDMGEXP"]<-0
economy_1$result <- economy_1$PROPDMG * 10^(as.numeric(economy_1$PROPDMGEXP))
eco1<-aggregate(economy_1$result,list(economy_1$EVTYPE),sum)
names(eco1)<-c("EVTYPE","PROPERTY_DAMAGE")
eco1<- arrange(eco1,desc(PROPERTY_DAMAGE))
eco1<-head(eco1,n=10L)
```

* Extract the 10 most crop damaging weather events in the U.S.

```r
economy_2[toupper(economy_2$CROPDMGEXP)=="H","CROPDMGEXP"]<-2
economy_2[toupper(economy_2$CROPDMGEXP)=="K","CROPDMGEXP"]<-3
economy_2[toupper(economy_2$CROPDMGEXP)=="M","CROPDMGEXP"]<-6
economy_2[toupper(economy_2$CROPDMGEXP)=="B","CROPDMGEXP"]<-9
economy_2[toupper(economy_2$CROPDMGEXP)=="" | is.na(economy_2$CROPDMGEXP),"CROPDMGEXP"]<-0
economy_2$result <- economy_2$CROPDMG * 10^(as.numeric(economy_2$CROPDMGEXP))
eco2<-aggregate(economy_2$result,list(economy_2$EVTYPE),sum)
names(eco2)<-c("EVTYPE","CROP_DAMAGE")
eco2<- arrange(eco2,desc(CROP_DAMAGE))
eco2<-head(eco2,n=10L)
```

* Plot that comparison graphs between the 10 most property and crop damaging weather events and then answer question 2

```r
fig3<-ggplot(eco1, aes(x=EVTYPE,weight = PROPERTY_DAMAGE)) + 
	  geom_bar(aes(x=EVTYPE, y=PROPERTY_DAMAGE),stat="identity",fill="steelblue") + 
      ylab("Property Damage (US$)") +  
	  xlab("Weather Event Type") + 
      theme(axis.text.x = element_text(angle = 45, hjust = 1)) + 
      ggtitle("Total property loss by Severe Weather")
fig4<-ggplot(eco2, aes(x=EVTYPE,weight = CROP_DAMAGE)) + 
	  geom_bar(aes(x=EVTYPE, y=CROP_DAMAGE),stat="identity",fill="steelblue") + 
      ylab("Crop Damage (US$)") +  
	  xlab("Weather Event Type") + 
      theme(axis.text.x = element_text(angle = 45, hjust = 1)) + 
      ggtitle("Total Crop loss by Severe Weather")
grid.arrange(fig3, fig4, ncol = 2)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 

###Conclusion 2: 
FLOODS cause maximum property damage whereas DROUGHT cause maximum crop damage.

### Summary
Weather event causing the most number of fatalities - TORNADO<br/>
Weather event causing the most number of injuries - TORNADO<br/>
Weather event causing the most number of property damage - FLOODS<br/>
Weather event causing the most number of crop damage - DROUGHT<br/>
