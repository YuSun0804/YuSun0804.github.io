---
title: a Review for Recommender Systems in Industry
description: review recommender system industry
categories:
 - ML
 - DL
 - RL
 - AI
tags:
---

## Introduction
For the recommender systems, it has been wild used in recent years. The AI model varied from Collaborative Filtering to Reinforcement Learning. 

## Architecture(Online & Offline)
About the architecture of recommender systems, it is a difficult problem in industry. There are two main components called recall layer and ranking layer.

## Recall Layer Methods
There are two main method used in recommender systems, including Collaborative Filtering Recommendation and Content-based Recommendation. 

For CF, there are two categories called user-based method and item-based method. In e-commerce, the size of user is usually larger than the size of item(e.g. Amazon.com had 29 million users and millions of catalog items in 2003 [Linden et al. 2003]). But it has Data sparsity and Cold start problem, also need user to rate the items without any other information about items. 

And another method called Content-based, to analyse the similarity between the items by TF-IDF.

## Ranking Layer Methods
In terms of machine learning, logistic regression firstly is used for classfication task and we can treat the action of recommend as 1, while not recommend as 0. After that, tree model like GDBT and random tree are used for recommender systems. 


