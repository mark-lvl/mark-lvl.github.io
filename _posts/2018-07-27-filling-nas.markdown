---
layout: post
title:  "Filling NAs"
date:   2018-07-26 21:46:30 +0300
categories: datacleaning
---
## Filling NAs
Most Machine Learning algorithms cannot work with missing features, in order to fix them we have three options:

* Getting rid of the rows with na in the particular attribute
* Getting rid of the whole attribute in the dataset
* Set the values to some values (zero, mean, median, etc.)

{% highlight python %}
dataset_df.dropna(subset=["COLUMN_WITH_NA"]) # option 1 dataset_df.drop("COLUMN_WITH_NA", axis=1) # option 2
median = dataset_df["COLUMN_WITH_NA"].median()  dataset_df["COLUMN_WITH_NA"].fillna(median) # option 3
{% endhighlight %}

Scikit-Learn provides a handy class to take care of missing values: Imputer. Here is how to use it. First, you need to create an Imputer instance, specifying that you want to replace each attribute’s missing values with the median of that attribute:

{% highlight python %}
from sklearn.preprocessing import Imputer
imputer = Imputer(strategy="median")

# Since median can only be computed on numerical attributes, make sure
# the dataset is the subset of numerical attributes then call the fit()
# method on the dataset.
imputer.fit(dataset_df)

# Returns median of each attribute in its statistics_ instance variable
imputer.statistics_

# Transform the dataset by replacing missing values by learned median
X = imputer.transform(dataset_df)

# Result is a plain Numpy array, put it back into a dataframe
dataset_df_transformed = pd.DataFrame(X, columns=dataset_df.columns)

{% endhighlight %}
