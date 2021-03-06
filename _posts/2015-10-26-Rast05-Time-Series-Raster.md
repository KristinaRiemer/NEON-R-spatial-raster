---
layout: post
title: "Lesson 05: Raster Time Series Data in R"
date:   2015-10-24
authors: [Jason Williams, Jeff Hollister, Kristina Riemer, Mike Smorul, Zack Brym,
Leah Wasser]
contributors: [Test Human]
packagesLibraries: [raster, rgdal, rasterVis]
dateCreated:  2014-11-26
lastModified: 2015-11-10
category: time-series-workshop
tags: [module-1]
mainTag: GIS-Spatial-Data
description: "This lesson covers how to work with a raster time series, using the
R RasterStack object. It also covers how practical assessment of data quality in
Remote Sensing derived imagery. Finally it covers pretty raster time series plotting 
using the RasterVis library."
code1: 
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Raster-Times-Series-Data-In-R/
comments: false
---

{% include _toc.html %}

##About
This lesson will explore the functions and libraries needed to work with time series
rasters in `R`. 

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">

###Goals / Objectives

After completing this activity, you will know:

* What time series raster format is.
* How to work with a set of time series rasters.
* How to plot and explore time series raster data using the `plot` function.
* Advanced plotting using `rasterVis` library and `levelplot`

###Things You'll Need To Complete This Lesson

###R Libraries to Install:

* **raster:** `install.packages("raster")`
* **rgdal:** `install.packages("rgdal")`
* **rasterVis:** `install.packages("rasterVis")`

Note: the `rasterVis` library can be used to create nicer plots of raster time
series data! <a href="https://cran.r-project.org/web/packages/rasterVis/rasterVis.pdf"
target="_blank">Learn More about the rasterVis library</a>

####Tools To Install

Please be sure you have the most current version of `R` and preferably
R studio to write your code.


####Data to Download

* <a href="http://figshare.com/articles/NEON_AOP_Hyperspectral_Teaching_Dataset_SJER_and_Harvard_forest/1580086" class="btn btn-success"> DOWNLOAD Sample NEON LiDAR data in Raster Format & Vegetation Sampling Data</a>


The LiDAR and imagery data used to create the rasters in this dataset were 
collected over the Harvard and San Joaquin field sites 
and processed at <a href="http://www.neoninc.org" target="_blank" >NEON </a> 
headquarters. The entire dataset can be accessed by request from the NEON website.  

####Recommended Pre-Lesson Reading


* <a href="http://cran.r-project.org/web/packages/raster/raster.pdf" target="_blank">
Read more about the `raster` package in R.</a>

</div>


    library(raster)
    library(rgdal)

#About the Time Series Data

In this lesson, we are working with a set of rasters, that were derived from the 
Landsat satellite - in `GeoTiff` format. Each
raster covers the <a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank">NEON Harvard Forest field site</a>.

##NDVI data
The first set of rasters, located in the `Landsat_NDVI\HARV\ndvi` is the Normalized
Difference Vegetation Index (NDVI). NDVI is a quantitative index of greeness ranging
from 0-1 where 0 is the least amount of greenness and 1 is maximum greenness following 
the index. NDVI is often used for a quantative measure of vegetation health, cover
and vegetation phenology (life cycle stage) over large areas.

Both sets of rasters are available for the same time periods throughout the year
of 2013. 

##Understanding the metadata
?? we could teach this but Leah would need to rename the files to the original
names -- thoughts?

##Goals

In this lesson, we will

1. Import NDVI data derived from the Landsat Sensor in raster (`geotiff`)format
2. Plot one full year of NDVI raster time series data. 
3. Generate an average NDVI value for each time period throughout the year.

##Getting Started 

To begin, we will create a list of raster file names that can be used to
generate a `RasterStack`. We can use `list.files` to generate the list. We will 
tell R to only find files with a `.tif` extention using the syntax `pattern=".tif$"`.

If we specify `full.names=TRUE`, the full path for each file will be added to the 
list. We will be able to create a `RasterStack` directly from the list.

