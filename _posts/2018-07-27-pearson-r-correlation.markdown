---
layout: post
title:  "Pearson-r correlation"
date:   2018-07-26 21:46:30 +0300
categories: modelling
---
## Pearson-r correlation
If your dataset is not too large, you can easily compute the standard correlation coefficient (also called Pearson’s r) between every pair of attributes using
the corr() method:

{% highlight python %}
corr_matrix = dataset_df.corr()
{% endhighlight %}

In order to look at how much each attribute correlates with a specific column:

{% highlight python %}
corr_matrix["desired_column"].sort_values(ascending=False)
{% endhighlight %}

The correlation coefficient ranges from –1 to 1. When it is close to 1, it means that there is a strong positive correlation. When the coefficient is close to –1, it means that there is a strong negative correlation. Finally, coefficients close to zero mean that there is no linear correlation.

**NOTE:** The correlation coefficient only measures linear correlations (“if x goes up, then y generally goes up/down”). It may completely **miss out on nonlinear relationships** (e.g., “if x is close to zero then y generally goes up”).

Another way to check for correlation between attributes is to use Pandas’ **scatter_matrix** function, which plots every numerical attribute against every other numerical attribute.

{% highlight python %}
from pandas.tools.plotting import scatter_matrix

scatter_matrix(dataset_df)
{% endhighlight %}
