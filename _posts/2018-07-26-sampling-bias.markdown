---
layout: post
title:  "Sampling bias"
date:   2018-07-26 21:46:30 +0300
categories: sampling machinelearning scikitlearn python
---
## Sampling bias

Considering purely random sampling methods are generally fine if your dataset is large enough (especially relative to the number of attributes), but if it is not, you run the risk of introducing a significant sampling bias. When a survey company decides to call 1,000 people to ask them a few questions, they don’t just pick 1,000 people randomly in a phone booth. They try to ensure that these 1,000 people are representative of the whole population. For example, the US population is composed of 51.3% female and 48.7% male, so a well-conducted survey in the US would try to maintain this ratio in the sample: 513 female and 487 male. This is called stratified sampling: the population is divided into homogeneous subgroups called strata, and the right number of instances is sampled from each stratum to guarantee that the test set is representative of the overall population. If they used purely random sampling, there would be about 12% chance of sampling a skewed test set with either less than 49% female or more than 54% female. Either way, the survey results would be significantly biased.

As an example imagine we have a dataframe by having a category column that we know is a very important attribute to preict the outcome. We may want to ensure that the test set is representative of he various categories of the category column in the whole dataframe. We can split the dataset into stratums by having same proportion of category column as shown below:

{% highlight python %}
from sklearn.model_selection import StratifiedShuffleSplit

split = StratifiedShuffleSplit(n_splits=1, test_size=0.2, random_state=42)

for train_index, test_index in split.split(df, df["discrete_column"]):
  strat_train_set = df.loc[train_index]
  strat_test_set = df.loc[test_index]
{% endhighlight %}
