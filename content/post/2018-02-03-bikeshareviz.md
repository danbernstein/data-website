---
title: bikeshareviz
author: Dan Bernstein
date: '2018-02-03'
slug: bikeshareviz
categories: []
tags: []
description: ''
image: ''
keywords: ''
draft: no
---
Cycling in Washington, DC is on the rise, and Capital Bikeshare has 
definitely played a role in getting more people out of their cars and onto their 
bikes. This project started with the idea to create a map of the most popular cycling
routes around the city, and it quickly became a larger endeavor to understand the patterns 
of Bikeshare usage at each station. This shiny app allows you to choose any station in
the network and then visualize up to the top ten most popular routes leaving from or ending at 
that station.

Data: Records of all Bikeshare rentals from the second quarter of 2016 through the end of
the first quarter of 2017 were taken from Capital Bikeshare's system data 
(https://s3.amazonaws.com/capitalbikeshare-data/index.html). There's usually a few months lag
after each quarter before the data is released, hence the time period used. A larger project
would include all rentals included in the system data going back to the fourth quarter of 2010,
giving a better visualization of how routes have developed as the network developed and 
residents signed on. It wouldn't be too difficult to aggregate all rides during each quarter
or calendar year and produce chloropleth maps to visualize how the system has developed
over the past eight years.


**Tools:**

-Shiny: Shiny is a wonderful R package that makes it incredibly easy to create dynamic, reactive web apps and deploy them for free. 

-Leaflet: Leaflet is a well-known Javascript library that helps create web mapping applications. Simply piping through 

The Code:
-Routing Algorithm: The map relies on the quick generation of the routes between start and end stations. 

```{r}
routing_function <- function(odf){
  odf$ID <- seq.int(nrow(odf))
  
  l <- vector("list", nrow(odf))
  for(i in 1:nrow(odf)){
  # extracts the longitude and latitude of the starting and ending stations
    o = c(odf$start.lon[i], odf$start.lat[i])
    d = c(odf$end.lon[i], odf$end.lat[i])
  # Creates a Line object between the start and end location
    l[[i]] <- sp::Lines(list(sp::Line(rbind(o, d))), as.character(i))
  }
  
  l <- sp::SpatialLines(l)
  # defines the coordinate reference frame
  proj4string(l) <- CRS("+init=epsg:4326")
  # adds station features to the existing Line object
  l <- SpatialLinesDataFrame(l, odf, match.ID = "ID")
  
  # generates the cycling route using the Open Source Routing Machine 
  routes_fast <- line2route(l = l, route_fun = route_osrm)
}
```

# **Future Work:**

Compare across years: As I previously mentioned, it would be great to compare how the network has developed over time by mapping how the most popular routes from each station change as more stations are added and adoption increases. 

Tweak the routing algorithm: The current routing algorithm does not have a good understanding of which roads cyclists are likely to travel as they move between two points; the algorithm looks for the shortest path in road networks. The algorithm often maps routes that utilize major roads that lack cyclist infrastructure and likely do not mimic the route that cyclists actually take. There are other routing algorithms that produce routes that are more cyclist-friendly based on speed limits and other factors. 

