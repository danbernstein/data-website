---
title: Visualizing Personal Use
author: Dan Bernstein
date: '2018-03-03'
slug: personalviz
categories: []
tags:
  - R
  - Shiny
  - rvest
description: ''
image: 'bikeshareviz-map.png'
keywords: ''
draft: no
---
Personalized maps: A natural extension of looking at where capital bikeshare members ride would be to personalize the visualization, allowing individuals to upload their ride record and quickly aggregate the route frequencies and visualize. 
Check out the visualization at [https://bikeshareviz.shinyapps.io/personal_bikesharedata/](https://bikeshareviz.shinyapps.io/personal_bikesharedata/)

## **Data**  
This Shiny app relies on webscarping user ride history from the [CaBi website](https://www.capitalbikeshare.com/) using the [rvest](https://github.com/hadley/rvest) package. After entering account login credentials, the server determines the number of rides the user has taken, and then cycles through all the webpages to extract relevant information (start and end station, start and end time, and duration). The data is geocoded by merging with the [Capital Bike Share Locations](http://opendata.dc.gov/datasets/capital-bike-share-locations) dataset freely available from the DC government open data initiative.

## **Tools**

rvest: 

Shiny: [Shiny](https://shiny.rstudio.com/) is a wonderful R package that makes it incredibly easy to create dynamic, reactive web apps and deploy them for free. Unforunately, the app's performance is incredibly slow, with a few possible explanations: 
   •  The code is not optimized (there are numerous CSS styling elements included in the R script. Moving       these elements to the style sheet would improve readability and possibly performance
   •  The free Shiny services are limited to 1GB RAM, limiting the speed of complex server operations


## **Code Features**  
Routing Algorithm: The map uses the routing functions provided by the [stplanr](https://github.com/ropensci/stplanr) package. The algorithm simply extracts the geographic coordinates of the start and end stations to generate. The routes are based on a simple dataset (keypairs_latlon) of the start and end locations (including geographic coordinates), and the number of rides on that routes in the dataset (freq). After routing all keypairs, the dataset is merged into a single network, with weight given to road segments that have high frequency in the dataset. 


Visualization: The final dataset is fed into ggplot2, then a png file is rendered to increase resolution.
The app then pipes the data through the parameters chosen by the user. Users can either enter login credentials for webscraping their personal ride history, or they can use the example data, which includes the 100 most popular routes from the second quarter of 2016 through the end of the first quarter of 2017.
The initial ggplot parameters are identical, however there are two reactive elements, one for the user data and another for the example data. The output object that is ultimately displayed uses a simple if/else construct to determine which to display in the UI. 


```{r}
  
```




```{r}

```

## **Future Work**

The app is quite quick when run locally, but is rendered exceedingly slow when deployed bceause the server-side operations (webscraping, routing, updating the rendered image). I will investigate the potential issues in the future and hopefully redeploy, either on Shiny or somewhere else, when I have a solution. For the time being, users familiar with R should have no problem running the script locally. 

Expanding tthe scope: With the introduction of various dockless bikeshare systems in 2017, there is room to also scrape and visualize users' usage of other bikeshare programs. Similarly, simple expansions in the webscraping would allow riders who use ridewithgps or strava to layer their riding network as well. 


Tweak the routing algorithm: The current routing algorithm does not have a good understanding of which roads cyclists are likely to travel as they move between two points; the algorithm looks for the shortest path in road networks. The algorithm often maps routes that utilize major roads that lack cyclist infrastructure and likely do not mimic the route that cyclists actually take. There are other routing algorithms that produce routes that are more cyclist-friendly based on speed limits and other factors that are worth exploring.



