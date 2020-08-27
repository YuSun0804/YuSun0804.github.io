---
title: A Review for Recommender Systems in Industry
description: review recommender system industry
categories:
 - ML
 - DL
 - RL
 - RS
tags:
---

## Introduction
For the recommender systems, it has been wild used in recent years. The AI model varied from Collaborative Filtering to Reinforcement Learning. 

## Architecture(Online & Offline)
About the architecture of recommender systems, it is a difficult problem in industry. There are two main components called recall layer and ranking layer.



## Recall Layer Methods(Memory-based Collaborative Filtering,Model-based collaborative filtering)

There are two main method used in recommender systems, including Collaborative Filtering Recommendation and Content-based Recommendation. 

For CF, there are two categories called user-based method and item-based method. In e-commerce, the size of user is usually larger than the size of item(e.g. Amazon.com had 29 million users and millions of catalog items in 2003 [Linden et al. 2003]). But it has Data sparsity and Cold start problem, also need user to rate the items without any other information about items. 

And another method called Content-based, to analyse the similarity between the items by TF-IDF.But some type of material is hard to definied the content except text.

## Ranking Layer Methods(lr、gbdt、fm、ffm、dnn、widedeep、dcn、deepfm)

In terms of machine learning, logistic regression firstly is used for classfication task and we can treat the action of recommend as 1, while not recommend as 0. After that, tree model like GDBT and random tree are used for recommender systems.

# References
https://static.googleusercontent.com/media/research.google.com/ru//pubs/archive/45530.pdf
https://zhuanlan.zhihu.com/p/58160982
https://zhuanlan.zhihu.com/p/100019681
https://juejin.im/post/5c8a0c205188257ded10e4a0

