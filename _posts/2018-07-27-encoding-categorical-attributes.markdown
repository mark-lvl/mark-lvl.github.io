---
layout: post
title:  "Encoding Categorical Attributes for Machine Learning Algorithms"
date:   2018-07-27 21:46:30 +0300
categories: encoder machinelearning scikit
---
## Encoding Categorical Attributes for Machine Learning
Most Machine learning algorithms prefer to work with numbers. So if we have some attributes by having categorical values, we can convert them by using following methods.

 - Using Scikit-learn **LabelEncoder** transformer

{% highlight python %}
from sklearn.preprocessing import LabelEncoder
encoder = LabelEncoder()

# Subset attributes with categorical values
# Consider we have to select only one column for each transformation
dataset_cat = dataset_df["CAT_ATTRIBUTE"]

# transform categorical values to numerical
dataset_cat_encoded = encoder.fit_transform(dataset_cat)

# Look at the result
print(dataset_cat_encoded)

# Look at the mapping classes
print(encoder.classes_)
{% endhighlight %}

**NOTE**: One issue with this representation is that machine learning algorithms will assume that two nearby values are more similar than two distant values. In many datasets that wouldn't be the case, to fix this issue, a common solution is to create *one binary attribute per category* which called **one-hot encoding**.

 - Using Scikit-learn **OneHotEncoder** to convert integer categorical values  into one-hot vectors.

{% highlight python %}
from sklearn.preprocessing import OneHotEncoder
encoder = OneHotEncoder()

# 1. Converting encoded dataset from the outcome of LabelEncoder.
# 2. Note that fit_transform() expects a 2D array, but dataset_cat_encoded is a 1D array, so we need to reshape it.
# 3. NumPy’s reshape() function allows one dimension to be –1, which
# means “unspecified”: the value is inferred from the length of the
# array and the remaining dimensions.
dataset_cat_1hot = encoder.fit_transform(dataset_cat_encoded.reshape(-1,1))

# The result will be a sparse matrix only stores the location of the nonzero elements.
dataset_cat_1hot

# In order to convert it to a Numpy array just call toarray()
dataset_cat_1hot.toarray()

{% endhighlight %}

- Third method is actually the combination of both above transformations (from text categories to integer categories and then from integer categories to one-hot vectors) in one shot by using Scikit-learn **LabelBinarizer** class:

{% highlight python %}
from sklearn.preprocessing import LabelBinarizer
encoder = LabelBinarizer()

dataset_cat_1hot = encoder.fit_transform(dataset_cat)

# Note that LabelBinarizer returns a dense NumPy array by default
dataset_cat_1hot

{% endhighlight %}
