---
title: 👻 Bot or Not? 
layout: post
tags: [twitter, machine learning, bot detection]
mathjax: true
social-share: false
readtime: true
---

**Introduction**

Twitter is an extremely popular social media platform, containing millions of daily active users (although given the recent controversy surrounding the recent change in management and mass amount of layoffs by CEO Elon Musk, this remains to be seen). As a free and easy-to-use platform, users engage in international news, politics, sports, memes, and a variety of other topics. However, many users do not realize they often engage with Twitter bots, which often tweet malicious links, interfere in elections [1, 2], spread fake news and propaganda [3], and attempt to steal user data. Bots are simply automated programs that attempt to co-exist with human users by imitating genuine users. Given the unquestionable importance of user safety and maintaining the integrity and respect of social media platforms, many researchers have focused their attention in creating scalable solutions for identifying Twitter Bots. In this blog post, we do the same by exploring and improving on the recent research advances in Twitter Bot detection. 

**TwiBot-20 Data Set**
	
We use a subset of the TwiBot-20 Data Set [4], a comprehensive twitter bot detection benchmark. Presented in the International Conference of Information, Knowledge, and Management 2020, this is the first publicly available data set of Twitter users that includes follower relationships. The authors of this data set leveraged the Twitter API to collect three modals of user information: semantic, property, and neighborhood information. To create a diverse user data set (unlike many of its predecessors), a breath-first algorithm was used to select users from the Twittersphere (graph of Twitter users, where nodes are users and edge represents a connection between two users). Below are some of the unique features this dataset contains relative to the three modals collected: 
- Semantic: user tweets, retweets, and replies
- Property: follower count, following count (friend count), verification status
- Neighborhood: users followed and following (in the form of Twitter ID numbers)

**Data Exploration**

In order to obtain domain knowledge in the differences in engagement with the social media platform between bots and humans, we asked ourselves 4 questions we hoped to answer with the data we had. 

1. Is there a difference in the amount of tweeting between bots and humans?
2. Are humans good at noticing if an account is a bot?
3. Do the tweets of bots receive lots of engagement from Twitter users?
4. Are there noticeable differences in the twitter profile between a bot and a human?

Containing over 11.8K samples in our data set, about 56% of the samples are bots and 44% are humans. Below are box plots summarizing the number of tweets and retweets across our dataset between bots and humans. 

![Number of Tweets Breakdown](/static/img/tweets-breakdown.jpg)
![Number of Retweets Breakdown](/static/img/retweets-breakdown.jpg)

There is a noticeable difference in the spread of the number of tweets and retweets between bots and users. We notice that most humans tend to have between 185-205 tweets, with outliers spanning all the way down to 0 tweets. Bots on the other hand have a wider range between 143 and 200 tweets. Hence, to answer question 1: *although the spread of tweets and retweets is larger for bots than humans, there is not a noticeable difference in the amount of tweeting between the two*. 

Below are box plots summarizing the spread in followers, friends (following), and favorites (total number of likes the user’s tweets/retweets have received). 


![Summary of Followers Count](/static/img/follower-count-summary.jpg)
![Summary of Friends Count](/static/img/friend-count-summary.jpg)
![Summary of Favorites Count](/static/img/favorites-count-summary.jpg)

It’s evident that the number of followers a bot has tends to be drastically lower than the number of followers of a human. Meanwhile, bots have a much wider spread in the number of people they follow compared to humans. Hence, this suggests that *humans are capable of detecting whether accounts are bots and do not follow these accounts*. However, noticeably the spread in favorites count between humans and bots is shockingly similar, and a bot is the sample in this dataset with the highest number of favorites. Thus, although humans might be able to tell that an account is a bot (maybe because the account has several of the same tweets), *it is more difficult to tell if a tweet is from a bot account*. 

Finally, we examined whether there are differences in the profile makeup between bot and human accounts. In particular, we looked at whether bots tend to have empty descriptions, locations, and/or urls compared to humans. We also compared if either group clearly uses the default profile image more than others. Below are our findings: 

![Profile Comparison](/static/img/profile-comparison.jpg)

