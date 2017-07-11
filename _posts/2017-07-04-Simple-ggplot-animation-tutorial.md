---
title: Spatial animations in R with ggplot2 and the animation package
published: true
---

It's relatively easy to make simple, exportable spatial animations in R using [ggplot2](http://ggplot2.org/) to create static images, and the [animation](https://cran.r-project.org/web/packages/animation/index.html) package to export the stitched images as an animation. Here, I also use [ggmap](https://cran.r-project.org/web/packages/ggmap/index.html) to plot the data on a Google Maps background.  

In this example I'm going to animate crime data from [data.police.uk](https://data.police.uk/), consisting of date, locations and categories of crimes commited in the Liverpool area during December 2016.

The finished animation looks like this:

First, load the packages and data:

```
library(ggplot2)
library(ggmap)
library(animation)

# Read the data: Crimes in the Liverpool area during December 2016 
# (Contains public sector information licensed under the Open Government Licence v3.0)
# Data for Merseyside Police, December 2016 obtained from https://data.police.uk/
# Grab my edit for Liverpool area and with extraneous columns removed:
df=read.csv("")
```