---
layout: post
title: "Bridgebots ML: Introduction"
date: 2022-04-09 11:05:09 -0700
categories: bridge bridgebots
---

I have been interested in applying Machine Learning to Bridge data for a long time; it was one of the original impetuses for building Bridgebots. Over the past few months I have made slow progress towards that goal. Today I would like to make a three-part introduction to that work. The first section will focus on Machine Learning, the second on Bridge, and the third on applying ML to Bridge.

## Machine Learning
Machine Learning is a huge field that encompasess many different concepts and techniques. At its core, it is the application of statistics to find patterns in data. Most ML can be put in one of three categories:
1. Supervised Learning
2. Unsupervised Learning
3. Reinforcement Learning

#### Supervised Learning
[Supervised Learning](https://en.wikipedia.org/wiki/Supervised_learning) describes tasks for which we have labeled data, and we want to train a model to predict those labels. In Bridge data, we might give the model the auction and ask it to predict the number of High Card Points (HCP) held by each player. From historical matches we have lots of examples we can give the model to learn this relationship. When the model makes an incorrect prediction, we use the label from the training data to instruct the model to update itself to be more accurate in the future. This is accomplished through techniques like [Backpropagation](https://en.wikipedia.org/wiki/Backpropagation) and [Stochastic Gradient Descent](https://en.wikipedia.org/wiki/Stochastic_gradient_descent).

#### Unsupervised Learning
[Unsupervised Learning](https://en.wikipedia.org/wiki/Unsupervised_learning) describes finding patterns in unlabeled data. The most common applications are clustering analysis and anomaly detection. For Bridge, we could cluster bidding systems or partnerships together based on auctions or cardplay. Anomaly detection could be used to find highly unusual bids or plays which could be indicative of cheating. Popular techniques include [k-means Clustering](https://en.wikipedia.org/wiki/K-means_clustering) and [Principal Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis).

#### Reinforcement Learning
[Reinforcement Learning](https://en.wikipedia.org/wiki/Reinforcement_learning) describes the process of teaching an agent how to interact with its environment to maximize reward. It is not supervised learning because the agent is not taught exactly what to do, and it is not unsupervised because the agent is influenced by an outside goal. For Bridge, reinforcement learning could be used to teach an agent to bid or play, similar to the techniques that were used to train [AlphaGo Zero](https://en.wikipedia.org/wiki/AlphaGo_Zero) and [AlphaZero](https://en.wikipedia.org/wiki/AlphaZero) to play Go and Chess at superhuman levels.

### Training a Supervised Learning Model
For my first ML project with Bridgebots I wanted to focus on supervised learning. It is much easier to understand what a model is doing and how it is performing when you have labeled data to check against. The general process for approaching a supervised learning problem looks like this:
1. Collect historical data. More data is better, but having clean (error-free) data is generally even more important.
2. Generate labels that you would like the model to be able to predict, either programmatically or by hand.
3. Split the data into training, validation, and test sets. There are several ways to approach this, but in general the training set is used to teach the model, the validation set is used to verify that the model has learned the general principles of the problem well (not just memorized the training data), and the test set is used as the final measure of the performance of the model.
4. Develop features that will be used as inputs to the model. For bridge this might be a representation of the bids that were made, the cards in a player's hand, the vulnerability, or the players involved.
5. Train a model to predict the target labels and evaluate its performance.
6. Iterate over model architectures and model training parameters to improve model performance.
7. Release a final version of the model which can be used for future predictions.

## Bridge
Bridge is a complicated game, and I am assuming most readers of this blog are familiar with the basics. If you are familiar, then feel free to skip this section. If you are not, check out [this introduction page](https://www.acbl.org/learn/) from the ACBL or [this tutorial on youtube](https://www.youtube.com/watch?v=2IomnCvxWzM) and/or read the below description. Each bridge hand is composed of three sections:

### The Deal
Two partnerships sit in alternating order (each player across from their partner). Thirteen cards are delt to each player. One player is chosen to start the auction, and each team is assigned a vulnerability. A vulnerable team receives more rewards for making their contracts, but also receives a higher penalty for failing. 

### The Auction
Players compete for the right to name the trump suit for the play of the hand (the trump suit will be stronger than any other suit). At their turn, each player may either pass or raise the stakes of the hand (the number of tricks that must be taken to receive the reward). In general, higher contracts pay better rewards, so players try to bid as high as they safely can. The auction continues until one player makes a bid and the other three all pass. While the premise of the auction is fairly straightforward, in practice it can be incredibly complicated. This is because each partnership assigns special meanings to various sequences of bids, creating a language that is used to communicate information about their hands.

### The Play
The team that won the auction will have one of their players (the declarer) make all the decisions for the rest of the hand. The declarer's teammate's hand (the dummy) will be revealed to everyone. This means the declarer has access to perfect information about all their team's assets, while the defenders have imperfect information, but all of their assets are hidden from declarer. In sequence, each player will play a single card to a trick. Each player must follow the suit of the first card led to the trick. The highest card of that suit will win the trick unless a trump card is played, in which case the highest trump will win. The winner of a trick starts the next trick. If the declarer makes the number of tricks they contracted for in the auction, their team receives a positive score. If they fail, the defenders get the plus score.

## Bridge ML

### Gathering Data
Following the roadmap that I laid out above, the first step to training a supervised learning model is to collect data. There are a few different sites that host bridge data, but for this project I decided to use records from [The Vugraph Project](https://www.sarantakos.com/bridge/vugraph.html). It has records from dozens of high level tournaments across many years of bridge. My hope is that if our models can understand this data, then they will be able to handle a wide variety of bridge situations. The data can be downloaded fairly simply with [wget](https://www.gnu.org/software/wget/manual/html_node/index.html) using the [recursive flag](https://www.gnu.org/software/wget/manual/html_node/Recursive-Retrieval-Options.html). The data is in LIN file format; luckily the Bridgebots core library can already handle that.