---
title: Network Analysis in International Scientific Collaboration
author: Dan Bernstein
date: '2018-05-26'
slug: crispr-network
categories: []
tags:
  - R
  - Network Analysis
  - Bibliometrics
description: ''
image: ''
keywords: ''
draft: no
---

## **Motivation**
I have been thinking about networks of science for awhile: collaboration, competition, intellectual property and licensing, etc. At the same time, the CRISPR genome editing technique has emerged as the primary gene-editing tool in laboratories across the world (sorry TALENS). 


## **Data**  
Publication entries on the topic "crispr" were collected from Web of Science (WOS) on May 26, 2018. Using the topic field query includes both documents that include the search term in the title and abstract. The exported data from WOS is complex, including author information, various IDs, and citation fields, including total citations and a full list of all documents that have referenced the given publication. The introduction to the [Bibliometrix package](https://cran.r-project.org/web/packages/bibliometrix/vignettes/bibliometrix-vignette.html) gives a great overview of the data structure. 

## **Tools**

In addition to the Bibliometrix package, I used the traditional R packages for network analysis, including sna (stands for Social Network Analysis), igraph, intergraph, and network, as needed.

## **Code Features**  

## **Future Work**

Compare across years: As I previously mentioned, it would be great to compare how the network has developed over time by mapping how the most popular routes from each station change as more stations are added and adoption increases. 

A larger project would include all rentals included in the system data (dating to late 2010), giving a better visualization of how routes have developed as the network developed and residents signed on. It wouldn't be too difficult to aggregate all rides during each quarter
or calendar year and produce chloropleth maps to visualize how the system has developed
over the past eight years or extend the shiny app with an additional year parameter that might move play through the top routes year over year, similar to using [gganimate](https://github.com/dgrtwo/gganimate).


Tweak the routing algorithm: The current routing algorithm does not have a good understanding of which roads cyclists are likely to travel as they move between two points; the algorithm looks for the shortest path in road networks. The algorithm often maps routes that utilize major roads that lack cyclist infrastructure and likely do not mimic the route that cyclists actually take. There are other routing algorithms that produce routes that are more cyclist-friendly based on speed limits and other factors that are worth exploring.

Personalized maps: A natural extension of looking at where capital bikeshare members ride would be to personalize the visualization, allowing individuals to upload their ride record and quickly aggregate the route frequencies and visualize. 

