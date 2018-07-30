---
layout: post
title:  "Transformer, Scaling and Pipelines in Scikit-Learn"
date:   2018-07-30 10:00:00 +0300
categories: transformer machinelearning scikit
---
## Custom transformers

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

## Feature Scaling
One of the most important transformations you need to apply to your data is feature scaling. With few exceptions, Machine Learning algorithms don’t perform well when the input numerical attributes have very different scales.

**NOTE**: Scaling the target values is generally not required.

There are two common ways to get all attributes to have the same scale:  
 * min-max scaling (normalization)
 * standardization

 Normalization is quite simple: values are shifted and rescaled so that they end up ranging from 0 to 1. We do this by **subtracting the min value and dividing by the max minus the min**.
 Scikit-Learn provides a transformer called **MinMaxScaler** for this. It has a *feature_range* hyperparameter that lets you change the range if you don’t want 0–1 for some reason.

 Standardization is quite different: first it **subtracts the mean value, and then it divides by the variance** so that the resulting distribution has unit variance. Unlike min-max scaling, standardization does not bound values to a specific range, which may be a problem for some algorithms (e.g., neural networks often expect an input value ranging from 0 to 1). However, **standardization is much less affected by outliers**.
 Scikit-Learn provides a transformer called **StandardScaler** for standardization.

 ## Pipelines
Usaully there are many data transfomation steps that need to be executed in the right order. Scikit-Learn provides the **Pipeline** class to help with such sequences of transformations.

The Pipeline constructor takes a list of name/estimator pairs defining a sequence of steps. All but the last estimator must be transformers (i.e., they must have a fit_transform() method). The names can be anything you like.

Here is a small pipeline for the numerical attributes:

{% highlight python %}
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

# Construct a pipeline by having 3 transformers
num_pipeline = Pipeline([
  # Filling NAs by using imputer in all attributes
  ('imputer', Imputer(strategy="median")),
  # Applying some custom transformer on attributes
  ('custom_attributes', CustomTransformer()),
  # Applying standardization scaling on attributes
  ('std_scaler', StandardScaler()), ])

# Applying the pipeline on dataset_df (filtered for numerical attribute)
transformed_dataset_df = num_pipeline.fit_transform(dataset_df)
{% endhighlight %}

What we've seen so far was applicable only on numerical attributes but what if we need to transform both numerical and categorical attributes simultaneously? Fortunately, Scikit-Learn provides a **FeatureUnion** class for this. What we need to do is to provide datasets with specific column types to the pipelines and pass all the pipelines to FeatureUnion constructor.

{% highlight python %}
from sklearn.pipeline import FeatureUnion

# list of numerical attributes
num_attribs
# list of categorical attributes
cat_attribs

# Defining numerical attributes pipeline
num_pipeline = Pipeline([
  # Filtering dataset only for numerical attributes
  ('selector', DataFrameSelector(num_attribs)),
  # A bunch of tranformers
  ('imputer', Imputer(strategy="median")),
  ('custom_attributes', CustomTransformer()),
  ('std_scaler', StandardScaler()),
  ])

# Defining categorical attributes pipeline
cat_pipeline = Pipeline([
  # Filtering dataset only for categorical attributes
  ('selector', DataFrameSelector(cat_attribs)),
  # one-hot transformer for categorical attributes
  ('label_binarizer', LabelBinarizer()),
  ])

full_pipeline = FeatureUnion(transformer_list=[ ("num_pipeline", num_pipeline),
("cat_pipeline", cat_pipeline),
])

transformed_dataset_df = full_pipeline.fit_transform(dataset_df)
{% endhighlight %}

As we've just seen in above snippet, Each subpipeline starts with a **selector** transformer: it simply transforms the data by selecting the desired attributes (numerical or categorical), dropping the rest, and converting the resulting DataFrame to a *NumPy* array. There is nothing in Scikit-Learn to handle Pandas DataFrames, 20 so we need to write a simple custom transformer for this task:

{% highlight python %}
from sklearn.base import BaseEstimator, TransformerMixin

class DataFrameSelector(BaseEstimator, TransformerMixin):
  def __init__(self, attribute_names):
    self.attribute_names = attribute_names
  def fit(self, X, y=None):
    return self
  def transform(self, X):
    return X[self.attribute_names].values
{% endhighlight %}
