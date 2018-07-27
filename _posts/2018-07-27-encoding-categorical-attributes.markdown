---
layout: post
title:  "Encoding Categorical Attributes for Machine Learning Algorithms"
date:   2018-07-27 21:46:30 +0300
categories: encoder machinelearning scikit
---
## Encoding Categorical Attributes for Machine Learning
Most Machine learning algorithms prefer to work with numbers. So if we have some attributes by having categorical values, we can convert them by using following methods.

1. Using Scikit-learn **LabelEncoder** transformer

{% highlight python %}
from sklearn.preprocessing import LabelEncoder
encoder = LabelEncoder()

# Subset attributes with categorical values
# Consider we have to select only one column for each transformation
dataset_cat = dataset_df["CAT_ATTRIBUTE"]

# transform categorical values to numerical
dataframe_cat_encoded = encoder.fit_transform(dataset_cat)

# Look at the result
print(dataframe_cat_encoded)

# Look at the mapping classes
print(encoder.classes_)
{% endhighlight %}