As we expected, more bot profiles than human profiles are incomplete (i.e., have empty descriptions, locations, urls, or use the default profile image). *However, the differences between the two types of Twitter users is minimal across all the categories*. This suggests that bot algorithms are quite sophisticated today and motivates the need for more advanced techniques than simple heuristics to detect whether an account is a twitter bot or not. Thus, we present three different models in the rest of this blog post. 

Current work in Twitter Bot Detection mainly falls into three categories: Feature-based methods, Text-based methods, and Graph-based methods. Insights from each of these methods are discussed in the next few sections.

**Feature-Based Method**

Feature-based methods mainly focus on feature engineering with profile information and employing  conventional classification algorithms for bot detection. We used the following features from the TwiBot-20 dataset for training the models. Few of the features were used directly while few features were derived from the features present in the dataset. The following features were used directly from the dataset:

- `listed_count`: The number of public lists that this user is a member of
- `followers_count`: The number of followers this account currently has
- `statuses_count`: The number of tweets (including retweets) issued by the user
- `friends_count`: The number of users this account is following (AKA their “followings”)
- `favorites_count`: The number of tweets this user has liked in the account’s lifetime
- `verified`: When true, indicates that the user has a verified account.
- `default_profile`: When true, indicates that the user has not altered the theme or background of their user profile

The following features were derived from other features in the dataset:

- `profile_image_present`: Derived from the profile_image_url feature which is a HTTPS-based URL pointing to the user’s profile image
- `screen_name_length`: Length of the screen name feature which is a handle, or alias that this user identifies themselves with.
- `screen_name_digits`: Number of digits in the screen name
- `screen_name_entropy`: Entropy of the screen name feature. Entropy is a measure of the randomness of data and this is particularly useful for detecting bots
- `name_length`: Length of the name feature, which is the name of the user, as they’ve defined it
- `name_digit`: Number of digits in the name
- `name_entropy`: Entropy of the name
- `description_length`: Length of the description feature, which is the description of the profile as provided by the user
- `description_digit`: Number of digits in the name
- `description_entropy`: Entropy of the name
- `location_present`: If there is a user-defined location for this account’s profile

The below heatmap shows how the features are correlated among themselves and also with the label:

![Feature Based Heatmap](/static/img/feature_based_heatmap.png)

From the correlation map, we find that the label is highly negatively correlated with the `verified` information. This makes sense as verified profiles are likely to be humans rather than bots. There also exists a high positive correlation between the followers and the listed count. This possibly means that, more the number of followers, more a user is likely to be listed. 

We next explored various binary classification models such as XGBoost Classifier, AdaBoost Classifier, LightGBM Classifier, CatBoost Classifier and RandomForest Classifier and performed hyperparameter tuning to improve the respective base models. The model performance was evaluated in terms of accuracy, precision, recall, f1_score and AUC. Table 1 summarizes the results on both the validation set and the test set. It can be seen that the Random Forest Classifier performs the best in terms of accuracy among all the other models on the test set while Catboost Classifier works best on the validation data. It was interesting to see that manually experimentation on selecting and dropping certain features had a significant impact on the model performance. These decisions were influenced by the insights from the correlation table. Overall, all models achieved about 81% accuracy

![Feature Based Importances Table 1](/static/img/feature_based_table1.png)

The feature importances as obtained from training the xgboost model is as follows:

![Feature Based Importances](/static/img/feature_based_importances.png)

The 4 most important features that the model trains on are `description_entropy`, `tweet count`, `followers`, and `friends`. 

As part of feature selection, we also tried out the forward select based SequentialFeatureSelector (SFS). This technique adds features to form a feature subset in a greedy fashion. At each stage, this estimator chooses the best feature to add based on the cross-validation score of an estimator. Catboost classifier was used as a base model for SFS. This run selected 11 features, namely, `listed`, `followers`, `tweets`, `friends`, `verified`, `screen_name_digits`, `location_present`, `name_entropy`, `screen_name_entropy`, `desc_entropy`, `default_profile`, as the final features to be used for modeling. Using these features, we re-trained the same classifiers that we used before. It gave improvements in validation accuracy and, but it could not give better results that the best model from the previous experiment of 81.9%.

