---
layout: post
title: "Bridgebots ML: Introduction"
date:   2022-06-18 23:00:09 -0700
categories: bridge bridgebots machine-learning
---

I have been interested in applying Machine Learning to Bridge data for a long time; it was one of the original impetuses for building Bridgebots. Over the past few months I have made slow progress towards that goal. Today I would like to make a three-part introduction to that work. The first section will focus on Machine Learning, the second on Bridge, and the third on applying ML to Bridge.

## Machine Learning
Machine Learning is a huge field that encompasses many different concepts and techniques. At its core, it is the application of statistics to find patterns in data. Most ML can be categorized as one of the following:
1. Supervised Learning
2. Unsupervised Learning
3. Reinforcement Learning

#### Supervised Learning
[Supervised Learning](https://en.wikipedia.org/wiki/Supervised_learning) describes tasks for which we have labeled data, and we want to train a model to predict those labels. In Bridge data, we might give the model the auction and ask it to predict the number of High Card Points (HCP) held by each player. From historical matches we have lots of examples we can give the model to learn this relationship. When the model makes an incorrect prediction, we use the label from the training data to instruct the model to update itself to be more accurate in the future. This is accomplished through techniques like [Backpropagation](https://en.wikipedia.org/wiki/Backpropagation) and [Stochastic Gradient Descent](https://en.wikipedia.org/wiki/Stochastic_gradient_descent).

#### Unsupervised Learning
[Unsupervised Learning](https://en.wikipedia.org/wiki/Unsupervised_learning) describes finding patterns in unlabeled data. The most common applications are clustering analysis and anomaly detection. For Bridge, we could cluster bidding systems or partnerships together based on auctions or cardplay. Anomaly detection could be used to find highly unusual bids or plays which could be indicative of cheating. Popular techniques include [k-means Clustering](https://en.wikipedia.org/wiki/K-means_clustering) and [Principal Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis).

#### Reinforcement Learning
[Reinforcement Learning](https://en.wikipedia.org/wiki/Reinforcement_learning) describes the process of teaching an agent how to interact with its environment to maximize reward. It is not supervised learning because the agent is not taught exactly what to do, and it is not unsupervised because the agent is influenced by an outside goal. For Bridge, Reinforcement Learning could be used to teach an agent to bid or play, similar to the techniques that were used to train [AlphaGo Zero](https://en.wikipedia.org/wiki/AlphaGo_Zero) and [AlphaZero](https://en.wikipedia.org/wiki/AlphaZero) to play Go and Chess at superhuman levels.

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
![bridge ML logo](/assets/bridge/bridgebots/bridge_nn_with_shared_connections_small.png)

### Gathering Data
Following the roadmap that I laid out above, the first step to training a supervised learning model is to collect data. There are a few different sites that host bridge data, but for this project I decided to use records from [The Vugraph Project](https://www.sarantakos.com/bridge/vugraph.html). It has records from dozens of high level tournaments across many years of bridge. My hope is that if our models can understand this data, then they will be able to handle a wide variety of bridge situations. The data can be downloaded fairly simply with [wget](https://www.gnu.org/software/wget/manual/html_node/index.html) using the [recursive flag](https://www.gnu.org/software/wget/manual/html_node/Recursive-Retrieval-Options.html). The data is in LIN file format; luckily the [Bridgebots core library](https://github.com/forrestrice/bridge-bots/tree/master/bridgebots) can already [handle that](/posts/Introducting-Bridgebots-Part-3). I created two datasets - one which can contain the same deal multiple times, and one which has a unique bidding/play record for each deal.

I have not yet merged the code used in this blog post into the main Bridgebots repository, but you can find it on this [branch](https://github.com/forrestrice/bridge-bots/tree/vugraph-project-sequence-learning) or see the diff in this [pull request](https://github.com/forrestrice/bridge-bots/pull/14).

### Labeling
Now that we have downloaded a few thousand deals, we need to decide which targets we want our model to predict. For my first project, I decided to focus on the auction. At each player's turn to act, I want to predict three things:
1. The next bid.
2. The number of high card points held by each player.
3. The shape of each player's hand.

I chose these targets because they are important components of playing bridge, and because it is straightforward to programmatically calculate all three of them (unlike e.g. evaluating the wisdom of a given bid). I will discuss the code used to produce these labels in a future post.

### Train/Validation/Test Split
For now, I am using a simple and standard split of 80% training, 10% validation, and 10% test.

### Model Features
This is another subject where I will go into more detail in the future, but the basic features I included in the model were:
1. The auction so far.
2. The vulnerability.
3. The position of the current player to act (dealer, dealer + 1, etc).
4. The cards held by the current player encoded as 52 bit flags.

Features I wrote but have not yet included because I am not sure of their validity in the data yet:
1. Which bids were alerted.
2. Which bids were explained.

I have also given some thought about how to represent a partnership's bidding system to the model. My eventual hope is to use [embeddings](https://en.wikipedia.org/wiki/Embedding) to allow the model to take specific information about players or systems into account. Alternatively, we could devise a way to pass structured information with the bid (e.g. 3+ clubs, 10+ HCP for an opening of 1C for a partnership playing standard, 16+ HCP for a partnership playing precision). For now, all these models have no knowledge of the players or systems, so they treat every auction the same.

### Model Training and Iteration
I am still in the process of training these models. There are many different techniques and architectures that can be used. The auction is a sequence of actions, so we can use models that specialize in handling sequential data like [LSTMs](https://en.wikipedia.org/wiki/Long_short-term_memory) and [Transformers](https://en.wikipedia.org/wiki/Transformer_(machine_learning_model)) to make predictions based on the history so far. 

For the task of predicting the next bid, we can use classification metrics to measure the performance of our model. These include [categorical crossentropy](https://en.wikipedia.org/wiki/Cross_entropy), [precision](https://en.wikipedia.org/wiki/Precision_and_recall), and [recall](https://en.wikipedia.org/wiki/Precision_and_recall) but there are [many more](https://en.wikipedia.org/wiki/Evaluation_of_binary_classifiers). 

For the task of predicting the players' shapes or HCP, we can use a regression based model. This will predict some average value (e.g. median or mean depending on the loss function used to train), but we could also augment the model to predict other aspects of the target distribution (e.g. standard deviation could tell us how wide of a range of HCP is expected for a bid). 

For this work I used [Tensorflow](https://www.tensorflow.org/), a popular Machine Learning library developed by Google.

### Releasing a Model
I spent some time building a saving/loading mechanism that will bundle the features needed by the model with the model internals (weights + structure). A user can download these models and run them locally using the Bridgebots Python library. These models are far from perfect, but as you will see below, they already see that have some promise. Even when they are not provided with any system/partnership information, they make fairly reasonable predictions about the auction and holdings of the players. Like the rest of Bridgebots, the models and the code used to train them will be released under the [MIT Open Source License](https://en.wikipedia.org/wiki/MIT_License), which means they will be freely available for anyone to use or modify.

## Examples
I have published the first generation of these models to a [public GCP storage bucket](https://console.cloud.google.com/storage/browser/bridgebots-public;tab=objects?prefix=&forceOnObjectsSortingFiltering=false), and I built an example [Google Colab notebook](https://colab.research.google.com/drive/1DoW2cVw8IBS5DoOgajcVd-jD4p785pIP?usp=sharing) which shows how they can be loaded and run.

### Loading the Model
The models can be loaded by installing the bridgebots-sequence package.
```shell
pip install bridgebots-sequence
```
Then an "Inference Engine" can be created from the saved model data:
```python
from pathlib import Path
from bridgebots_sequence.inference import BiddingInferenceEngine


hcp_engine = BiddingInferenceEngine(
    model_path=Path("./bridgebots-public/bid_learn/gen_1/hcp/"),
)
```

### Running the Model
Each model needs inputs in order to make predictions. This information should be provided in the form of Bridgebots representations of cardinal directions, cards, and bids. For example:
```python
from bridgebots import Card, Direction

dealer = Direction.SOUTH
dealer_vulnerable = True
dealer_opp_vulnerable = False
player_holding = [
    Card.from_str(s) for s in ["S7", "S2", "HA", "HJ", "H9", "H2", "DK", "DJ", "D2", "CA", "CQ", "CJ", "C4"]
]
bidding_record = ["1S", "PASS", "2C", "PASS", "2H", "PASS", "3H", "PASS", "3S", "PASS", "4D", "X", "PASS", "PASS"]
bidding_metadata = []

print(
    "Predicted hcp:\n",
    hcp_engine.predict(
        dealer, dealer_vulnerable, dealer_opp_vulnerable, player_holding, bidding_record, bidding_metadata
    ),
)
```
```text
Predicted hcp:
 {SOUTH: 12.845368, WEST: 4.947876, NORTH: 15.906226, EAST: 6.1790986}
```

### Evaluating the Models
At the risk of repeating myself, the technical details of the model evaluation is something I will write up in more detail later. However, I will provide some examples of the model predictions below, and I have set up a [Google Colab notebook](https://colab.research.google.com/drive/1DoW2cVw8IBS5DoOgajcVd-jD4p785pIP?usp=sharing) so that anybody can try the models for themselves. For these examples, I chose to look at [this segment](https://www.bridgebase.com/tools/handviewer.html?bbo=y&linurl=https://www.bridgebase.com/tools/vugraph_linfetch.php?id=74272 ) of the [2022 d'Orsi](http://www.worldbridge.org/competitions/wbf-championships/world-bridge-teams-championships/dorsi-seniors-trophy/) between the USA and India. For each of these hands, we will "replace" one of the players with the models and see how they would evaluate the hand. That means that only 13 of the 52 cards are provided as input.

#### Board 24
![board 24](/assets/bridge/bridgebots/dot_24.png)
The model evaluates the board from the perspective of Meckstroth after the auction reaches 2SX:
```text
NORTH - HCP:12.056150436401367, Shape:[1.8675611 3.1290896 3.823617  4.12866  ]
EAST - HCP:7.969760894775391, Shape:[3.3415089 3.3108985 3.2528265 3.0557857]
SOUTH - HCP:12.726337432861328, Shape:[2.9363637 3.3428063 3.112618  3.529826 ]
WEST - HCP:7.195767402648926, Shape:[4.8077374 3.1735537 2.7231622 2.2366753]
Predicted bid: 3C
Predicted bid probabilities:[('3C', 0.27392504), ('2NT', 0.2338383), ('3D', 0.18260154), ('PASS', 0.10459597), ('3H', 0.06501402)]
```
The model did well understanding that West held spades after they opened a multi 2D, and that North would bid 3C (27%) or 2NT (23%), both of which were suggested by the commentators. The model struggled with the fact that East's 2S actually showed a preference for hearts, and overestimated South's strength. 

### Board 31
![board 31](/assets/bridge/bridgebots/dot_31.png)
Let's look at what the model thought was going on at each of Meckstroth's calls.

After `["1S", "PASS"]`:
```text
NORTH - HCP:15.839217185974121, Shape:[2.021296  3.9838076 3.0657048 4.007261 ]
EAST - HCP:5.896975994110107, Shape:[2.8044546 3.5252647 3.7001972 3.08169  ]
SOUTH - HCP:12.090693473815918, Shape:[5.4067245 2.3355904 2.9286385 2.4316034]
WEST - HCP:6.003118991851807, Shape:[2.8464196 3.2721624 3.4069438 3.5730052]
Predicted bid: 2C
Predicted bid probabilities:[('2C', 0.4598667), ('2NT', 0.2520364), ('2H', 0.12931933), ('2D', 0.07926174)]
```
After `["1S", "PASS", "2C", "PASS", "2H", "PASS"]`:
```text
NORTH - HCP:16.015335083007812, Shape:[2.015837  3.9580772 3.071818  4.0097213]
EAST - HCP:5.516820907592773, Shape:[3.0574307 3.0101004 3.9873986 3.0293846]
SOUTH - HCP:12.106705665588379, Shape:[4.9549217 3.3470654 2.5047426 2.2745743]
WEST - HCP:6.300617694854736, Shape:[3.0348947 2.7714062 3.5119736 3.7505522]
Predicted bid: 2NT
Predicted bid probabilities:[('2NT', 0.40432608), ('3H', 0.20415133), ('4H', 0.09077729), ('3C', 0.05689908), ('2S', 0.053402733)]
```
After `["1S", "PASS", "2C", "PASS", "2H", "PASS", "3H", "PASS", "3S", "PASS"]`:
```text
NORTH - HCP:16.078683853149414, Shape:[1.9772431 4.0498543 2.9730036 4.0501876]
EAST - HCP:5.4062113761901855, Shape:[3.059058  2.6944509 4.2516003 3.0805616]
SOUTH - HCP:12.723127365112305, Shape:[4.9007063 3.614962  2.5420392 2.0281525]
WEST - HCP:5.835963726043701, Shape:[3.1310077 2.736113  3.3029883 3.9128149]
Predicted bid: 4C
Predicted bid probabilities:[('4C', 0.50220186), ('3NT', 0.23680453), ('4NT', 0.08573343), ('4D', 0.06225779), ('4H', 0.06137136)]
```
After `["1S", "PASS", "2C", "PASS", "2H", "PASS", "3H", "PASS", "3S", "PASS", "4D", "X", "PASS", "PASS"]`
```text
NORTH - HCP:15.906225204467773, Shape:[1.9650984 4.0142136 2.962     4.0986676]
EAST - HCP:6.1790995597839355, Shape:[2.93841   2.4142017 4.4957485 3.230924 ]
SOUTH - HCP:12.84537124633789, Shape:[5.094196  3.867588  2.5129228 1.612994 ]
WEST - HCP:4.947877407073975, Shape:[3.0642023 2.8014474 3.0946152 4.124716 ]
Predicted bid: XX
Predicted bid probabilities:[('XX', 0.9760938)]
```

The model quickly recognized an opening spade bid in terms of shape and strength, and predicted the 2/1 auction. It adjusted its belief about South's shape after the 2H rebid, but probably not as much as we would want, since this almost always shows 4+. It also missed that North would raise hearts (could be related to the incorrect prediction about shape), and overemphasized the likelihood of redoubling at Meckstroth's last call. Overall we can see the model could still be improved a lot, but its predictions are at least in the right neighborhood for many aspects of the hand.

#### Board 18
![board 18](/assets/bridge/bridgebots/dot_18.png)
Lets do the same step by step evaluation for Saha on Board 18.

After `["PASS", "1C"]`
```text
NORTH - HCP:8.551470756530762, Shape:[3.9040601 3.1166744 3.1397586 2.8878965]
EAST - HCP:6.108532428741455, Shape:[3.74385   2.9586966 3.2413125 3.0909529]
SOUTH - HCP:14.163474082946777, Shape:[3.381772  3.0403666 2.552261  4.079942 ]
WEST - HCP:10.894444465637207, Shape:[2.0130272 3.9442337 4.1041923 2.9768863]
Predicted bid: PASS
Predicted bid probabilities:[('PASS', 0.42284918), ('1H', 0.30451122), ('1D', 0.15629105), ('X', 0.06527666)]
```

After `['PASS', '1C', 'PASS', '3C', 'X', 'PASS']`
```text
NORTH - HCP:7.928995132446289, Shape:[2.6428466 2.4678926 3.2113147 4.795836 ]
EAST - HCP:7.6490960121154785, Shape:[4.353987  3.5895529 3.3248572 1.8434293]
SOUTH - HCP:13.27055835723877, Shape:[4.0041056 3.0925465 2.5056815 3.5044522]
WEST - HCP:10.901622772216797, Shape:[2.0748663 3.9804525 4.0774617 2.9409585]
Predicted bid: 3H
Predicted bid probabilities:[('3H', 0.46870205), ('4H', 0.3312904), ('3D', 0.06510229)]
```
Nice job on this one! The model correctly assessed that Saha would pass and then bid 3H. It also understood that North had long clubs with less than invitational values, and that East had length in the other suits. It likely predicted that East's spades were longer because we told it West's hand, which has four hearts, four diamonds, and only two spades. The final HCP predictions are just about right as well!

## Wrapping Up
Thanks for following along! I hope you now find the possibilities of applying ML to Bridge as exciting as I do. If you are interested in how these models were trained or even in training some models of your own, stay tuned - I have a lot more to show you!