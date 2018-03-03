---
title: personalviz
author: Dan Bernstein
date: '2018-03-03'
slug: personalviz
categories: []
tags:
  - R
  - Shiny
  - rvest
description: ''
image: ''
keywords: ''
draft: no
---
/*Background*/
Cycling in Washington, DC is on the rise, and Capital Bikeshare has 
definitely played a role in getting more people out of their cars and onto their 
bikes. This project started with the idea to create a map of the most popular cycling
routes around the city, and it quickly became a larger endeavor to understand the patterns 
of Bikeshare usage at each station. This shiny app allows you to choose any station in
the network and then visualize up to the top ten most popular routes leaving from or ending at 
that station.  
Check out the visualization at [https://bikeshareviz.shinyapps.io/shiny/](https://bikeshareviz.shinyapps.io/shiny/)

## **Data**  
This Shiny app relies on webscarping user ride history from the [CaBi website](https://www.capitalbikeshare.com/) using the [rvest](https://github.com/hadley/rvest) package. After entering account login credentials, the server determines the number of rides the user has taken, and then cycles through all the webpages to extract relevant information (start and end station, start and end time, and duration). The data is geocoded by merging with the [Capital Bike Share Locations](http://opendata.dc.gov/datasets/capital-bike-share-locations) freely available from the DC government open data initiative.

## **Tools**

rvest: 

Shiny: [Shiny](https://shiny.rstudio.com/) is a wonderful R package that makes it incredibly easy to create dynamic, reactive web apps and deploy them for free. Unforunately, the app's performance is incredibly slow, with a few possible explanations: 
   •  The code is not optimized (there are numerous CSS styling elements included in the R script. Moving       these elements to the style sheet would improve readability and possibly performance
   •  The free Shiny services are limited to 1GB RAM, limiting the speed of complex server operations


## **Code Features**  
Routing Algorithm: The map uses the routing functions provided by the [stplanr](https://github.com/ropensci/stplanr) package. The algorithm simply extracts the geographic coordinates of the start and end stations to generate. The routes are based on a simple dataset (keypairs_latlon) of the start and end locations (including geographic coordinates), and the number of rides on that routes in the dataset (freq). After routing all keypairs, the dataset is merged into a single network, with weight given to road segments that have high frequency in the dataset. 


Visualization: The final dataset is fed into ggplot2, then a png file is rendered to increase resolution.
The app then pipes the data through the parameters chosen by the user. The output of the piping enters the routing function.  

```{r}
    ## reactive() is a shiny call to that updates data in real-time as users change parameters
    reactive_bikeroutes <- reactive({
    ## Parameter: Is the station the origin or destination? 
    if(input$Origin == "Origin"){
      return((keypairs_latlon) %>% 
               filter(Start.station.number == input$Start.station.number,
                      Start.station.number != End.station.number) %>% 
               arrange(desc(as.numeric(freq))) %>% 
    ## Parameter: The number of routes to visualize is chosen with the slice function
               slice(1:input$routes) %>% 
               as.data.frame() %>% 
               routing_function())}
    else{
      return((keypairs_latlon) %>% 
               filter(End.station.number == input$Start.station.number,
                      Start.station.number != End.station.number) %>% 
               arrange(desc(as.numeric(freq))) %>% 
               slice(1:input$routes) %>% 
               as.data.frame() %>% 
               routing_function()) }
  })
```


The subset of station key pairs is then converted to straight lines between the start and end stations, and the routes is generated using the line2route function from the stplanr package, an extensive set of tools for transportation planning.  

```{r}
routing_function <- function(odf){
  odf$ID <- seq.int(nrow(odf))
  
  # creates a list of empty vectors for each route
  l <- vector("list", nrow(odf))
  for(i in 1:nrow(odf)){
  # extracts the longitude and latitude of the starting and ending stations
    o = c(odf$start.lon[i], odf$start.lat[i])
    d = c(odf$end.lon[i], odf$end.lat[i])
  # Creates a Line object between the start and end location, this is a straight line that has not been modified by the street network yet
    l[[i]] <- sp::Lines(list(sp::Line(rbind(o, d))), as.character(i))
  }
  
  l <- sp::SpatialLines(l)
  # defines the coordinate reference frame
  proj4string(l) <- CRS("+init=epsg:4326")
  # adds station features to the new Line objects
  l <- SpatialLinesDataFrame(l, odf, match.ID = "ID")
  
  # generates the cycling route using the line2route function from the stplanr package and the Open Source Routing Machine (OSRM) routing function
  routes_fast <- line2route(l = l, route_fun = route_osrm)
}
```

## **Future Work**

Compare across years: As I previously mentioned, it would be great to compare how the network has developed over time by mapping how the most popular routes from each station change as more stations are added and adoption increases. 

A larger project would include all rentals included in the system data (dating to late 2010), giving a better visualization of how routes have developed as the network developed and residents signed on. It wouldn't be too difficult to aggregate all rides during each quarter
or calendar year and produce chloropleth maps to visualize how the system has developed
over the past eight years or extend the shiny app with an additional year parameter that might move play through the top routes year over year, similar to using [gganimate](https://github.com/dgrtwo/gganimate).


Tweak the routing algorithm: The current routing algorithm does not have a good understanding of which roads cyclists are likely to travel as they move between two points; the algorithm looks for the shortest path in road networks. The algorithm often maps routes that utilize major roads that lack cyclist infrastructure and likely do not mimic the route that cyclists actually take. There are other routing algorithms that produce routes that are more cyclist-friendly based on speed limits and other factors that are worth exploring.

Personalized maps: A natural extension of looking at where capital bikeshare members ride would be to personalize the visualization, allowing individuals to upload their ride record and quickly aggregate the route frequencies and visualize. 

