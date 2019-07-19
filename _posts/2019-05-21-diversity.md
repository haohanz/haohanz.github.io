---
layout: post
title: "Diversity: MMR, xQuAD and PM2"
subtitle: "Learning Notes for CMU Course 11642 Search Engine"
date: 2019-05-21 23:45:13 -0400
background: '/img/posts/12.jpeg'
---


## Table of Contents
- <a href="#motivation">Motivation</a>
- <a href="#metrics">Diversity Metrics</a>
    - P@k
    - P-IA@k
    - Alpha-NDCG
- <a href="#implicit">Implicit Methods</a>
    - MMR
    - LeToR
    - Relational-LeToR
- <a href="#explicit">Explicit Methods</a>
    - <a href="#xquad">xQuAD</a>
    - <a href="#pm2">PM2</a>
- <a href="#summary">Summary</a>

<div id="motivation"/>
<hr>

## Motivation
- Documents are not always independent
- The query has different intents (most short queries)
- There can be multiple tasks for the same interpretation
- User intents are usually not clear
- Find a balance between robustness and relevance
- The traditional IR tend to find textual similarity between doc and query, 
which tend to lead to a query focus mainly on the most probable intent, 
which only good for some users. 
The target is to maximize the expected SAT in each position. 


<div id="metrics"/>
<hr>

## Diversity Metrics
- $$P@k$$: Only R or NR, ignore the diversity of ranking
- $$P-IA@k$$:
\begin{equation}
     P−IA@k= \sum_{q_i} \Pr(q_i│q) P@k_{q_i}
\end{equation}
Tend to give higher priority to documents that satisfy multiple intents
- Alpha-NDCG:
\begin{equation}
      NDCG@k=Z_k \sum_{i=1}^k G@k \times D@k
\end{equation}
    - $$D@k = \frac{1}{log_2⁡(1+k)} $$, same as before
    - $$G@k = 2^{R(k)}+1 $$, original gain
    - $$G@k = \sum_{q_i} R(d_k, q_i)  (1−\alpha)^{r_{q_i,k−1} } $$
    , intent aware gain
    - $$R(d_k, q_i) = $$ Relevance of $d_k$ to $q_i$
    - $$r_{q_i, k−1} = \sum_{j=1}^{k−1} R(d_j, q_i)$$, the intent redundancy of $d_k$
    - $$(1−\alpha)^{r_{q_i,k−1} } $$, the discount for covering an intent already




<div id="implicit"/>
<hr>

## Implicit Methods
- Maximum Marginal Relevance (MMR)

$$\begin{eqnarray} 
MMR &= arg \max_{d_i \in R/S}⁡\lambda \Pr(d_i, q) \\
&− (1−\lambda) \max_{d_j \in S}⁡Similarity(d_i, d_j )
\end{eqnarray}$$

- Similarity not symmetric
    - Use VSM, calculate "anything to anything" similarity
    - KLD: anything to anything, but not symmetric
    - Use Jensen-Shannon Divergence - anything to anything
\begin{equation}
JS(x||y) = \frac{1}{2} KL(x||M) + \frac{1}{2} KL(y||M)
\end{equation}
Where
\begin{equation}
M=\frac{x+y}{2}
\end{equation}
- LeToR for divergence
    - Approach: fit LeToR model directly to optimize a diversity measure metric, use diversification based training data
    - Fail reason:
        - There's no diversification features
        - There's nothing about doc-to-doc similarity
        - There's nothing about sub-intents
        - => relevance based features cannot produce a diverse ranking
        - The training data confuses the training model
- Relational-LeToR
    - Use both relevance and relationship features
    - Use similarity, supervised model of MMR
    - Features for similarity
        - Has links pointing to each other
        - URL: from same website or not
        - Text similarity - KLD
        - Topic similarity - topic cosine between docs' topic distribution in topic model
        - Category similarity - overlap of docs' categories in a predefined ontology
    - Diff with MMR:
        - MMR: each position is independent of previous ones
        - Relational-LeToR: the positions are not independent
    - Same with MMR:
        - Greedy: docs are picked from top to bottom sequentially. 
        - Relational - ListMLE:
\begin{equation}
\Pr(d_i│S_i, w) = \frac {exp(w_r x_i + w_d y_i) } {\sum_{x_j∈S_i} exp⁡(w_r x_j+w_d y_j)}
\end{equation}
\begin{equation}
L(R│w)= − \prod \Pr(d_i |w)
\end{equation}

    - Diff with ListMLE:
        - Has a weight for diversity and feature y for diversity added
        - Cannot calclulate the h(x) all of a time then re rank, has to build while rank, because the y diversity features are built on the previous ranked docs.
    - Performance is the best
    - Each feature could be either min or max or avg

- Implicit Methods Pros and Cons
    - Don't favor any intent, all intents treated equally
    - Don't need prior knowledge

<div id="explicit"/>
<hr>

## Explicit Methods
- Query intent discovery
    - Use search log to get intent, others are trick and open provlems
    - Intent 
        - Ambiguous
        - Faceted - related interpretations of a query
    - Suggested queries - higher precision
    - Related queries - higher recall
<div id="xquad"/>
- xQuAD (Equations)
    - Key idea: (Same as MRR, but the formula has different symbols)
        - To reduce the redundancy as much as possible
        - To cover the intent as much as possible
    - Features:
        - The docs cover more intents are tend to be preferred. (Just like MMR, just like $P-IA@k$)
        - The intents are treated equally (Just like MMR)
        - The final rank prefer the mediocre docs that covers more documents, then prefer the top relevant document, documents that doesn't rank high and doesn't cover a lot of intents would not be promoted.
        - One query has 7-10 intents, thus time complexity is high
    - Equation:
\begin{equation}
d^∗ = arg⁡\max_{d \in R/S}⁡(1−\lambda) \Pr(d│q) + \lambda \Pr(d, \bar{S}|q)
\end{equation}
\begin{equation}
\Pr(d, \bar{S}│q) = \sum_{q_i \in Q} [\Pr(q_i│q) \Pr(d│q_i)  \prod_{d_j \in S} (1 − \Pr (d_j|q_i))]
\end{equation}
    - Why suggest queries are more effective than related queries
        - The cons of xQuAD: the intents are treated equally, thus when there's a high recall algorithm that provided original rank, the discovering query intetns would find many rare or unpopular intents, which users doesn't need.
<div id="pm2"/>
- PM2
    - Key Idea: The number of documents for each subtopic should be proportional to each subtopic's popularity
    - Greedy
        - At each rank r, select the query intent $q_i$ that must be covered next to maintain proportional coverage of intents in the ranking
        - Select a document that covers the $q_i$, and might also cover other query intents.
    - The update process
        - Get doc that has the max
\begin{equation}
\lambda qt[i^∗] \Pr(d_i│q_{i^∗}) + (1 − \lambda) \Pr(d_{j\ others} | q_{j\ others})
\end{equation}
        - Update the rank after each iteration
\begin{equation}
s_i += \frac {\Pr(d^∗│q)} {\sum_{q_j \in Q} \Pr(d^∗ |q_j)     }
\end{equation}

<div id="summary"/>
<hr>
## Summary
##### Contrast: xQuAD vs PM2:
- $\lambda$ is on the weight of new intent to cover
- xQuAD wouldn't care about the none mediocro and none covered a lot docs
- Both use uniform weight

##### Implicit vs Explicit Diversification Algorithms
- Explicit is more effective than MMR (better metric), but having the query intent resource is difficult, the suggestions might not suit your task
- R-LeToR works the best for supervision




