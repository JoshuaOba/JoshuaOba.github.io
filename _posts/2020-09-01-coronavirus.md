---
layout: post
title:  "Modeling Coronavirus Data with Jupyter Notebooks"
date:   2020-09-02 15:13:12 -0600
--- 

The Coronavirus Pandemic of 2020 took the world by storm. With many experts and governments responding to the threat in a multitude of different ways. In the Digital Age, computers and software can help decision makers make the most informed choices possible and with the right data, display it in easy to interpret forms.

Taking data from John Hopkins University's Coronavirus Resource Center and analyzing the data given, we can use simple Python code to sort and display it on Google Maps to see where the virus has inflicted the most damage. 

We start by importing our libraries and data:

{% highlight ruby %}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.colors as mcolors
import random
import math
import time
from sklearn.model_selection import RandomizedSearchCV, train_test_split
from sklearn.svm import SVR
from sklearn.metrics import mean_squared_error, mean_absolute_error
import datetime
import operator
import gmaps
import gmaps.datasets
import gmaps.geojson_geometries
gmaps.configure(api_key="AIzaSyBcns7lcFhOACDVrQGcXclTqUhP54BhuYI")
plt.style.use('seaborn')
%matplotlib inline
{% endhighlight %}

{% highlight ruby %}
confirmed_cases = pd.read_csv('Downloads/time_series_covid19_confirmed_global.csv')
deaths_reported = pd.read_csv('Downloads/time_series_covid19_deaths_global.csv')
recovered_cases = pd.read_csv('Downloads/time_series_covid19_recovered_global.csv')
{% endhighlight %}