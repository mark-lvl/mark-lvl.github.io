---
layout: post
title:  "Custom Transformers in Scikit-Learn"
date:   2018-07-30 10:00:00 +0300
categories: transformer machinelearning scikit
---
Although Scikit-Learn provides many useful transformers, sometimes we need to write our own for tasks such as custom cleanup operations or combining attributes. What we want is a transformer which works seamlessly with Scikit-Learn functionalities (such as pipelines) and as we already know Scikit-Learn relies on duck typing, we need to implement three methods:
 * fit()
 * transform()
 * fit_transform()

Simply by adding *TransformerMixin* as a base class we get fit_transform without implement manually. Moreover, if we add *BaseEstimator* as a base class (without any \*args or \**kargs) we will get *get_params()* and *set_params()* that will be useful for automatic hyperparameter tuning.

{% highlight python %}
from sklearn.base import BaseEstimator, TransformerMixin

class CustomTransformers(BaseEstimator, TransformerMixin):
  def __init__(self, some_hyperparameter = 'BETTER_SET_DEFAULT_VALUE'):
    self.some_hyperparameter = some_hyperparameter
  def fit(self, y=None):
    return self # basically nothing to do here

  def transform(self, X, y=None):
    # doing transform operation on X attributes
    # by consider hyperparameter to control the flow of modifications
    # e.g. if some_hyperparameter do something else do something else
    # and finally return X
    return X

customTrans = CustomTransformers(some_hyperparameter = 'SOMETHING')
transformed_dataset_df = customTrans.transform(dataset_df)
{% endhighlight %}

**NOTE**: Generally hyperparameters will allow us to easily find out wether adding new transformed attribute to dataset helps the Machine Learning algorithms or not. This approach is helpful to find great combination between attributes just by defining various hyperparameter for each combination and try them.
