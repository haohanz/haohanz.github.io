---
layout: post
title: "Neural Ranking: DSMM, DRMM, K-NRM, Conv-KNRM"
subtitle: "Learning Notes for CMU Course 11642 Search Engine"
date: 2019-05-21 23:45:13 -0400
background: '/img/posts/10.jpeg'
---
##### Why user Deep Learning in Search Engine?
More features, more complex combination of weight and features, less hand-crafted features and functions.
- <a href="#dsmm"> DSMM </a>
- <a href="#drmm"> DRMM </a>
- <a href="#knrm"> K-NRM </a>
- <a href="#convknrm"> Conv-KNRM </a>
- <a href="#summary"> Summary </a>

<div id="dsmm"/>
<hr>
## DSMM
##### Motivation
- Representation: solve query vocabulary mismatch, represent text in concept based space
- Loss: $$ L(*) = −\log\prod{\Pr(D^+|Q)}$$
, where $$\Pr(D│Q) = \frac{exp⁡(\lambda R(Q, D))}   {  (\sum_{D^′ \in D} exp⁡(\lambda R(Q,D^′))   } $$
- Use 500k term -> 30k trigram hashing vector
    - Robust to the out of vocabulary problem
    - Low collision rate: 22 out of 500k terms
- The hashed representation captures the orthographic (continuous term representation) similarity
    - Spelling 
    - Hyphenation
    - Misspell
    - Case
    - But not conceptual similarity
- After projection, use cosine similarity between the 128 len query qtf representation and document tf representation. 
- How training data are selected:
    - Use query log, The relevant are clicked, non-relevant are sampled from top N
    - Use pairwise training
        - Can't use pointwise/listwise due to accuracy
    - Minimize lose function: only consider the positive samples
    - Different from our target: 
        - If there are many pairs, it tend to give higher score to clicked documents.
- Drawbacks:
    - Better than BM25, this is not surprising, supervised > unsupervised
    - Just re-rank top 15 documents, not robust
    - No idf or document length considered, only tf considered. Perhaps not needed to re-rank the top 15 from a stronger ranker, perhaps not needed to re-rank titles.

<div id="drmm"/>
<hr>
## DRMM
##### Continuous Representation - Both DSMM, DRMM (word2vec), Why DRMM
- Previous continuous representation models ignore the "exact match" strong signal.

##### Key idea of DRMM & Steps

- Firstly use Word2Vec to present different source of terms: query terms separated with document terms.
- Structure:
    - -> Embedding layer 
    - -> Pyramid pooling (Binning) 
    - -> Feed forward matching to learn the interaction between (q_i, d_i)
    - -> term gating using idf 
    - -> sum scores of all the terms (linear combination)
    - $$ g_i =\frac{ exp⁡(w \cdot idf(q_i))  }  {  \sum_{j=1}^n exp⁡(w \cdot idf(q_j))   }      $$, where i is the index for q_i  in a query q.
    - (Where does the idf table come from?)
- Training:
    - Pairwise training with hinge loss, minimize it
    - $$ L(q, d^+, d^−)= \max⁡(0, 1−s(q, d^+ )+s(q, d^−)) $$
- Evaluation:
    - Better than rankSVM
    - Should be compared to query expansion? Should? Because used word2vec which input a lot of texts!

##### What's the similarity and difference for DRMM and old retrieval models?
- Same:
    - Use tf, idf, log(tf)
    - Can tell (represent) exact match of query term and document term
    - Use bag-of-words model
    - Summation of scores for each query term
- Diff:
    - Use "bin" value for matches of different quality
    - Exact and soft match of query terms and document terms are all represented by continuous representations (word2vec)
    - It learns how to combine evidences (non-linear) during the feed forward matching phase. Non-linear combination of match values of different quality (through feed forward matching after pyramid pooling)
    - Get interaction patterns for matching using e.g. CNN, hierarchical patterns.

<div id="knrm"/>
<hr>
## K-NRM

##### Motivation
- More convenient for DRMM - use end-to-end training to train all parts of the model - use a layer to replace the pyramid pooling layer, which is not differentiable. 
- Use search log data - more than TREC and more noises. 
- Use as much part as the DRMM

##### What's same with DRMM?
- Text representation of query and docs

##### What's different with DRMM?
- Softly count soft-match term frequencies (kernel pooling instead of pyramid pooling)
- Embeddings tuned for ad-hoc search - need more training data.
- Consistent training (end-to-end)
- No idf consider like DRMM (no need for the only top 15 docs from a strong engine result)
- Stronger soft-match features (more noise tolerable)
- A simpler model with exact-match features - one exact match score for a (q, d) pair.


##### Architecture of K-NRM:
- -> Embedding layer 
- -> translation layer (M_(n×m)) matrix
- -> kernel pooling 
    - each bin has a kernel that center at the bin's center, 11 bins->11 kernels!
    - Each q_i  term has a soft-tf score per bin, which is the sum of the kernel on all thet docs that are in this range.
    - Thus each q_i  has 11 scores after the kernel pooling. Just the same as DRMM's pyramid pooling output
    - Log sum each 11 length vector for all the query terms, finally get 1×11 output. (k=11), there's only one feature for each kernel!
        - Why log sum? Panelize the query terms with few matches. 
- -> Feed Forward Matching (1 layer NN) (pairwise training)
- -> Final score


##### Training of K-NRM
- Exactly the same with DRMM


##### Performance of K-NRM: 
- Conv-KNRM > K-NRM > DRMM > RankSVM > DSMM

##### Why K-NRM is better? The source of effectiveness?
- Embeddings trained end-to-end for ad-hoc search
    - The tuning of word embeddings makes a lot of pairs changed (58%)
    - Word2Vec make it has conceptual similarities for pairs
    - The Words are de-couples e.g. cats vs dogs
    - The rare words are punished by the log summation
    - The soft match patterns are discovered: (pdf, reader)
    - The matching strength are changed.
- Kernel pooling is more effective than other pooling: combine matches in a score range.


<div id="convknrm"/>
<hr>
## Conv-KNRM
##### Motivation: 
- To support n-gram matching between different lengths.

##### The source of effectiveness for Conv-KNRM:
- (N-grams)
    - A CNN to form n-grams
    - Fewer parameters to train than discrete n-grams
- (Cross-matching n-grams of different lengths)
    - Another use of soft matching

<div id="summary"/>
<hr>
## Representation Based vs. Interactive Based Model
- Similarity
    - Identify local matches between two pieces of text - cosine similarity of term vectors
    - Use continuous representation of text (embedding)
- Difference
    - Interactive: Learn interaction patterns for matching
        - Often hierarchical patterns
        - E.g. CNN
    - Interactive: Time complexity is high, each term has to compare with each document term, usually used for re-ranking
    - Interactive: consider both positive and negative examples using hinge lost of pairwise training data, but representative (DSMM) only use the positive ones, the goal of which is different from our target. 
    - Interactive: Integration of exact-match and soft-match signals,
        - Older systems only have discrete (IR) or continuous(DSMM), not both. 
        - The combination is effective and robust
    - The trained data is required more in interactive based models, the static embeddings need little data.




