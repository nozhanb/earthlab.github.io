---
layout: single
title: "An example of creating modular code in R - Efficient scientific programming"
excerpt: "This lesson provides an example of modularizing code in R. "
authors: ['Carson Farmer', 'Leah Wasser']
modified: '2017-03-30'
category: [course-materials]
class-lesson: ['intro-APIs-r']
permalink: /course-materials/earth-analytics/week-10/leaflet-r/
nav-title: 'Leaflet '
week: 10
sidebar:
  nav:
author_profile: false
comments: true
order: 6
---

{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

*

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson.

</div>






```r
library("dplyr")
library("ggplot2")
library("RCurl")
library("rjson")
library("jsonlite")
library(leaflet)
```




## Interactive maps with Leaflet

Static maps are useful for creating figures for reports and presentation. Sometimes,
however, we want to interact with our data. We can use the leaflet package for
R to overlay our data on top of interactive maps. You can think about it like
google  maps with your data overlane on top!

### What is leaflet?

<a href="http://leafletjs.com" target="_blank">Leaflet</a> is an open-source `JavaScript` library that can be used to create mobile-friendly interactive maps.

Leaflet:

* Is designed with *simplicity*, *performance* and *usability* in mind,
* Has a beautiful, easy to use, and <a href="http://leafletjs.com/reference.html" target="_blank">well-documented API</a>


The `leaflet` `R` package 'wraps' Leaflet functionality in an easy to use `R` package! Below, you can see some code that creates a basic web-map.


```r
map = leaflet() %>%
  addTiles() %>%  # use the default base map which is OpenStreetMap tiles
  addMarkers(lng=174.768, lat=-36.852,
             popup="The birthplace of R")
print(map)
```


<iframe title="Basic Map" width="80%" height="600" src="{{ site.url }}/leaflet-maps/birthplace_r.html" frameborder="0" allowfullscreen></iframe>


## Create your own interactive map

Let's create our own interactive map using the surface water data that we used
in the previous lessons, using leaflet. To do this, we will follow the steps below:

1. Request and get the data from the colorado.gov SODA API in `R` using `getURL()`.
2. Convert the data from JSON to a data.frame in `R` using `fromJSON()`.
3. Address column data types to ensure our numeric data are in fact numeric.
3. Remove `NA` (missing data) values.

The code below is the same code that we used to process the surface water
data in the previous lesson.


```r
base_url <- "https://data.colorado.gov/resource/j5pc-4t32.json?"
full_url <- paste0(base_url, "station_status=Active",
            "&county=BOULDER")
water_data <- getURL(URLencode(full_url))
water_data_df <- fromJSON(water_data)
# remove the nested data frame
water_data_df <- flatten(water_data_df, recursive = TRUE)

# turn columns to numeric and remove NA values
water_data_df <- water_data_df %>%
  mutate_each_(funs(as.numeric), c( "amount", "location.latitude", "location.longitude")) %>%
  filter(!is.na(location.latitude))
```

Once our data are cleaned up, we can create our leaflet map. Notice that we are
using pipes `%>%` to set the parameters for the leaflet map.



```r
# create leaflet map
leaflet(water_data_df) %>%
  addTiles() %>%
  addCircleMarkers(lng=~location.longitude, lat=~location.latitude)
```


The code below provides an example of creating the same map without using pipes.


```r
map <- leaflet(water_data_df)
map <- addTiles(map)
map <- addCircleMarkers(map, lng=~location.longitude, lat=~location.latitude)
```



<iframe title="Basic Map" width="80%" height="600" src="{{ site.url }}/leaflet-maps/water_map1.html" frameborder="0" allowfullscreen></iframe>

## Customize leaflet maps

We can customize our leaflet map too. Let's do the following:

1. add some popups to our map and adjust the marker symbology.
2. adjust the basemap. Let's use a basemap from
3. <a href="https://carto.com/blog/getting-to-know-positron-and-dark-matter" target="_blank">CartoDB</a>
called Positron.

Notice in the code below that we can specify the popup text using the `popup=`
argument.

`addMarkers(lng=~location.longitude, lat=~location.latitude, popup=~station_name)`

We specify the basemap using the addProviderTiles() argument:

`addProviderTiles("CartoDB.Positron")`


```r
leaflet(water_data_df) %>%
  addProviderTiles("CartoDB.Positron") %>%
  addMarkers(lng=~location.longitude, lat=~location.latitude, popup=~station_name)
```



<iframe title="Basic Map" width="80%" height="600" src="{{ site.url }}/leaflet-maps/water_map2.html" frameborder="0" allowfullscreen></iframe>

### Custom icons

We can even specify a *custom* icon, just for fun. BElow, we are using an icon
from the <a href="http://tinyurl.com/jeybtwj" target="_blank">web</a>.

Notice also that we are customizing the popup even more, adding BOTH the station
name AND the discharge value.

`paste0(station_name, "<br/>Discharge: ", amount)`

We are using paste0() to do this. Remeber that paste0() will paste together a
series of text strings and object values.


```r
# let's look at the output of our popup text before calling it in leaflet 
paste0(station_name, "<br/>Discharge: ", amount)
## Error in paste0(station_name, "<br/>Discharge: ", amount): object 'station_name' not found
```

Finally, let's see what the custom icon and popup text looks like on our map! 


```r
# Specify custom icon
url = "http://tinyurl.com/jeybtwj"
water = makeIcon(url, url, 24, 24)

leaflet(water_data_df) %>%
  addProviderTiles("Stamen.Terrain") %>%
  addMarkers(lng=~location.longitude, lat=~location.latitude, icon=water,
             popup=~paste0(station_name,
                           "<br/>Discharge: ",
                           amount))
```



<iframe title="Basic Map" width="80%" height="600" src="{{ site.url }}/leaflet-maps/water_map3.html" frameborder="0" allowfullscreen></iframe>

That's it. Experiment with leaflet a bit more to learn more about customizing your maps.