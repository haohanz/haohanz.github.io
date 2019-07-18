---
layout: post
title: "Authority Metrics: PageRank, TSPR and HITS"
subtitle: "Rearning Notes for CMU Course 11642 Search Engine"
date: 2019-05-01 23:45:13 -0400
background: '/img/bg-post.jpg'
---

# PageRank + TSPR + HITS

Why authority metrics
- To decide which website to trust more, 
- to give some content-irrelevant metrics.


Properties for PR
- Usually for journals or papers for citation analysis
- Query independent, cause mistakes
- The computation itself makes it easy to manipulate
    - Pages farms or page exchanges
    - The new page
    - The sink page
    - The links inside one page


How PR works?
- Random walk, voting procedure
    - Larger d, harder to converge!
    - Init:
$$PageRank(p_i) = \frac{1}{\|C\|}$$
    - Iteration: 
$$PageRank(p_k)=\frac{1−d}{\|C\|} + d\sum_{p_j∈Inlinks(p_i)}\frac{PageRank(p_j)}{\|OutLinks(p_j)\|}$$

What is TSPR (Topic sensitive!)
- TSPR for a topic of d: only teleportation over the same topic of d
- The score of final:
\begin{equation}
TSPR_q (d)= \sum_{i∈I_q} w_i \times TSPR_i (d) 
\end{equation}


What is HITS (HyperLink included Topic Search)
- Only include a top n's result's expansion of inLink and outLink
- For each iteration, init all = 1  
\begin{equation}
     H(p_k )= \sum_{p_j \in OutLink(p_k)} A(p_j )
\end{equation}
\begin{equation}
    A(p_k )= \sum_{p_j \in InLink(p_k )} H(p_j )
\end{equation}
- Normalize after each iteration


Properties of HITS
- Pros
    - The base set has a strong query specific focus
    - Very efficient (set is small)
    - Score calculated at query time!
- Cons
    - It's easier to make spam page than it is for PR
        - It's easy to create a page with high Hub score
    - Its run-time cost is higher than PR
- Applications
    - Find a community
    - Find expert within a community


