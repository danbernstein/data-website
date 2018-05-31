---
title: Regression Analysis of Total Citations among CRISPR Publications 
author: Dan Bernstein
date: '2018-05-29'
slug: crispr-regression
categories: []
tags:
  - R
  - Regression Analysis
  - Bibliometrics
description: ''
image: ''
keywords: ''
draft: no
---

## **Motivation**
This is a quick and dirty regression analysis of the total number of citations that a corpus of documents have received as of the time that the bibliographic data was downloaded. 


## **Data**  
Publication entries on the topic "crispr" were collected from Web of Science (WOS) on May 26, 2018. Using the topic field query includes both documents that include the search term in the title and abstract. The exported data from WOS is complex, including author information, various IDs, and citation fields, including total citations and a full list of all documents that have referenced the given publication. The introduction to the [Bibliometrix package](https://cran.r-project.org/web/packages/bibliometrix/vignettes/bibliometrix-vignette.html) gives a great overview of the data structure. 

## **Tools**

The Bibliometrix package was used to extract metadata about the authors' countries of origin. The data wrangling and linear regression are simply a combination of base R and tidyverse tools.

## **Results**

<div style="width:image width px; font-size:80%; text-align:center;"><img src="/img/crispr_regression.png"  style = "display: block;width:50%;
    margin: 0 auto; background-color:white;"; />Fig. Screenshot of the regression output. </div>



## **Future Work**

Future work will include: 

- Regression analysis on the number of times a publication is cited, using only the information included in the WOS exported data, including the name of the journal, the home country of the lead author, and other transformed variables that we can create using the dataset.

- A bibliographic coupling analysis to attempt to cluster documents based on subject matter.

Stay tuned!

