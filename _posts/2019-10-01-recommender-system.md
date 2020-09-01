---
title: A Review for Recommender Systems
categories:
 - ML
 - DL
 - RL
 - RS
tags:
---

## Introduction
Recommender system is very important in today's digital world, which helps the user to find their interested products, movies, musics in varied web platform. The main idea and goal of recommender systems is to utilize these various sources of data to infer customer interests. There are various techniques and approaches which have been implemented for recommender systems. 

About the architecture of recommender systems, it is a difficult problem in industry. There are two main components called recall layer and ranking layer. Besides, there are also some other small components like re-ranking for diversity and advertisement.
![vm_directory @8x]({{ "/assets/images/post/rs.png" | absolute_url }})

The first is recall layer, which is mainly based on part of the user's characteristics. From the massive inventory, quickly retrieve a small part of the user's potentially interesting items, and then hand it over to the ranking layer. The ranking layer can incorporate more features and use complex models for accuracy. Make personalized recommendations. Recall emphasizes fast, ranking emphasizes accuracy.

## Recall Layer Methods(Memory-based Collaborative Filtering,Model-based collaborative filtering, DNN)

The traditional standard recall structure is generally a multi-way recall, as shown in the figure above. If we divide the call loops according to whether there are user personalization factors, it can be divided into two categories: one is call loops without personalization factors, such as the recall of popular products or popular articles or materials with high historical click-through rates; the other is A category is a recall loop that contains personalized factors, such as user interest tag recall. How should we view the calling circuit that contains individual factors? Actually, you can look at it this way, you can think of a call loop as the sorting result of a single-feature model sorting. This means that a road recall can be regarded as the sorting result of a sorting model, but this sorting model only uses one feature on the user side and the item side. For example, tag recall is actually a single-feature sorting result sorted by user interest tags and item tags; another example is collaborative recall, which can be regarded as a sorting result of only two features including UID and ItemID... and so on.

There are two main method used in recommender systems, including Collaborative Filtering Recommendation and Content-based Recommendation. 

For CF, there are two categories called user-based method and item-based method. In e-commerce, the size of user is usually larger than the size of item(e.g. Amazon.com had 29 million users and millions of catalog items in 2003 [Linden et al. 2003]). But it has Data sparsity and Cold start problem, also need user to rate the items without any other information about items. 

And another method called Content-based, to analyse the similarity between the items by TF-IDF.But some type of material is hard to definied the content except text.

## Ranking Layer Methods(lr、gbdt、fm、ffm、dnn、widedeep、dcn、deepfm)

The earliest LR model is basically manual feature engineering and manual feature combination, which is simple and effective but time-consuming and laborious; then it is developed to the high-order feature combination automation of LR+GBDT and the second-order feature combination automation of the FM model; The introduction of the DNN model, the purely simple DNN model is actually based on the embedding of the features of the FM model, adding several layers of MLP hidden layers to perform implicit feature nonlinear automatic combination. The so-called implicit means that there is no explicit network structure to directly model the second-order and third-order combinations of features. It is only through MLP to allow different features to interact. As for how the interaction occurs, how to perform the feature combination, who It is also not clear that this is the function of the implicit feature combination of the MLP structure. Of course, due to the introduction of MLP, the nonlinear relationship between features will also be considered during feature combination. The typical modeling of high-order feature combinations, such as Deep& Cross, XDeepFM model, etc., is such an idea.

## References
1. Covington P, Adams J, Sargin E. Deep neural networks for youtube recommendations[C]//Proceedings of the 10th ACM conference on recommender systems. 2016: 191-198.
2. Wang R, Fu B, Fu G, et al. Deep & cross network for ad click predictions[M]//Proceedings of the ADKDD'17. 2017: 1-7.
3. Guo H, Tang R, Ye Y, et al. Deepfm: An end-to-end wide & deep learning framework for CTR prediction[J]. arXiv preprint arXiv:1804.04950, 2018.
4. Lian J, Zhou X, Zhang F, et al. xdeepfm: Combining explicit and implicit feature interactions for recommender systems[C]//Proceedings of the 24th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. 2018: 1754-1763.

