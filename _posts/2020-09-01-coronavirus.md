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
gmaps.configure(api_key="AI...")
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

At this point, we can ask jupyter to output this organized data in a variety of ways, whether that's pie charts or graphs. However, we want to visualize it on a Chloropleth Map using Google Maps. We start by loading in the countries geometries in geojson and making a list with the countries and cases, similar to the previous step:

{% highlight python %}
countries_geojson = gmaps.geojson_geometries.load_geometry('countries')
country_and_cases = {}

for i in range(len(unique_countries)):
    country_and_cases[unique_countries[i]] = country_confirmed_cases[i]
{% endhighlight %}

We then want to assign a color to each country depending on the number of cases it has respective to other countries. So we scale the values between 0 and 1 and assign it a color from matplotlib:

{% highlight python %}
from matplotlib.cm import viridis
from matplotlib.colors import to_hex

min_gini = min(country_and_cases.values())
max_gini = max(country_and_cases.values())
gini_range = max_gini - min_gini

def calculate_color(gini):
    """
    Convert the data to a color
    """
    # make gini a number between 0 and 1
    normalized_gini = (gini - min_gini) / gini_range

    # invert gini so that high inequality gives dark color
    inverse_gini = 1.0 - normalized_gini

    # transform the data to a matplotlib color
    mpl_color = viridis(inverse_gini)

    # transform from a matplotlib color to a valid CSS color
    gmaps_color = to_hex(mpl_color, keep_alpha=False)

    return gmaps_color
{% endhighlight %}

Here we assign the colors from the previous step and check for potential holes in our data. If the program cannot find a value for a country, or the country's name in the data file does not match the one provided from countries_geojson, it will be output as text and assigned a default color:

{% highlight python %}
colors = []
errors = []
for feature in countries_geojson['features']:
    country_name = feature['properties']['name']
    try:
        gini = country_and_cases[country_name]
        color = calculate_color(gini)
    except Exception as e:
        # no data for that country: return default color
        color = (0, 0, 0, 1)
        errors.append(str(e))
    colors.append(color)

errors
{% endhighlight %}

Finally, we can output our finished map:

{% highlight python %}
fig = gmaps.figure()
gini_layer = gmaps.geojson_layer(
    countries_geojson,
    fill_color=colors,
    stroke_color=colors,
    fill_opacity=0.5)
fig.add_layer(gini_layer)
fig
{% endhighlight %}

![coronachloropleth](/assets/Chloropleth.png)