![Feature Based Importances Table 2](/static/img/feature_based_table2.png)

*Disadvantages of feature-based method:*
While this method can identify the important features that can help in detecting Twitter bot accounts,  the bot operators are increasingly aware of the hand-crafted features and often try to tamper with these features to elude detection. Hence, it is difficult for the feature-based methods to keep up with the competition from the evolving bots. However, this when combined with other text and graph based techniques can make the model more robust.

**Text-Based Method**

Text-based methods utilize natural language processing (NLP) techniques to identify Twitter bots by the user's description and tweets. With the recent advancement in computing capability, large-scale pre-trained language models have shown excellent performance on various NLP tasks, such as language translation, sentiment analysis, summarization, and question answering. Additionally, one can fine-tune the language models on specific problems using limited amounts of training data and still obtain surprisingly good results. In particular, we use RoBERTa-large as our language model from [Hugging Face](https://huggingface.co/roberta-large) and finetune it on the TwiBot-20 dataset. 

RoBERTa [5] (Robustly Optimized BERT Pre-Training Approach) is a variant of BERT [6] (Bidirectional Encoder Representations from Transformers), a popular language processing model developed by researchers at Google. RoBERTa was introduced in 2019 by researchers at Facebook AI and implemented several design improvements based on BERT [6] to make it even more effective at NLP tasks. The model was pretrained with the Masked language modeling (MLM) objective, which allows it to learn an inner representation of the English language that can then be used to extract features useful for downstream tasks. We extract embedding from the user description and user tweets respectively to generate a description embedding and tweet embeddings for each user. Since a user can have up to 200 tweets in the dataset, we average up to 10 tweet embeddings to obtain a single tweet embedding for each user to save preprocessing time. The description embedding, tweets embedding along with labels are then used to train a multilayer perceptron(MLP) head for fine-tuning. The figure below shows the architecture of our model.

![Text-Based Model Architecture](/static/img/text_based_model.jpg)

We implement our model using PyTorch, a popular deep learning library and use [Ray Tune](https://docs.ray.io/en/latest/tune/index.html) for hyper-parameters tuning. The parameters we search for included hidden dimension and dropout rate of the MLP, learning rate, and weight decay value of the Adam optimizer. After obtaining the optimal parameters according to the search, we train the model on the training set and track the validation loss for early stopping. The below figure shows the training and validation loss during each training epoch. We take epoch = 17 as our final model.

![Text Base Loss Plot](/static/img/text_base_loss_plot.jpg)

We again evaluate the performance of our model based accuracy, precision, recall, f1_score and AUC. The result is shown in the table below.

![Text Base Model Score Table](/static/img/text_base_score.jpg)

We can see that the performance of our text-based method (test accuracy: 0.767)  is not as good as the feature-based method (test accuracy: 0.819). This is reasonable as text alone may contain less information compared to the rich engineered features we used in the feature-based methods. Yet, as these two methods utilize distinct information sources, we believe combining the two will boost the model's performance. Therefore, we decided to integrate both methods by stacking.

**Stack Feature-Based and Text-Based Method**

In this short section, we explore stacking our feature-based and text-based method.  As shown in the figure below, we first generate out-of-fold predictions on the training set using our text-based model. 

![Stack Models Architecture](/static/img/stack_oof.jpg)

Next, we add the out-of-fold predictions from our text-based method as a new feature to our feature-based method and train a catboost classifier on the new set of features. The result of this model is shown in the table below.

![Stacked Model Score Table](/static/img/stack_score.jpg)

We can see our stack model significantly outperform both the feature-based and text-based models in all evaluation metrics. In particular, it obtains a test accuracy of 0.853 which is a 0.04 gain over the best feature-based method! Our experiment results support the common wisdom that stacking two independent learners (both in terms of models and features) can yield much better performance!! 

After exploring the property and semantic features of the TwiBot-20 dataset by feature-based and text-based models, there is one last feature left unexplored – the neighborhood feature! In the next section, we will see how the following/follower relationship between users can help us better identify Twitter Bot!! 
