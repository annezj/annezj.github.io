---
title: Spatial animations in R with ggplot2 and the animation package
published: true
---

A relatively easy way to make animations in R without the need to install additional tools is by using the [animation](https://cran.r-project.org/web/packages/animation/index.html) package to stitch and export sequences of images as an html/javascript animation. In this example I'm going to demonstrate how to create a spatial animation using [ggplot2](http://ggplot2.org/) and [ggmap](https://cran.r-project.org/web/packages/ggmap/index.html) to plot point data on a Google Maps background.  

This example uses crime data from [data.police.uk](https://data.police.uk/), consisting of date, locations and categories of crimes commited in the Liverpool area during 2016.

The finished animation looks like [this](https://annezj.github.io/assets/animations/anim-embed.html){:target="_blank"}.

First, load the packages and data. The data is in the format of one row per crime, and I've filtered it to include only the columns of interest for this exercise.  

```
library(ggplot2)
library(ggmap)
library(animation)

# Read the data: Crimes in the Liverpool area during 2016 
# (Contains public sector information licensed under the Open Government Licence v3.0)
# Data for Merseyside Police, January to December 2016 obtained from https://data.police.uk/
# Grab my edit for Liverpool area with extraneous columns removed:
df=read.csv("https://github.com/annezj/basic_R_tutorials/raw/master/data/Liverpool-01-2016-12-2016.csv")
```

Next, get a Google Maps background for the area of interest.
```
latmin=min(df$Latitude)
latmax=max(df$Latitude)
lonmin=min(df$Longitude)
lonmax=max(df$Longitude)
mymap<-get_map(location=c(lonmin,latmin,lonmax,latmax)) 
```

The animation will be one frame per month, so process the date string to get month numbers and list month names ready for labelling.

```
df$monthnum=as.integer(substr(df$Month, 6, 7))
monthstrings=c("January", "February", "March", "April", "May", 					"June", "July", "August", "September",
               "October", "November", "December")
               df$monthname=factor(df$monthnum,levels=1:12,labels=monthstrings)
```

A quick plots shows us the data has loaded OK and the approximate number of crimes we'll expect to see per category. Also we see an interesting seasonal variation.

```
ggplot(df)+
  geom_bar(aes(x=monthname, fill=Crime.type), position="stack")+
  theme_bw()+ylab("Number of crimes")+
  xlab("")
```


