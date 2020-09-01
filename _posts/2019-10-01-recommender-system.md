---
title: A Review for Recommender Systems
description: This bolg is going to illustrate development of recommender system in industry.
categories:
 - ML
 - DL
 - RL
 - RS
tags:
---

## Introduction
Recommender system is very important in today's digital world, which helps the user to find their interested products, movies, musics in varied web platform. The main idea and goal of recommender systems is to utilize these various sources of data to infer customer interests. There are various techniques and approaches which have been implemented for recommender systems. 

## Architecture
About the architecture of recommender systems, it is a difficult problem in industry. There are two main components called recall layer and ranking layer. 
![vm_directory @8x]({{ "/assets/images/post/rs.png" | absolute_url }})

The first is recall layer, which is mainly based on part of the user's characteristics. From the massive inventory, quickly retrieve a small part of the user's potentially interesting items, and then hand it over to the ranking layer. The ranking layer can incorporate more features and use complex models for accuracy. Make personalized recommendations. Recall emphasizes fast, ranking emphasizes accuracy.

#### Recall Layer Methods(Memory-based Collaborative Filtering,Model-based collaborative filtering, DNN)

There are two main method used in recommender systems, including Collaborative Filtering Recommendation and Content-based Recommendation. 

For CF, there are two categories called user-based method and item-based method. In e-commerce, the size of user is usually larger than the size of item(e.g. Amazon.com had 29 million users and millions of catalog items in 2003 [Linden et al. 2003]). But it has Data sparsity and Cold start problem, also need user to rate the items without any other information about items. 

And another method called Content-based, to analyse the similarity between the items by TF-IDF.But some type of material is hard to definied the content except text.

#### Ranking Layer Methods(lr、gbdt、fm、ffm、dnn、widedeep、dcn、deepfm)

In terms of machine learning, logistic regression firstly is used for classfication task and we can treat the action of recommend as 1, while not recommend as 0. After that, tree model like GDBT and random tree are used for recommender systems.

## References
1. Covington P, Adams J, Sargin E. Deep neural networks for youtube recommendations[C]//Proceedings of the 10th ACM conference on recommender systems. 2016: 191-198.

https://zhuanlan.zhihu.com/p/58160982
https://zhuanlan.zhihu.com/p/100019681
https://juejin.im/post/5c8a0c205188257ded10e4a0

