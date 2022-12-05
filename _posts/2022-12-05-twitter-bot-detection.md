---
title: :ghost: Bot or Not? 
# sub-title: Model Serving
layout: post
tags: [twitter, machine learning, bot detection]
mathjax: true
cover-img: /static/img/linux-vm-cover.png
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



