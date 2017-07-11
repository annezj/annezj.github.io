---
title: Spatial animations in R with ggplot2 and the animation package
published: false
---

A relatively easy way to make animations in R without the need to install additional tools is by using the [animation](https://cran.r-project.org/web/packages/animation/index.html) package to stitch and export sequences of images as an html/javascript animation. In this example I'm going to demonstrate how to create a spatial animation using [ggplot2](http://ggplot2.org/) and [ggmap](https://cran.r-project.org/web/packages/ggmap/index.html) to plot point data on a Google Maps background.  

This example uses crime data from [data.police.uk](https://data.police.uk/), consisting of date, locations and categories of crimes commited in the Liverpool area during 2016.

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

A quick plot shows us the data has loaded OK and the approximate number of crimes we'll expect to see per category. Also we see an interesting seasonal variation.

```
ggplot(df)+
  geom_bar(aes(x=monthname, fill=Crime.type), position="stack")+
  theme_bw()+ylab("Number of crimes")+
  xlab("")
```
![]({{site.baseurl}}/assets/images/posts/crime_bar.png)

Now on to the animation, which will have 12 frames. Define the plotting function required by the animation package. This function must create one plot per frame.

```
create.plots <-function()
{
  # Set an frame rate of 1 per second
  ani.options(interval=1, verbose=FALSE)
  # animvec is used to loop over frames
  animvec=1:nframes

  # loop over months/frames
  for(iframe in animvec)
  {
    # Pick up crimes occurring this month
    dfi=df[which(df$monthnum==iframe),]
    
    # Print the base map and title, implement various appearance tweaks
    titlestr=paste(monthstrings[iframe],'\n',2016,sep='')
    p=ggmap(mymap)+ggtitle(titlestr)+theme_bw()+
      theme(plot.title = element_text(hjust = 0.5),text = element_text(size=16))+
  	theme(axis.title.x=element_blank(),axis.text.x=element_blank(),axis.ticks.x=element_blank())+
      theme(axis.title.y=element_blank(),axis.text.y=element_blank(),axis.ticks.y=element_blank())
    
    # Plot all crimes for this frame
    p=p+geom_point(data=dfi,aes(x=Longitude,y=Latitude,color=Crime.type), alpha=0.4)
    
    # Fix the color scale across all frames so no categories are dropped
    p=p+scale_color_discrete("", drop=F)
    
    # Fix transparency, size of points and spacing on the legend
    p=p+guides(colour = guide_legend(override.aes = list(alpha = 1, size=4), keyheight = 1.5))
    print(p)  
  }
}
```

Animate by calling the function to create an html animation from the plotting function.

```
saveHTML(create.plots(),
         img.name="crime_animation",
         autoplay=T,
         outdir=getwd(),
         htmlfile=paste("crime_animation.html", sep=""),
         ani.height=800,ani.width=800,
         title="Crime Animation",
         description=c("none")
)
```

As configured here the html file will appear in the current working directory and open automatically in the default web browser. The html, javascript and css directory structure can then be copied for sharing.

The finished animation looks like [this](https://annezj.github.io/assets/animations/anim-embed.html){:target="_blank"}.

Not the prettiest media player, but good enough for a quick demo that can be run locally or shared without needing to use a public server.
