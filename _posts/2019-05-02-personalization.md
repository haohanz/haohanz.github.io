---
layout: post
title: "Personalization for Search Engine"
subtitle: "Learning Notes for CMU Course 11642 Search Engine"
date: 2019-05-02 23:45:13 -0400
background: '/img/posts/08.jpeg'
---
### Topic-based personailization
$$ \beta \Pr (d│q)+(1−\beta) \Pr_{cat} (d│q, u) $$

$$ \Pr_{cat} (d|q, u)=  \sum_c \Pr (c│d) \Pr (c|u) $$
- Metric: MRR - improved a lot on the ambiguous queries
- Main idea:
    - consider the user's and doc's topic-category matches
    - A personalized SE consider
        - Relevance
        - The query-independent value of scores
        - The topic is interested in or not

### Long-short term personalization
### 10 features for atypical discovery
1. Query length
2. Query length divergence
3. SAT Reading Level
4. SAT Reading Level Divergence
5. Topic divergence
6. Ratio of noun
7. Verb ratio divergence
8. Adjective ratio divergence
9. Longest query position
10. Question ratio

Divergence = distance of current session to historical vocabs/topic categories/features

