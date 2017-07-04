---
title: Geospatial animations in R with ggplot2 and the animation package
published: true
---

One way to make exportable geospatial animations in R is by using [ggplot2](http://ggplot2.org/) to create static images, and the [animation](https://cran.r-project.org/web/packages/animation/index.html) package to export the stitched images as an animation. Here, I also use [ggmap](https://cran.r-project.org/web/packages/ggmap/index.html) to plot the data on a Google Maps background.  

Here, I'm going to animate point data from ...

The finished animation looks like this:

First, load the packages and grab the data:

```
library(ggplot2)
library(ggmap)
library(animation)

# read the data: Earthquakes in central Italy in August and September 2016 
# data were obtained from https://www.kaggle.com/blackecho/italy-earthquakes
df=read.csv(paste(datadir,'earthquakes.csv',sep=''))
```