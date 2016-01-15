---
layout: post
title: "Lesson 04: Crop Raster Data and Extract Summary Pixels Values From 
Rasters in R"
date:   2015-10-23
authors: [Joseph Stachelek, Leah A. Wasser, Megan A. Jones]
contributors: [Sarah Newman]
dateCreated:  2015-10-23
lastModified: 2016-01-15
packagesLibraries: [rgdal, raster, ggplot2]
category: 
mainTag: vector-data-workshop
tags: [vector-data, vector-data-workshop, R, spatial-data-gis]
workshopSeries: [vector-data-series]
description: "This tutorial covers how to modify (crop) a raster extent using
the extent of a vector shapefile. It also covers extracting pixel values from 
defined locations stored in a spatial object."
code1: 04-vector-raster-integration-advanced.R
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/crop-extract-raster-data-R/
comments: false
---

{% include _toc.html %}

##About
This lesson explains how to crop a raster using the extent of a vector
shapefile. We will also cover how to extract values from a raster that occur
within a set of polygons, or in a buffer (surrounding) region around a set of
points.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">
#Goals / Objectives
After completing this activity, you will:

 * Be able to crop a raster to the extent of a vector layer.
 * Be able to extract values from raster that correspond to a vector file
 overlay.
 
##Things You’ll Need To Complete This Lesson
To complete this lesson: you will need the most current version of R, and 
preferably RStudio, loaded on your computer.

###Install R Packages

* **raster:** `install.packages("raster")`
* **rgdal:** `install.packages("rgdal")`
* **sp:** `install.packages("sp")`
* **ggplot2:** `install.packages("ggplot2")`

* [More on Packages in R - Adapted from Software Carpentry.]({{site.baseurl}}R/Packages-In-R/)


###Download Data
{% include/dataSubsets/_data_Site-Layout-Files.html %}
{% include/dataSubsets/_data_Airborne-Remote-Sensing.html %}

****

{% include/_greyBox-wd-rscript.html %}

**Vector Lesson Series:** This lesson is part of a lesson series on 
[vector data in R ]({{ site.baseurl }}self-paced-tutorials/spatial-vector-series). It is also
part of a larger 
[spatio-temporal Data Carpentry Workshop ]({{ site.baseurl }}self-paced-tutorials/spatio-temporal-workshop)
that includes working with
[raster data in R ]({{ site.baseurl }}self-paced-tutorials/spatial-raster-series) 
and  
[tabular time series in R ]({{ site.baseurl }}self-paced-tutorials/tabular-time-series).
</div> 

##Load Packages

We will use the `rgdal` and `raster` libraries for this lesson. 


 
##Crop a Raster to Vector Extent
We often work with spatial layers that have different spatial extents.

<figure>
    <a href="{{ site.baseurl }}/images/spatialVector/spatial_extent.png">
    <img src="{{ site.baseurl }}/images/spatialVector/spatial_extent.png"></a>
    <figcaption>The spatial extent of a shapefile or R spatial object represents
    the geographic "edge" or location that is the furthest north, south east and 
    west. Thus is represents the overall geographic coverage of the spatial object. 
    Image Source: National Ecological Observatory Network (NEON) 
    </figcaption>
</figure>

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/view-extents-1.png) 



The graphic above illustrates the extent of several of the spatial layers that 
we have worked with in this vector data lesson series including:

* Area of interest (AOI).
* A roads/trails.
* And plot locations.
* A canopy height model (CHM) in GeoTIFF format.

However, sometimes we have a raster file that is must larger than our study 
area or area of interest. It is often more efficient to `crop` the raster to the 
extent of our study area, to keep file sizes smaller as we process our data. 


## Import Data
We will use four vector shapefiles with data on (field site boundary,
roads/trails, tower location, and study plot locations) and one raster GeoTIFF
file, a Canopy Height Model for the Harvard Forest, Massachusetts. 

If you completed
[Vector Data in R: Open & Plot Shapefiles]({{site.baseurl}}/R/open-shapefile-in-R/)
and [.csv to Shapefile in R]({{site.baseurl}}/R/csv-to-shapefile-in-R/)
you may already have these `R` spatial objects. 


    #set working directory to data folder
    #setwd("pathToDirHere")
    
    #Imported in L00: Vector Data in R - Open & Plot Data
    # shapefile 
    aoiBoundary_HARV <- readOGR("NEON-DS-Site-Layout-Files/HARV/",
                                "HarClip_UTMZ18")

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "NEON-DS-Site-Layout-Files/HARV/", layer: "HarClip_UTMZ18"
    ## with 1 features
    ## It has 1 fields

    #Import a line shapefile
    lines_HARV <- readOGR( "NEON-DS-Site-Layout-Files/HARV/",
                           "HARV_roads")

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "NEON-DS-Site-Layout-Files/HARV/", layer: "HARV_roads"
    ## with 13 features
    ## It has 15 fields

    #Import a point shapefile 
    point_HARV <- readOGR("NEON-DS-Site-Layout-Files/HARV/",
                          "HARVtower_UTM18N")

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "NEON-DS-Site-Layout-Files/HARV/", layer: "HARVtower_UTM18N"
    ## with 1 features
    ## It has 14 fields

    #Imported in L02: .csv to Shapefile in R
    #import raster Canopy Height Model (CHM)
    chm_HARV <- 
      raster("NEON_RemoteSensing/HARV/CHM/HARV_chmCrop.tif")

