---
layout: post
title:  "Chapter 10: Choropleth Maps"
date:   2025-05-23
categories: book
TOC_order: 10
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

A popular way to visualize geospatial data is to use _choropleth maps_, which associate the visual propeties of the geographic regions to the data within each region. In the previous chapter, we plotted points over a map without region boundaries. In order to create choropleth maps using GeoPandas, we also need to have the cartographic boundary files. These can be obtained from various sources such as the U.S. Census Bureau [1]. The U.S. Census Bureau foremost has data about its people, and we start with an example with data about population size and density of the states.

## 10.1 Example: Population Size and Density of U.S. States

For the population data of the U.S. states, we download the data from the U.S. Census Bureau [2] for the dataset "2019: ACS 1-Year Estimates Data Profiles" and save it as a CSV file. The population data is also readily available from other resources such as Wikipedia. We can wrangle this data by writing some Pandas code and putting it into a CSV file (states-48_population.csv) for the population of the 48 states in the continental U.S., which looks like:

```text
state,population
Alabama,4903185
Arizona,7278717
Arkansas,3017804
California,39512223
Colorado,5758736
...
```

For the cartographic boundary file, we use the file cb_2018_us_state_20m.shp (downloaded from the U.S. Census Bureau [1]). This file also contains the land area data for each state, which can be used to calculate the population density.

The code for making a choropleth map of the state population is below. First, we load the CSV file for population data into a Pandas data frame, and then convert it into units of millions. Next, we load the cartographic boundary file into a GeoPandas data frame `states`. The population data frame is merged into the `states` data frame; the target columns of the two data frames contain the name of the states but with different column names: 'NAME' and 'state', respectively.

Then the PyPlot figure is constructed and the `figsize` parameter is used to set the size with the desired aspect ratio. Using the `plot()` function of the GeoPandas data frame, we plot the data, specifying the axis of the figure. We also plot the legend with the colormap scale. We use a grayscale colormap (for making a color image, a color scale such as viridis can be substituted). The result is at the top of Figure 10.1.

```python
import pandas as pd
import geopandas
import matplotlib as mpl
import matplotlib.pyplot as plt
from geodatasets import get_path

# load the population data
population_data = pd.read_csv('states-48_population.csv')
# convert to millions
population_data['population'] = population_data['population'] / 1000000

# load the cartographic boundary file into a GeoPandas data frame
states = geopandas.read_file('cb_2018_us_state_20m.shp')

# merge the population data into the data frame
states = states.merge(population_data, left_on='NAME', right_on='state')

# construct the figure
fig, ax = plt.subplots(1, 1, figsize=(6, 4))

# plot the data frame
states.plot(column='population', cmap=mpl.colormaps['Greys'], 
            linewidth=0.8, ax=ax, edgecolor='0.8', 
            legend=True, legend_kwds={'shrink': 0.5})

ax.set_title('State Population (millions)')
ax.set_axis_off()
plt.show()
```

![Figure 10.1a](/assets/images/book/us-states_population_cmap-greys_v01-e_crop.png)  

![Figure 10.1b](/assets/images/book/us-states_pop-density_cmap-greys_v01-e_crop.png)  
_Figure 10.1. Population and population density of the U.S. states (2019). Data source: U.S. Census Bureau [1]._

To make the **population density** choropleth map (Figure 10.1-bottom), only a minor modification is needed. The cartographic boundary file (cb_2018_us_state_20m.shp) also contains the land area for each state under the column "ALAND". This is in square meters, which we can convert to square miles by dividing by 2.59e6. The population density of a state is defined as its population divided by its land area. Doing this calculation and adding a new column "pop_density" to the GeoPandas data frame `states` can be done in one line of code:

```python
states['pop_density'] = (states['population']*1000000)/(states['ALAND']/2.59e6)
```

Then to make the plot, in the `states.plot()` function, we use the parameter `column='pop_density'`, and also set the appropriate text for the title in the `ax.set_title()` function.

## 10.2 Example: Focus-Plus-Context Choropleth Maps

In the choropleth maps for the U.S. states, it can be somewhat difficult to see the smaller states in the northeast of the country. One way to address this issue is to use the focus-plus-context technique that we covered earlier in Chapter 8.

For example, we can take the above data for the population density of the U.S. states and make a focus-plus-context choropleth map using the following ASCII layout design:

```text
...AA
...AA
BBBBB
BBBBB
```

The corresponding code is:

```python
fig, axes = plt.subplot_mosaic('...AA;...AA;BBBBB;BBBBB', figsize=(6, 4.5))
axesA, axesB = axes['A'], axes['B']
```

The rest of the code is very similar to what we have done in chapter 8, and we it as an exercise. We mention one major difference: For the regular Pyplots, we use the `Axes` functions `set_xlim()` and `set_ylim()` to set the bounds of the contents in the focus region and in the context region. For the GeoPandas plots, we instead use the `clip()` fucntion of the GeoPandas data frame, and provide the axis as a parameter. In our current example, the code for plotting the clipped map as the focus region (with a legend) and full map as the context region is:

```python
states.clip([patch_left, patch_bottom, patch_right, patch_top]).plot(
    column='pop_density', ax=axesA, 
    cmap=cmap, linewidth=0.8, edgecolor='0.8',
    legend=True, legend_kwds={'shrink': 0.5})
states.plot(
    column='pop_density', ax=axesB, 
    cmap=cmap, linewidth=0.8, edgecolor='0.8')   
```

The resulting image is shown in Figure 10.2. We can see the small northeast states such as Rhode Island much better compared to the simple map (Figure 10.1-bottom).

![Figure 10.2](/assets/images/book/us-states_pop-density_cmap-greys_clip_cutout_v02-e_crop.png)  
_Figure 10.2. A focus-plus-context plot of the population density of the U.S. states (2019). Data source: U.S. Census Bureau [1]._

## Exercises

Ex.10.1. Make a focus-plus-context choropleth map for the U.S. states that looks like the one in Figure 10.2.

*Ex.10.2. Make choropleth maps for the U.S. states for the Covid-19 cases and deaths, and also for cases and deaths per capita. Downloaded the dataset files (time_series_covid19_confirmed_US.csv and time_series_covid19_deaths_US.csv) from the CSSE website: <https://coronavirus.jhu.edu/about/how-to-use-our-data>
