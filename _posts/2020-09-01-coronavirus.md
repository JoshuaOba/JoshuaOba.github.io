---
layout: post
title:  "Modeling Coronavirus Data with Jupyter Notebooks"
date:   2020-09-02 15:13:12 -0600
--- 

The Coronavirus Pandemic of 2020 took the world by storm. With many experts and governments responding to the threat in a multitude of different ways. In the Digital Age, computers and software can help decision makers make the most informed choices possible and with the right data, display it in easy to interpret forms.

Taking data from John Hopkins University's Coronavirus Resource Center and analyzing the data given, we can use simple Python code to sort and display it on Google Maps to see where the virus has inflicted the most damage. 

We start by importing our libraries and data:

{% highlight python %}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.colors as mcolors
import random
import math
import time
import datetime
import operator
import gmaps
import gmaps.datasets
import gmaps.geojson_geometries
gmaps.configure(api_key="AIzaSyBcns7lcFhOACDVrQGcXclTqUhP54BhuYI")
plt.style.use('seaborn')
%matplotlib inline
{% endhighlight %}

{% highlight python %}
confirmed_cases = pd.read_csv('Downloads/time_series_covid19_confirmed_global.csv')
deaths_reported = pd.read_csv('Downloads/time_series_covid19_deaths_global.csv')
recovered_cases = pd.read_csv('Downloads/time_series_covid19_recovered_global.csv')
{% endhighlight %}

Next, we take all the column heads and assign variables for the values corresponding to each dataset:

{% highlight python %}
cols = confirmed_cases.keys()

confirmed = confirmed_cases.loc[:, cols[4]:cols[-1]]
deaths = deaths_reported.loc[:, cols[4]:cols[-1]]
recoveries = recovered_cases.loc[:, cols[4]:cols[-1]]
{% endhighlight %}

Then, we create variables to sort some basic data and get some general values for the current global totals:

{% highlight python %}
dates = confirmed.keys()
world_cases = []
total_deaths = []
mortality_rate = []
total_recovered = []

for i in dates:
    confirmed_sum = confirmed[i].sum()
    death_sum = deaths[i].sum()
    recovered_sum = recoveries[i].sum()
    world_cases.append(confirmed_sum)
    total_deaths.append(death_sum)
    mortality_rate.append(death_sum/confirmed_sum)
    total_recovered.append(recovered_sum)
{% endhighlight %}

Cleaning up the data to make sure it's ready for the next stages or sorting:

{% highlight python %}
days_since_1_22 = np.array([i for i in range(len(dates))]).reshape(-1, 1)
world_cases = np.array(world_cases).reshape(-1, 1)
total_deaths = np.array(total_deaths).reshape(-1, 1)
total_recovered = np.array(total_recovered).reshape(-1, 1)
{% endhighlight %}

We can see the latest values using a variable dedicated to the most recent data entry in each file:

{% highlight python %}
latest_confirmed = confirmed_cases[dates[-1]]
latest_deaths = deaths_reported[dates[-1]]
latest_recoveries = recovered_cases[dates[-1]]
{% endhighlight %}

We then want to see in our notebook the countries and their respective case numbers. We do that by listing all the unique countries in our data, attaching their respective case values, and sorting them from largest to smallest in our array:

{% highlight python %}
unique_countries = list(confirmed_cases['Country/Region'].unique())
country_confirmed_cases = []
no_cases = []
for i in unique_countries:
    cases = latest_confirmed[confirmed_cases['Country/Region']==i].sum()
    if cases > 0:
        country_confirmed_cases.append(cases)
    else:
        no_cases.append(i)
        
for i in no_cases:
    unique_countries.remove(i)
    
unique_countries = [k for k, v in sorted(zip(unique_countries, country_confirmed_cases), key=operator.itemgetter(1), reverse=True)]
for i in range(len(unique_countries)):
    country_confirmed_cases[i] = latest_confirmed[confirmed_cases['Country/Region']==unique_countries[i]].sum()
{% endhighlight %}

At this point, we can ask jupyter to output this organized data in a variety of ways, whether that's pie charts or graphs. However, we want to visualize it using Google Maps. We start by loading in the countries geometries in geojson and making a list with the countries and cases, similar to the previous step:

{% highlight python %}
countries_geojson = gmaps.geojson_geometries.load_geometry('countries')
country_and_cases = {}

for i in range(len(unique_countries)):
    country_and_cases[unique_countries[i]] = country_confirmed_cases[i]
{% endhighlight %}

