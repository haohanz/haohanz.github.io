---
layout: post
title: "Search Log Analysis"
subtitle: "Learning Notes of CMU Course 11-642 Search Engine"
date: 2019-05-01 23:45:13 -0400
background: '/img/posts/11.jpeg'
---
#### Table of Contents
- <a href="#motivation">Motivation</a>
- <a href="#session">Sessions</a>
- <a href="#suggestion">Query Suggestions</a>
- <a href="#intent">Query Intents</a>
- <a href="#click">Click Models</a>
- <a href="#within">Within-Session Learning</a>

<div id="#motivation"/>
<hr>
#### Why Search Log Analysis
- Query Frequency, a power law (same shape as Zipf's law)
\begin{equation}
    Frequency(q) = K \times Rank(q)^{−\alpha}
\end{equation}
    - Where K = constant, positive
    - Alpha about 2.4 for Excite query log
    - Most unique queries occur very rarely, a small percentage of the unique queries are very common
    - The overlap of frequent queries goes down as time growths
    - 50% of the unique queries, 20% of the total queries are never seen before
- Web search behavior -> 3 dimensions
    - What
    - Who
    - How - session characteristics
        - Features:
            - The variation of the docs you click on 
            - % of queries with low/high click entropy
                - The session length, queries/session (entropy can be evaluated by the variation of the documents)
- "Who"
    - Representing users:
        - For users, cluster them based on queries
        - Firstly, build a pseudo document for each user, the docs contains the categories of the top 10 docs for all the queries in their search log
        - Use Jenson-Shannon Divergence or cosine correlation to find the similarity users.
        - Based on Yahoo Directory categories
    - Informational Users (how)
        - Less single-click sessions
        - More Non-navigational queries
        - Query suggestions are used more
        - Well-educated
        - Above-average income
    - Navigational Users (how)
        - Facebook, YouTube, mostly representative of the entire population
        - More single-click
        - More navigational
        - Less query suggestions
    - Transactional Users (how)
        - Low income and education
        - Diverse clicks
        - Less interaction
        - Has overlap with navigational users
        - Search for shopping, adult content, gaming
    - High level educated users
        - Use query suggestions more but click on them less
        - Tend to have shorter sessions
        - More: submit tail queries (more precise)
    - => Query log analysis is useful for personalization strategies designs

<div id="session"/>
<hr>
#### How to Segment Search Logs into Sessions
- Methods
    - Using time: the gap between two queries are less than delta, then they are in the same session.
    - Using the overlap terms (CT): iff there's overlap between two quries, they are in the same session
    - Using the rewrite classes (RT): the common reformulation patterns(add, delete, replace…) 
    - Both b and c would cause high precision and low recall
    - Other features to segment a log
        - The JSD of top 10 results of q1 and q2
        - The Yahoo page category overlap of top10 results of q1 and q2
        - The co-occurrence of queries in a query log
        - Etc.
- Differentiate the goal and mission
    - Information need == goal == sub tasks
    - Mission == a set of related information needs, which is higher-level or an extended information need (e.g. how to go to PIT)
    - Boundary for mission and boundary for goal: the queries must be sequential!
    - Same task: they might not be sequential, just two (a pair of) separate queries are from the same mission or goal
    - The trained time for mission is longer than that of goals
    - Method of detecting "same task" for pairs of related queries - classification based features
        - Temporal
            - Delta time, are sequential or not 
        - Edit distance (the distance between two strings)
            - Character and token-based metrics (e.g. CT, RT)
        - The occurrence of <q1, q2> pair in a larger query log
        - The Web search result: the cosine distance (JSD) of top 50 search results fore each query 
        - To decide the if <q1, q2> are related queries (same for mission/goal) Use two methods
            - Heuristic (only time delta, CT, RT)
            - A classification-based approach - use the features mentioned above to do classification.
            - The classification is somewhat a little better. The heuristic is not bad!
        - The "same task" task has higher accuracy than adjacent query task "boundary task" because the "same task" has more negative samples.
        - The "Edit Distance" is very important, cosine distance is ok, time along is primitive, but with combination, it works.





<div id="suggestion"/>
<hr>
#### Query Suggestions
- Source
    - Pseudo documents
        - For many queries - almost any query can use this method
    - Co-occurrence statistics
        - For reasonably frequent queries - can solve the mid-frequent and the common queries. 

- Pseudo documents
    - Build the pseudo document, success is the last query in a session
    - Use the query to search for the corpus of pseudo documents

- Co-occurrence Statistics
    - Get Set1 and Set2
        - Set1: the most frequent queries with q as a substring
        - Set2: the most frequent queries that follows q
    - Get Score of the union of Set1 and set2 queries q_s
        $$ Score(q_s) = \frac {Count(q_s)+ \lambda_1 }{N_1+\lambda_1  } \times \frac {count_follows (q, q_s )+λ_2 } {N_2+λ_2 }

<div id="intent"/>
<hr>
#### Query Intents
- Find the most popular intent of a query
    - Steps
        - Expand
        - Filter to improve precision
        - Cluster - maybe textbased distance/similarities
        - Random walk to estimate popularity
        - Name → to the query with highest weight from random walk
    - Reformulation patterns and the click information can be combined  to identify the common intents for an ambiguouos query.

<div id="click"/>
<hr>
#### Click Models
- Models
    - DCTR
    - PBM
    - RCM
    - RCTR
    - CM

- Use of click models
    - Generate artificial clicks
    - Generate artificial relevance r(DCTR)

<div id="within"/>
<hr>
#### Within-Session Learning
- Key ideas
- Procedural vs declarative
- Features for user expertise and behavior
    - The display time (short)
    - The query complexity (high)
    - The entropy (few topic, less entropy)
    - The Focus (category)
    - The domain count (number of unique domains in the SERPs)
- The result of sessions:
    - domain count would increase, the declarative > procedural domain count
    - Domain count and query complexity: declarative > procedural
    - Features to identify the page is a declarative page
        - The text len, sentence len, term len
        - Cover of query terms in title, distribution of query terms
        - Par of speech distribution (POS)
        - The page complexity, query complexity, reading difficulty metrics


