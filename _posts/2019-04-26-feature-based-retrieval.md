---
layout: post
title: "Feature Based Retrieval - Machine Learning in Search Engine"
subtitle: "Learning Notes of CMU Course 11-642 Search Engine"
date: 2019-04-26 23:45:13 -0400
background: '/img/bg-index.jpg'
---

## A brief introduction to machine learning
    - Two topic we focus:
        - What are examples and labels? 
        - How are examples encoded as feature vectors?

##### LeToR overview
    - Features we use: Pieces of evidences
        ○ BM25 & Indro for (d, q) for different fields
        ○ VSM (q, d)
        ○ PageRank(d)
        ○ Length_URL(d)
        ○ Difficulty(q)
        ○ Coordination Match (number of query terms that match doc d)
        ○ Spam score
        ○ Query term density (how query terms are separated, are they dense)
        ○ Time on site
        ○ Page per session (of a website that client view)
        ○ Bounce rate (how often people leave instantly)
        ○ Website security (https>http)
    - In our task, score is not important, but the rank of document
    - We are predicting lables
    - Efficiency problem - it's expensive!
        ○ First 10 features has most importance, adding more doesn't help so much
    - LeToR don't find docs, it just rerank it. 
    - P11 Effectiveness & efficiency trade off! 
    
##### LeToR framework
        ○ Y vector size = m = number of docs per query
        ○ H(x) = score/label
        ○ N = query numbers of training data
        ○ More complicated the retrieval labels, the easier the learning problem for ML.
        
##### 3 approaches to LeToR
    - Similarity:
        ○ Model is always predicting score
        ○ All models are all using a feature vector x of (d, q)
    - Difference:
        ○ How they train the data (what they produce is the same)
        ○ Diff type of training data

##### Pointwise approach
    - Data: individual documents
    - Model: (label can be category or score) SVM or LR
    - Approaches:
        ○ Cranfield evaluation data to train a model
        ○ Given a new query q:
            § Ran a ranker to get init documents(BM25)
            § For each (d, q), get vec x, get label h(x)
            § Rerank docs
    - Pros: easy to implement
    - Drawback: less accurate
        ○ Firstly, predict categories is more difficult than predicting scores
            § Categories are ordinal, but most algorithm ignore that
            § Different types of error should have different costs
            § (To solve this, we use regression (0,1,2) -> (0, 1, 9)
        ○ Secondly, predict score is difficult
            § Try to predict the number, even scores are arbitrary, only ranking matters, but regression model is focusing on the score instead of ordering!
            § Scores are unimportant, but in regression, if change the scores, the model is totally different.
        ○ Also, focus on one document makes learned model fragile = wrong goal, wrong kind of error.
        ○ (To solve this, we use pairwise!)

##### Pairwise approaches
    - H(x_1 - x_2) > 0
    - Change into normal classification/regression tasks
    - Cons:
        ○ The ranking position is ignored! (as Pointwise!)
        ○ The noise can make a fuss! One doc is making a lot of training samples!
        ○ The distribution of R and NR docs might cause unbalance of training data - sample to make balance

##### Listwise approaches
    - Procs: consider position in ranking
    - Training to optimize NDCG/MAP on a ranking, goal is to maximize them, directly aligned the model with ranking target


## Summary

##### Datasets with Features 
    - Which is relevant which is not relevant to query q
    - How is IDF calculated - query irrelevant

##### Feature and Influences Summary
    - SiteRank, Url click count, url dwell time

##### Experiment results for 3 methods
    - RankSVM is similar to ListNet
    - RankSVM and RankBoost are the best
    - Pairwise is more used today, but eventually would be beated by Listwise
    
    

