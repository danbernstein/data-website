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

Under development.

## **Future Work**

Under development.