#Crop a Raster Using Vector Extent
We can use the `crop` function to crop a raster to the extent of another spatial 
object. To do this, we need to specify the raster to be cropped and the spatial
object that will be used to crop the raster. `R` will use the `extent` of the spatial
object as the cropping boundary.


    #plot full CHM
    plot(chm_HARV,
         main="LiDAR CHM - Not Cropped\nNEON Harvard Forest Field Site")

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/Crop-by-vector-extent-1.png) 

    #crop the chm
    chm_HARV_Crop <- crop(x = chm_HARV, y = aoiBoundary_HARV)
    
    #plot full CHM
    plot(extent(chm_HARV),
         lwd=4,col="purple",
         main="LiDAR CHM - Cropped\nNEON Harvard Forest Field Site",
         xlab="easting", ylab="northing")
    plot(chm_HARV_Crop,
         add=TRUE)

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/Crop-by-vector-extent-2.png) 

We can see from the plot above that the full CHM extent (plotted in purple) is
much larger than the resulting cropped raster. Our new cropped CHM now has the 
same extent as the `aoiBoundary_HARV` object that was used as a crop extent.


    #view the data in a plot
    plot(aoiBoundary_HARV, lwd=8, border="blue",
         main = "Cropped LiDAR CHM \n NEON Harvard Forest Field Site")
    
    plot(chm_HARV_Crop, add = TRUE)

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/view-crop-extent-1.png) 

    #lets look at the extent of all of our objects
    extent(chm_HARV)

    ## class       : Extent 
    ## xmin        : 731453 
    ## xmax        : 733150 
    ## ymin        : 4712471 
    ## ymax        : 4713838

    extent(chm_HARV_Crop)

    ## class       : Extent 
    ## xmin        : 732128 
    ## xmax        : 732251 
    ## ymin        : 4713209 
    ## ymax        : 4713359

    extent(aoiBoundary_HARV)

    ## class       : Extent 
    ## xmin        : 732128 
    ## xmax        : 732251.1 
    ## ymin        : 4713209 
    ## ymax        : 4713359

<div id="challenge" markdown="1">
##Challenge: Crop to Vector Points Extent
Crop the Canopy Height Model to the extent of the study plot locations. Then
plot the plot location points on top of the Canopy Height Model. 

If you completed
[.csv to Shapefile in R]({{site.baseurl}}/R/csv-to-shapefile-in-R/)
you have these locations as a the spatial `R` Spatial object
`plot.locationsSp_HARV`. Otherwise, import the locations from the
`\HARV\PlotLocations_HARV.shp` shapefile in the downloaded data. 

</div>

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/challenge-code-crop-raster-points-1.png) 


##Define an Extent
We can also use an `extent()` method to define an extent to be used as a cropping
boundary. This creates an object of class `extent`.


    #extent format (xmin,xmax,ymin,ymax)
    new.extent <- extent(732161.2, 732238.7, 4713249, 4713333)
    class(new.extent)

    ## [1] "Extent"
    ## attr(,"package")
    ## [1] "raster"

Once we have defined the extent, we can use the `crop` function to crop our raster. 


    #crop raster
    CHM_HARV_manualCrop <- crop(x = chm_HARV, y = new.extent)
    
    #plot extent boundary and newly cropped raster
    plot(aoiBoundary_HARV, 
         main = "Manually Cropped Raster\n NEON Harvard Forest Field Site")
    plot(new.extent, col="blue", lwd=4,add = TRUE)
    plot(CHM_HARV_manualCrop, add = TRUE)

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/crop-using-drawn-extent-1.png) 

Notice that our manual `new.extent` is smaller than the `aoiBoundary_HARV` 
and that the raster is now the same as the `new.extent` object.
 
See the documentation for the `extent()` function for more ways
to create an `extent` object. 

