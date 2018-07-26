---
layout: post
title:  "Data snooping bias"
date:   2018-07-26 21:46:30 +0300
categories: sampling
---
## Data snooping bias

In data analysis by having a quick glance at the data, surely you should learn a whole lot more about it before you decide what algorithms to use, right? This is true, but your brain is an amazing pattern detection system, which means that it is highly prone to overfitting: if you look at the test set, you may stumble upon some seemingly interesting pattern in the test data that leads you to select a particular kind of Machine Learning model. When you estimate the generalization error using the test set, your estimate will be too optimistic and you will launch a system that will not perform as well as expected. This is called data snooping bias.
