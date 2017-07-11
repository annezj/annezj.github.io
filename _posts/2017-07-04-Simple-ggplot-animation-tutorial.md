---
title: Spatial animations in R with ggplot2 and the animation package
published: true
---

A relatively easy way to make animations in R without the need to install additional tools is by using the [animation](https://cran.r-project.org/web/packages/animation/index.html) package to stitch and export sequences of images as an html/javascript animation. In this example I'm going to demonstrate how to create a spatial animation using [ggplot2](http://ggplot2.org/) and [ggmap](https://cran.r-project.org/web/packages/ggmap/index.html) to plot point data on a Google Maps background.  

This example uses crime data from [data.police.uk](https://data.police.uk/), consisting of date, locations and categories of crimes commited in the Liverpool area during 2016.

The finished animation looks like this:

<div markdown="0">
<script src="https://github.com/annezj/basic_R_tutorials/raw/master/output/crime_animation/js/jquery-1.4.4.min.js"></script>
<script src="https://github.com/annezj/basic_R_tutorials/raw/master/output/crime_animation/js/jquery.scianimator.min.js"></script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.3/highlight.min.js"></script>
  <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.3/languages/r.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

<link rel="stylesheet" href="https://github.com/annezj/basic_R_tutorials/raw/master/output/crime_animation/css/scianimator.css" />

<div class="scianimator"><div id="crime_animation" style="display: inline-block;"></div></div>
	<script src="https://github.com/annezj/basic_R_tutorials/raw/master/output/crime_animation/js/crime_animation.js"></script>

</div>

First, load the packages and data. The data is in the format of one row per crime, and I've filtered it to include only the columns of interest for this exercise.  

```
library(ggplot2)
library(ggmap)
library(animation)

# Read the data: Crimes in the Liverpool area during 2016 
# (Contains public sector information licensed under the Open Government Licence v3.0)
# Data for Merseyside Police, January to December 2016 obtained from https://data.police.uk/
# Grab my edit for Liverpool area and with extraneous columns removed:
df=read.csv("")
```