Note that the full path is relative to the working directory that was set.
{: .notice }


    # Create list of NDVI file paths
    NDVI_path <- "Landsat_NDVI/HARV/2011/ndvi"
    all_NDVI <- list.files(NDVI_path, full.names = TRUE, pattern = ".tif$")
    
    #view list - note that the full path (relative to our working directory)
    #is included
    all_NDVI

    ##  [1] "Landsat_NDVI/HARV/2011/ndvi/005_HARV_ndvi_crop.tif"
    ##  [2] "Landsat_NDVI/HARV/2011/ndvi/037_HARV_ndvi_crop.tif"
    ##  [3] "Landsat_NDVI/HARV/2011/ndvi/085_HARV_ndvi_crop.tif"
    ##  [4] "Landsat_NDVI/HARV/2011/ndvi/133_HARV_ndvi_crop.tif"
    ##  [5] "Landsat_NDVI/HARV/2011/ndvi/181_HARV_ndvi_crop.tif"
    ##  [6] "Landsat_NDVI/HARV/2011/ndvi/197_HARV_ndvi_crop.tif"
    ##  [7] "Landsat_NDVI/HARV/2011/ndvi/213_HARV_ndvi_crop.tif"
    ##  [8] "Landsat_NDVI/HARV/2011/ndvi/229_HARV_ndvi_crop.tif"
    ##  [9] "Landsat_NDVI/HARV/2011/ndvi/245_HARV_ndvi_crop.tif"
    ## [10] "Landsat_NDVI/HARV/2011/ndvi/261_HARV_ndvi_crop.tif"
    ## [11] "Landsat_NDVI/HARV/2011/ndvi/277_HARV_ndvi_crop.tif"
    ## [12] "Landsat_NDVI/HARV/2011/ndvi/293_HARV_ndvi_crop.tif"
    ## [13] "Landsat_NDVI/HARV/2011/ndvi/309_HARV_ndvi_crop.tif"

Now we have a list of all `geotiff` files in the `ndvi` directory for Harvard Forest.
Once we have this list, we can create a stack and begin to work with the
data. 

We can also explore the `geotiff` tags (the embedded metadata) to learn more about
key metadata about our data including `Coordinate Reference System (CRS)`, 
`extent` and raster `resolution`.


    # Create a time series raster stack
    NDVI_stack <- stack(all_NDVI)
    
    #view crs of rasters
    crs(NDVI_stack)

    ## CRS arguments:
    ##  +proj=utm +zone=19 +ellps=WGS84 +units=m +no_defs

    #view extent of rasters in stack
    extent(NDVI_stack)

    ## class       : Extent 
    ## xmin        : 239415 
    ## xmax        : 239535 
    ## ymin        : 4714215 
    ## ymax        : 4714365

    #view the y resolution of our rasters
    yres(NDVI_stack)

    ## [1] 30

    #view the x resolution of our rasters
    xres(NDVI_stack)

    ## [1] 30

##Challenge

Before you go any further, answer the following questions about our `RasterStack`.

1. What is the `CRS`?
2. what is the `resolution` of the data? And what `units` is that resolution in?

#Plotting Time Series Data

Once we have our raster stack, we can visualization our data. Remember from a 
previous lesson, that we can use the `plot` command to quickly plot a `RasterStack`.



    #view a histogram of all of the rasters
    #nc specifies number of columns
    plot(NDVI_stack, 
         zlim = c(1500, 10000), 
         nc = 4)

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/plot-time-series-1.png) 

However, if we have the `rasterVis` package loaded, we can create a nicer plot 
using the `levelplot` function. Let's check it out.


    library(rasterVis)
    
    #create a level plot - plot
    levelplot(NDVI_stack,
              main="Landsat NDVI\nHarvard Forest")

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/levelplot-time-series-1.png) 

##Adjust the Color Ramp

Let's change the red color ramp to a green one that is more suited to our data.
We can do that using the `colorRampPalette` function in r in combination with 
`colorBrewer`. 


    #use color brewer which loads with rasterVis to generate
    #a color ramp of yellow to green
    cols <- colorRampPalette(brewer.pal(9,"YlGn"))
    #create a level plot - plot
    levelplot(NDVI_stack,
              main="Landsat NDVI better colors \nHarvard Forest",
              col.regions=cols)

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/change-color-ramp-1.png) 


Note: we can make the `levelplot` even prettier by fixing the individual tile
names. We will cover this in 
[{{ site.baseurl }}/R/Plot-Raster-Times-Series-Data-In-R/](Lesson 06 - Plot Time Series Rasters in R)

##Taking a Closer Look at Our Data

Now that we are happy with our base plot, let's take a close look at the data. 
Given when you might know about the seasons in 
Massachusettes (where Harvard Forest is located), do you notice anything that 
seems unusual about the patterns of greening and browning of the vegetation at 
the site?

It seems like things get green, but there Julian days 277 and 293 look off. 
A plot of daily temperature for 2014 is below. There are no significant peaks
or dips in the late summer that might cause the vegetation to brown and then
quickly green again the following month. 

What is happening here? Maybe we should look at the data!

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/view-temp-data-1.png) 

Looking at the temperature data, the pattern that we are seeing in NDVI seems
off. Lucky for us, we have some RGB images creates from the SAME scene, at the same
time and over the same location as our NDVI data! They are located in the `RGB` 
directory. 

#CHALLENGE

Open up the RGB images from Julian dates 277 and 293. What do you see?
{: notice }


    #create histogram
    hist(NDVI_stack, xlim = c(0, 10000))
    
    # TODO: Challenge: two of the times have weird values because of clouds, have them figure that out
    
    
    #http://oscarperpinan.github.io/rastervis/

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/view-stack-histogram-1.png) 

#View All Landsat RGB images for Harvard Forest 2011

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/view-all-rgb-1.png) 

