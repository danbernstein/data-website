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
I have been thinking about networks of science for awhile: collaboration, competition, intellectual property and licensing, etc. At the same time, CRISPR (Clustered Regularly Interspaced Palindromic Repeats) has emerged as the preeminent gene-editing tool in laboratories across the world (sorry TALENs and all you other fun tools). MIT's Broad Institute and UC Berkeley have been in a protracted legal battle over the patent for the technology underlying new advances in medicine, agriculture, and other consumer industries. The democratization of the new gene-editing technologies is both exciting and scary; individuals with a basic biochemistry knowledge can mix the life sciences with targetted genome ediing to reengineer biological systems. 

With all this in mind, I set out to learn the basics of network analysis to explore trends in CRISPR. After reading through a User's Guide to Network Analysis in R, I have been trying to think of a project in which data collection would not be too arduous (isn't that always the problem?) and the analysis would give me opportunity for additional experience in metrics and modelling, rather than simply visualization. This post focuses on the methods for working with bibliographic data in R and network statistics. Future work will include modelling.


## **Data**  
Publication entries on the topic "crispr" were collected from Web of Science (WOS) on May 26, 2018. Using the topic field query includes both documents that include the search term in the title and abstract. The exported data from WOS is complex, including author information, various IDs, and citation fields, including total citations and a full list of all documents that have referenced the given publication. The introduction to the [Bibliometrix package](https://cran.r-project.org/web/packages/bibliometrix/vignettes/bibliometrix-vignette.html) gives a great overview of the data structure. 

## **Tools**

In addition to the Bibliometrix package, I used the traditional R packages for network analysis, including sna (stands for Social Network Analysis), igraph, intergraph, and network, as needed.

## **Results**
This analysis is a multi-step process that will involve both network analysis and traditional statistical analysis on non-network aspects. To get a general idea for how collaboration has developed over time, we can look at the size of the networks for all publications in each year in the dataset, as well as some network statistics. The number of countries engaged in crispr research steadily rose from 2005 to 2013 before adding nearly ten new countries per year between 2013 and 2018. It would worth noting that 2006 is an anomaly, 27 countries were in that network because a large international research collaboration published a single paper. That does not diminish the value of that collaboration, but from a standpoint of looking at the growth of research collaborations, one large collaboration must be distinguished from numerous smaller ones.


<div style="width:image width px; font-size:80%; text-align:center;"><img src="/img/network_summaries.png"  style = "display: block;width:50%;
    margin: 0 auto; background-color:white;"; />Fig. Network features and summary statistics (mean values) for publications in each year between 2005 and 2018. <br/> NB: the 2018 statistics are not comparable because the data was pulled in May 2018 and is thus incomplete. </div>
    
Looking at the number of isolates, or countries that are in the dataset but are not engaged in collaboration with any other countries, we see that there have been a reliable few every year, never rising beyond six. I hope to further analyze these isolates to determine if the same countries are consistently isolated or if they dabble in crispr research on their own and then begin engaging in collaborations afterwards. This analysis will include tracking countries after they appear as isolates in the subsequent years to determine how collaborative they become. 

Turning to the summary statistics, we notice a few things:

- The density and degree further prove that 2006 is an anomaly. With a density of 1.0, that means all nodes have ties to all other nodes in the network. That large international collaboration I mentioned? That's it. Because all those authors from different countries are on a single paper, they all share ties with each other, yielding a real number of ties that is equal to the total potential number. The average degree also shows that each node is tied to 26 other nodes, clearly the result of a single highly-collaborative paper.

- Since the explosion in countries involved in crispr research since 2013, the overall collaboration has risen evidenced by the graph density climbing to 0.64 in 2017, meaning 64% of all possible ties between countries are actually in the network. That being said, there might be a large cohort of countries that make up a highly-interconnected clique, whereas smaller groups only have a few ties. Examining this distribution and reasons for collaboration patterns, be it geopolitical, geographic, or another reason, is highly valuable.

- Various measures of node centrality, including betweenness, load, and stress, all rise across the time period. The eigenvector centrality measure ('eigen' in the table) actually drops over time. I will explore that reasoning in the future. 



## **Code Features**  

Under development.

## **Future Work**

Future work will include: 

- Regression analysis on the number of times a publication is cited, using only the information included in the WOS exported data, including the name of the journal, the home country of the lead author, and other transformed variables that we can create using the dataset. (I completed this analysis a few days later, check out the results [here](https://danbernstein.netlify.com/post/crispr-regression/))

- A bibliographic coupling analysis to attempt to cluster documents based on subject matter.



