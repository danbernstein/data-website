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
image: bikeshareviz-map.png
keywords: ''
draft: no
---

This project is a follow-on to my previous bikesharviz project, where I mentioned creating another app to enable users to quickly visualize their own ridership. This app enables users to gather their data through webscraping, then server-side processing to generate the cycling routes, and then client-side customization of the rendered graphic. 

Check out the app at [https://bikeshareviz.shinyapps.io/personal_bikesharedata/](https://bikeshareviz.shinyapps.io/personal_bikesharedata/)

## **Data**  
This Shiny app relies on webscraping user ride history from the [CaBi website](https://www.capitalbikeshare.com/) using the [rvest](https://github.com/hadley/rvest) package. After entering account login credentials, the server determines the number of rides the user has taken, and then cycles through all the webpages to extract relevant information (start and end station, start and end time, and duration). The data is geocoded by merging with the [Capital Bike Share Locations](http://opendata.dc.gov/datasets/capital-bike-share-locations) dataset freely available from the DC government open data initiative.

## **Tools and Code Features**

rvest: [rvest](https://github.com/hadley/rvest) is a webscraping package from Hadley Wickham that uses simple commands and integrates easily with magrittr. 


```{r}
## html_session directs the browser to the target webpage
cabi <- html_session("https://secure.capitalbikeshare.com/profile/login")

## the login form is extracted using html_form
cabi.form <- html_form(cabi)

## after the user inserts their username and password into the popup box, the app sets the CSS elements
## of the CaBi login page to that information
login <- set_values(cabi.form[[2]],
                        '_username'  = input$username,
                        '_password'  = input$password)
                        
## webpage navigation is easy using the jump_to function to jump to the ride history URL, which is universal to all users
gotin <- cabi %>% submit_form(login) %>% 
jump_to("https://secure.capitalbikeshare.com/profile/trips/YR5RUVKF-1?pageNumber=1") 

## To find out how many pages of rides there are, we use the follow_link page to "click" on 
## the "Oldest" button on the webpage, just like if we were actually browsing. The regular pattern is 
## "https://secure.capitalbikeshare.com/profile/trips/YR5RUVKF-1?pageNumber=[0-9]*", where the digits after the last "=" is the final page number and can be any number of digits greater than or equal to 1. 

## we use regular expressions to remove everything after the final "="
    
    last_page_html <- gotin %>% 
      follow_link("Oldest")
    
    last_page_number <- last_page_html$url %>% 
      gsub("^.*?=","",.)
      
## This leaves us with a numeric constant (7, 81, 900, etc.)
```

```{r}
## Using the number of pages found through regular expressions, we do a few things to scrape all ride history: 
(1) generate URLs for each ride history webpage
(2) create an empty data frame to hold all the scraped data
(3) loop through all the URLs to extract the HTML nodes with the relevant information
## The result is a data frame with five columns:  start date and time,end date and time, start station, end station, duration.

## l_out stores the numeric value of the number of pages. In the app, the user will have the option to select a checkbox if they only want to select the last 100 rides (each page contains 10 rides, thus the ifelse statement will loop through ten pages if checked)

  l_out <- ifelse(input$checkbox == FALSE, as.numeric(last_page_number), 10)  
  
  ## this needs to be set to the user's number of pages

    url_base = "https://secure.capitalbikeshare.com/profile/trips/YR5RUVKF-1?pageNumber="

    urls <- paste0(url_base, seq(1, by = 1, length.out = l_out))
    
    results<-data.frame()  
    
## withProgress is a Shiny feature that will display a loading bar in the UI
    withProgress(message = 'Gathering Your Rides', value = 0, {
      # Number of times we'll go through the loop
      n <- l_out
      
    for(i in 1:length(urls)){
      ok <- gotin %>% jump_to(urls[i]) %>% read_html()
      
      startdatetime <-  ok %>% 
        html_nodes(".ed-table__col_trip-start-date") %>% 
        html_text() %>% 
        parse_character() %>% 
        as.data.frame()
      
      enddatetime <-  ok  %>% 
        html_nodes(".ed-table__col_trip-end-date") %>% 
        html_text() %>% 
        parse_character() %>% 
        as.data.frame()
      
      startstation <- ok %>% 
        html_nodes(".ed-table__col_trip-start-station") %>% 
        html_text() %>% 
        parse_character() %>% 
        as.data.frame()
      
      endstation <- ok %>% 
        html_nodes(".ed-table__col_trip-end-station") %>% 
        html_text() %>% 
        parse_character() %>% 
        as.data.frame()
      
      duration <- ok %>% 
        html_nodes(".ed-table__item__info_trip-duration") %>% 
        html_text() %>% 
        parse_character() %>% 
        as.data.frame()
      
      bind <- cbind(startdatetime, enddatetime, startstation, endstation, duration)
      
      results <- rbind(results, bind)
      
      incProgress(1/n, detail = paste0("Scraping Page ", i, " out of ", l_out))
      
    }})
    

```

More regular expressions: The scraped nodes need to be processed to remove leading and trailing characters to create a pretty dataset that is easy to feed into the routing function. 

```{r}
## each column is given a name
 names(results) <- c("startdatetime", "enddatetime", "startstation", "endstation", "duration")

## the data.frame is processed to create datatime objects (using lubridate) and leading character strings are removed from the startstation, end station, and duration objects
    results0 <- results %>% 
      mutate(startdatetime = lubridate::mdy_hms(gsub("Start                        ", "", startdatetime)),
             enddatetime = lubridate::mdy_hms(gsub("End                        ", "", enddatetime)),
             startdate = lubridate::as_date(startdatetime),
             enddate = lubridate::as_date(enddatetime),
             startstation = gsub("\n                                ", "", startstation),
             endstation = gsub("\n                                ", "", endstation),
             duration = gsub("\n                                ", "", duration),

## the datetime objects are sliced to extract the hour, minute, and second features and then paste them back together in a format that will allow for users to order the rides by duration.
             hour = if_else(grepl(" h ", duration), as.character(str_match(duration, "(.*?) h")[,2]), "00"),
             hour = ifelse(nchar(hour) == 1, paste0("0", hour), hour),
             
             minute = if_else(grepl(" h ", duration), 
                              str_match(duration, " h (.*?) m")[,2], 
                              as.character(str_match(duration, "(.*?) m")[,2])),
             
             minute = ifelse(nchar(minute) == 1, paste0("0", minute), minute),
             
             second = str_match(duration, " m (.*?) s")[,2],
             second = ifelse(nchar(second) == 1, paste0("0", second), second),
             
             time = paste0(hour, ":", minute, ":", second)
      )
```

Shiny: [Shiny](https://shiny.rstudio.com/) is a wonderful R package that makes it incredibly easy to create dynamic, reactive web apps and deploy them for free. Unforunately, the app's performance is incredibly slow, with a few possible explanations: 
   •  The code is not optimized (there are numerous CSS styling elements included in the R script. Moving       these elements to the style sheet would improve readability and possibly performance
   •  The free Shiny services are limited to 1GB RAM, limiting the speed of complex server operations

The customization options are all introduced as inputs, and can be reset through an observeEvent trigger, where clicking the "reset" button will change all color inputs back to their original setting. The user still needs to click the "update" button to then initiate the default colors. The same goes for the reset button for the the filtering options. 

```{r}
## reset filters and color palette options
observeEvent(input$reset_colors, {
  updateColourInput(session, "background_color", value = "black")
  updateColourInput(session, "water_color", value = "#708090")
  updateColourInput(session, "ride_color", value = "red")
  updateSliderInput(session, "linewidth_slider", value = c(0.3,1.0)) 
})

observeEvent(input$reset_filters, {
  updateNumericInput(session, "pages", value = "100")
  updateDateRangeInput(session, "dateRange", start = "2010-01-01", end = Sys.Date())
})
```

Routing Algorithm: The map uses the routing functions provided by the [stplanr](https://github.com/ropensci/stplanr) package. The algorithm simply extracts the geographic coordinates of the start and end stations to generate. The routes are based on a simple dataset (keypairs_latlon) of the start and end locations (including geographic coordinates), and the number of rides on that routes in the dataset (freq). After routing all keypairs, the dataset is merged into a single network, with weight given to road segments that have high frequency in the dataset. 


Visualization: The final dataset is fed into ggplot2, then a png file is rendered to increase resolution.
The app then pipes the data through the parameters chosen by the user. Users can either enter login credentials for webscraping their personal ride history, or they can use the example data, which includes the 100 most popular routes from the second quarter of 2016 through the end of the first quarter of 2017.
The initial ggplot parameters are identical, however there are two reactive elements, one for the user data and another for the example data. The output object that is ultimately displayed uses a simple if/else construct to determine which to display in the UI. 

The basemap only includes the cycling routes and the overall background parameters: 
```{r}
map.base  <- eventReactive(input$submit | input$filter | input$update | input$playdata, {
  mapa.df <- map.out()

  mapa.df$scale.norm <- scales::rescale(mapa.df$count, 
                                        to = c(min(input$linewidth_slider),max(input$linewidth_slider)))
  mapa.df$scale.norm.log <- scales::rescale(mapa.df$count.log, to = c(0.1,1))
  
 total <-  ggplot() + 
    coord_map(xlim=c(min(mapa.df$long)-.01, max(mapa.df$long)+0.01),
              ylim=c(min(mapa.df$lat)-0.01, max(mapa.df$lat)+0.01)) +
    geom_path(data = mapa.df, aes(x=long, y=lat, group = group), 
              size = mapa.df$scale.norm, color = input$ride_color, lineend = "round")+
    #scale_color_gradient(low = "blue", high = "red")+
    theme(panel.grid.minor=element_blank(), panel.grid.major=element_blank(),
          panel.background=element_blank()) + 
    ## background color
    theme(plot.background=element_rect(fill="grey"),
          panel.background=element_rect(fill=input$background_color), 
          legend.background= element_rect(fill="black", colour=NA),
          legend.key = element_rect(colour = NA, col = "grey", 
                                    size = .5, fill = 'black'),
          axis.text = element_blank(),
          legend.position = "none",
          axis.line=element_blank(),axis.text.x=element_blank(),
          axis.text.y=element_blank(),axis.ticks=element_blank(),
          axis.title=element_blank())

    total
})
```
The default map that users see takes the basemap with the routes and adds the water layer, to provide a little orientation to the area. Future functions would allow users to add other layers, such as roads, points of interest, bike lanes, etc. 

```{r}
map.base.water  <- eventReactive(input$submit | input$filter | input$update | input$playdata, {
  ok <- map.base()
  ok+
  ggpolypath::geom_polypath(data = water.df, aes(long, lat, group = group), fill = input$water_color)
})
```


## **Future Work**

The app is quite quick when run locally, but is rendered exceedingly slow when deployed bceause the server-side operations (webscraping, routing, updating the rendered image). I will investigate the potential issues in the future and hopefully redeploy, either on Shiny or somewhere else, when I have a solution. For the time being, users familiar with R should have no problem running the script locally. 

Expanding tthe scope: With the introduction of various dockless bikeshare systems in 2017, there is room to also scrape and visualize users' usage of other bikeshare programs. Similarly, simple expansions in the webscraping would allow riders who use ridewithgps or strava to layer their riding network as well. 


Tweak the routing algorithm: The current routing algorithm does not have a good understanding of which roads cyclists are likely to travel as they move between two points; the algorithm looks for the shortest path in road networks. The algorithm often maps routes that utilize major roads that lack cyclist infrastructure and likely do not mimic the route that cyclists actually take. There are other routing algorithms that produce routes that are more cyclist-friendly based on speed limits and other factors that are worth exploring.



