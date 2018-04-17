---
title: Text Mining 
author: Dan Bernstein
date: '2018-03-20'
slug: text-mining-sotu
categories: []
tags:
  - R
  - plotly
  - text mining
description: ''
image: ''
keywords: ''
draft: no
---

I've wanted to explore text mining for awhile. The idea of drawing out core concepts and subtle themes among a corpus of documents adds a quantitative angle to analysis, whether in literature, policy, or science. This post is the first in a series using the [tidytext](https://www.tidytextmining.com/tidytext.html) library and the great free online book, [Text Mining in R](https://www.tidytextmining.com/tidytext.html) to explore some interesting collections of text.

## **The State of the Union**

Every president to date has delivered the State of the Union (SOTU) address annually to announce successes from the previous year and set administration priorities for the year to come. The enduring intent for the SOTU makes it a prime candidate for text mining, to illustrate how the language surrounding political messaging has changed over two hundred years. Using the tools available in the tidytext and [topicmodels](https://cran.r-project.org/web/packages/topicmodels/vignettes/topicmodels.pdf) packages, text mining reveals a number of trends.

<div>
    <a href="https://plot.ly/~danbernstein/5/?share_key=v9uX92K4tJ912F9YS2EBhe" target="_blank" title="Plot 5" style="display: block; text-align: center;"><img src="https://plot.ly/~danbernstein/5.png?share_key=v9uX92K4tJ912F9YS2EBhe" alt="Plot 5" style="max-width: 100%;width: 700px;"  width="700" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="danbernstein:5" sharekey-plotly="v9uX92K4tJ912F9YS2EBhe" src="https://plot.ly/embed.js" async></script>
</div>

Topic Modeling: 

The most apparent trend is the long-term changes in topic model compositions in each speech. Topic modelling is an unsupervised learning method that attempts to separate a collection of documents in distinct groups. I used the Latent Dirichlet allocation (LDA) readily availble in the topicmodels package to sort the 238 addresses into three classes (k = 3). LDA does allow for overlap in the use of terms in multiple topics, so the LDA outputs each document as a mixture of the classes. 

The two major outputs of the topicmodels LDA approach are word-topic probabilities (beta values), which measure the tendency for a particular term to appear in one topic, with higher values demonstrating higher proclivity. The other value is document-topic probabilities (gamma values) which estimates the number of words in a document that are attributed to each topic. 

In the chart above, we see a clear temporal component associated with the document-topic probabilities. The SOTU between 1790 and 1860 show little mixture in gamma values, indicating strong similarity in the language in the addresses. As the first topic class decreases, the second topic class increases, reaching its peak in 1908. It is worth noting that the third topic class shows little to no change as the other two classes fluctuate. The third topic class does spike at nearly 20% probability in Woodrow Wilson's 1917 address, further analysis might indicate why this speech was more similar to more recent addresses than other addresses from the time or from Wilson himself. Finally, from Nixon in 1970 onward, we see strong consistency of the third topic class, as the earlier document-topic probabilities drop to zero. 

Total Frequency-Inverse Document Frequency:

While topic modelling can help us group documents into similar classes, there are other measures to identify the unique aspects of each address. Inverse Document Frequency (IDF) is a measure of term usage that increases the weight for terms that are used infrequently across a collection of documents and decreases words that appear commonly. When combined with the word's term frequency within each document, we can identify words that are unique to each year's address. 

When you pan across the plot you will see the five terms with the highest TF-IDF value for each document, essentially creating a fingerprint for the address that distinguishes it from all others. Using this measure, we see words related to the War on Terror, such as "terrorist", "Iraq", and "Saddam" characterize all of George W. Bush's addresses, except the first address which was the only one to precede 9/11. 
While Ronald Reagan was the first president to invite acknowledged guests to attend the SOTU in 1982, we do not see the unique guest names until Reagan's 1985 address ("Hale" and "Jean"). It appears that Reagan's addresses before 1985 were dominated by references to spending and the deficit. 

## **Document Simarility**
The [quanteda](https://cran.r-project.org/web/packages/quanteda/quanteda.pdf) package provides a number of document similarity measures. This section outlines three common similarity measures that are implemented in the [shiny app](https://bikeshareviz.shinyapps.io/sotuviz_shiny/). 

**Correlation Similarity**
Correlation similarity is the pearson coefficient between the document terms. You can normalize the input document-feature matrix (dfm) to remove the bias towards longer documents by wrapping the input dfm in dfm_weight(x, "prop"). 

**Jaccard Similarity:**
The Jaccard Similarity Index interprets document similarity as the size of the intersection between two documents and the size of the union of the two. 

![Jaccard Equation](https://github.com/danbernstein/feb/blob/master/public/img/jaccard_eq.png)

It is important to note that the Jaccard Index is bias towards longer documents, so documents with a higher occurrence of a given word will give greater weight to that word, regardless of document length. The Jaccard Index also does not discriminate between words that are common across the corpus and words that are unique to a select few. For that reason, it is common to apply a transformation on the raw term frequency counts, such as TF-IDF, before applying a similarity measure. 


**Cosine Similarity:**
Cosine Similarity takes in documents represented as vectors, through TF-IDF or another transformation, and compares directionality to determine the cosine angle between each pair. A cosine value of one indicates that the document are identical, while a value of zero indicates that the documents share no similarity. When using cosine similarity, all documents are normalized, removing the length bias seen in Jaccard Similarity. This transformation is inherent in the method, so you do not need to apply dfm_weight as you might when using correlation. 

![Cosine Equation](https://github.com/danbernstein/feb/blob/master/public/img/cosinesimil_eq.png)


**Not All Similarity Measures Are Made the Same**
Using raw term usage can yield misleading similarity measures. When you think about the most common words in the english language (i.e., "and", "is"). To get a better measure of document similarity, we can first apply a transformation to weight words that are more unique in each document (a method called Total Frequency-Inverse Document Frequency, or TF-IDF), and then normalize coefficients to remove any bias towards larger values. 

If we look back at the method for calculating each of the similarity measures, we see that they won't all work with the TF-IDF correction. Jaccard Similarity simply measures the fraction of words that are common to two documents. TF-IDF adds weight to unique words and lessens the weight of words common across the corpus, but it will not change the actual presence of words in the documents. When we compare the Jaccard measures for similarity matrices using TF vs. TF-IDF, we still there is little difference between the outputs. 

Alternatively, both cosine and correlation similarity show much lower measures when we move from TF to TF-IDF data because the measures are based on more complex weighting, rather than the simple presence or absence of words. 

<div>
    <img src="https://github.com/danbernstein/feb/blob/master/public/img/blogs/text-mining/correlation_difference.png" />
</div>
