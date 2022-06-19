---
layout: post
title: "Bridgebots ML: Creating Training Data"
date: 2022-05-18 11:05:09 -0700
categories: bridge bridgebots machine-learning
---

In my [last post](/posts/Bridgebots-ML-Introduction) I introduced the work I have been doing to train ML models to predict the holdings or actions of players in the auction during a bridge hand. Today I will be doing a deeper dive into how raw bridge data can be prepared to enable model training, and how we can leverage Tensorflow components to build an efficient training pipeline.

## Building the Dataset
As I mentioned in the introduction post, I recursively crawled the [The Vugraph Project](https://www.sarantakos.com/bridge/vugraph.html) to download hundreds of LIN files from prestigious tournaments. We can then use a [simple script](TODO link to parse vugraph) to combine these records into a single Python list and then persist that data in binary format using [Pickle](https://docs.python.org/3/library/pickle.html). In this process, we also combine records for the same deal from across files, so each unique deal is saved with all records of boards where that deal was played.

## Splitting the Dataset
Data can be split in many ways for ML, such as [k-fold validation](TODO LINK), but for now I just used a straightforward 80% train, 10% validation, 10% test split. In the [create_data_splits](TODO link) script I use a weighted random number generator to split the data.

## Tensorflow SequenceExample
There are a few ways to feed data into Tensorflow. I was really impressed by their [Dataset API](TODO link), and wanted to use this project as an opportunity to learn it better. TF Dataset works epically well with Tensorflow [Example](TODO link) and [SequenceExample](TODO link) data structures. These are built on top of [Protobuff](TODO link), which is a cross-platform/cross-language library for efficient binary storage and transfer of data. Since we are focusing on the auction, our data is sequential in nature, and therefore well suited to use SequenceExample.

A SequenceExample has two sets of components: context features and sequence features. Context features are information provided to the model which does not vary across the sequence. For bridge, the vulnerability of the players is a constant throughout the auction, so it can be treated as a context feature. Sequence features are temporal in nature, i.e. there are a sequence of inputs for the model. For bridge, the auction is clearly a sequence feature.

There are some pieces of information which could be represented as either a context or sequence feature. For example, the cards held by the players do not change during the auction, so 