* `help.search("extent", package = "raster"))
* More on the 
<a href="http://www.inside-r.org/packages/cran/raster/docs/extent" target="_blank">
extent class in `R`</a>.

##Extract Raster Pixels Values Using Vector Polygons

Often we want to extract values from a raster layer for particular locations - 
for example, plot locations that we are sampling on the ground. 

To do this in `R`, we use the `extract()` function. We will begin by extracting 
all canopy height pixel values located within our `aoiBoundary` polygon which
surrounds the tower located at the NEON Harvard Forest field site. The `extract()`
function requires:

* The raster that you wish to extract values from.
* The vector layer containing the polygons that you wish to use as a boundary or 
boundaries.
* OPTIONALLY: you can tell it to store the output values in a `data.frame` using
`df=TRUE`.

<figure>
    <a href="http://neondataskills.org/images/spatialData/BufferSquare.png">
    <img src="http://neondataskills.org/images/spatialData/BufferSquare.png"></a>
    <figcaption> Extract raster information using a polygon boundary. 
    Source: National Ecological Observatory Network (NEON).  
    </figcaption>
</figure>


    #extract tree height for AOI
    #set df=TRUE to return a data.frame rather than a list of values
    tree_height <- extract(x = chm_HARV, 
                           y = aoiBoundary_HARV, 
                           df=TRUE)
    
    #view the object
    head(tree_height)

    ##   ID HARV_chmCrop
    ## 1  1        21.20
    ## 2  1        23.85
    ## 3  1        23.83
    ## 4  1        22.36
    ## 5  1        23.95
    ## 6  1        23.89

    nrow(tree_height)

    ## [1] 18450

When we use the extract command, `R` extracts the value for each pixel located 
within the boundary of the polygon being used to perform the extraction - in this
case the `aoiBoundary` object (1 single polygon). In this case, the function extracted
values from 18,450 pixels.

The `extract` function returns a `LIST` of values as default. You can tell `R` 
to summarize the data in some way or to return a `data.frame` (`df=TRUE`).


We can create a histogram of tree height values in within the boundary to better
understand the structure or height distribution of trees. We can also use the 
`summary()` function to view descriptive statistics including min, max and mean
height values. These values help us better understand vegetation at our field site.


    #view histogram of tree heights in study area
    hist(tree_height$HARV_chmCrop, 
         main="Histogram of CHM Height Values (m) \nNEON Harvard Forest Field Site",
         col="springgreen",
         xlab="Tree Height", ylab="Frequency of Pixels")

![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/view-extract-histogram-1.png) 

    #view summary of values
    summary(tree_height$HARV_chmCrop)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    2.03   21.36   22.81   22.43   23.97   38.17

* Check out the documentation for the `extract()` function for more details 
(`help.search("extract", package = "raster")`).

##Summarize Extracted Raster Values 

We often want to extract summary values from a raster. We can tell `R` the type of
summary statistic we are interested in using the `fun=` method. Let's extract
a mean height value for our AOI. 


    #extract the average tree height (calculated using the raster pixels)
    #located within the AOI polygon
    av_tree_height_AOI <- extract(x = chm_HARV, 
                                  y = aoiBoundary_HARV,
                                  fun=mean, df=TRUE)
    
    #view output
    av_tree_height_AOI

    ##   ID HARV_chmCrop
    ## 1  1     22.43018

It appears that the mean height value, extracted from our LiDAR data derived canopy
height model is 22.43 meters.

##Extract Data using x,y Locations
We can also extract pixel values from a raster by defining a buffer or area 
surrounding individual point locations using the `extract()` function. To do this
we define the summary method (`fun=mean`) and the buffer distance (`buffer=20`)
which represents the radius of a circular region around each point.

The units of the buffer are the same units of the data `CRS` - in this case
`UTM - meters`.


<figure>
    <a href="http://neondataskills.org/images/spatialData/BufferCircular.png">
    <img src="http://neondataskills.org/images/spatialData/BufferCircular.png"></a>
    <figcaption> Extract raster information using a buffer region. 
    Source: National Ecological Observatory Network (NEON).  
    </figcaption>
</figure>


    #what are the units of our buffer
    crs(point_HARV)

    ## CRS arguments:
    ##  +proj=utm +zone=18 +datum=WGS84 +units=m +no_defs +ellps=WGS84
    ## +towgs84=0,0,0

    #extract the average tree height (calculated using the raster pixels)
    #located within the AOI polygon
    #use a buffer of 20 meters and mean function (fun) 
    av_tree_height_tower <- extract(x = chm_HARV, 
                                   y = point_HARV, 
                                   buffer=20,
                                   fun=mean, 
                                   df=TRUE)
    
    #view data
    head(av_tree_height_tower)

    ##   ID HARV_chmCrop
    ## 1  1     22.38812

    #how many pixels were extracted
    nrow(av_tree_height_tower)

    ## [1] 1

Notice that the command above extracted 1259 pixels from our LiDAR derived canopy
height model. Does this number make sense?

<div id="challenge" markdown="1">
#Challenge: Extract Raster Height Values For Plot Locations

Use the plot location points shapefile `HARV/plot.locations_HARV.shp` to 
extract an average tree height value for each plot location in the study area.

Create a simple plot of tree height across each plot
</div>


![ ]({{ site.baseurl }}/images/rfigs/04-vector-raster-integration-advanced/challenge-code-extract-plot-tHeight-1.png) 