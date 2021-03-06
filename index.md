---
title       : Weather Man
subtitle    : How the Weather Man works behind the scenes?
author      : Cem Demircioglu
job         : Consultant
framework   : io2012   # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides

--- .class #id 

## Executive Summary


This presentation is about the Weather Man application that was designed by Cem Demircioglu. The application predicts the temperature of a selected date. The prediction is based on 10 years of data over a specific range of days.
<br>
<br>
There are three stages: <br>
1. Data acquisition<br>
2. Date range processing<br>
3. Prediction calculation<br>
 

--- 

## Inner workings -- Data Acquisition

Pull the weather data from the internet and format it. 

```r
    #Get the data set from the URL 
    mydataframe <- read.fwf(
      file = url("http://academic.udayton.edu/kissock/http/Weather/gsod95-current/WASEATTL.txt"),
      widths=c(1,2, 7,9, 6,10, 5,9))
    
    #Create the report date by doing string concat. 
    mydataframe$ReportDate <- as.Date(
      paste(as.character(mydataframe[,2]),as.character(mydataframe[,4]),as.character(mydataframe[,6]),sep = "/")
      , "%m/%d/%Y")

  #Pick the two columns from the dataframe
  mydataframe <- mydataframe[,c(8,9)]
    
  #Set the column names
  colnames(mydataframe) <- c("Temp","ReportDate")
```

--- 

## Inner workings -- Date Processing

At this stage, we calculate the date ranges, based on user input. 

```r
    mydateend <- as.Date('2014-04-20') #At Shiny, there variables are user defined. 
    mynumberofdays <- 10
    myfinaldataframe <- data.frame(ReportDate=as.Date(character()),Temp=numeric()) 
    
    for(i in 1:10) {  
      tdate <- as.POSIXlt(mydateend)
      tdate$year <- tdate$year - i            
      tempdf <- mydataframe[ #Search for the dates 
        mydataframe[,2] > as.Date(tdate) - mynumberofdays & 
          mydataframe[,2] < as.Date(tdate)
        ,]
      
      #Add the rows to the new data frame
      myfinaldataframe <- rbind(myfinaldataframe,tempdf)
    }
```

--- 

## Inner workings -- Prediction 

At this final stage, we predict the temparature. 

```r
    #Create the linear regression model
    fit <- lm(Temp ~ ReportDate,myfinaldataframe)
    
    #Set the predicted temp
    predicted <- predict(fit, data.frame(ReportDate = c(mydateend)))
    
    format(round(predicted, 2), nsmall = 2)
```

```
##       1 
## "47.35"
```
