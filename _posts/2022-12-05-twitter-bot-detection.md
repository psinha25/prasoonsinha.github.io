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
- `statuses_count`: The number of Tweets (including retweets) issued by the user
- `friends_count`: The number of users this account is following (AKA their “followings”)
- `favourites_count`: The number of Tweets this user has liked in the account’s lifetime
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
