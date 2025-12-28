---
layout: post
title:  "Chapter 9: Geospatial Visualization with GeoPandas"
date:   2025-05-19
categories: book
TOC_order: 9
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

So far, we have dealt with visualizing data over time. To visualize data over space, maps are often used. In this chapter, we cover some basics of geospatial visualization with GeoPandas and show examples of plotting data points over a map. And in the next chapter we will associate the visual properties of the geographic regions to the data within each region to create choropleth maps.

## 9.1 Plotting Points over a Map with GeoPandas

GeoPandas is an open source Python library for geospatial visualization that uses data structures which are compatible with Pandas. It also relies on Matplotlib, whose functions can be used to tweak the designs of the visualizations.

We briefly outline the overall process and then explain in more detail through a couple of examples with code. Before running the code, install the GeoPandas [1] and Geodatasets [2] (see Appendix A).

An overview of the plotting process is as follows: The geospatial data points can be represented by longitude and latitude. This data is put into a Pandas `DataFrame`. From this, a `geopandas.GeoDataFrame` object is constructed with additional information about the geometry and the coordinate reference system. The map data is read in from a separate file. Map data is available from the Geodatasets library or from other sources on the Internet. After that, the `GeoDataFrame.plot()` function is used to plot the data points over the map.

## 9.2 Example: Seven Summits on a World Map

The Seven Summits are the highest mountain peaks on the seven continents. According to Wikipedia [3], under different definitions of "continent", there are 9 peaks altogether. In this section, we show how use GeoPandas to plot these 9 peaks over a world map and place text labels near them with their names (Figure 9.1).

![Figure 9.1](/bgdv-book/assets/images/book/SevenSummits-9_v03-b_crop.png)  
_Figure 9.1. Seven Summits: 9 mountains from different definitions of the "seven continents". Data source: Wikipedia [3]._

Along with the name of the mountains, Wikipedia also has their geographic coordinates. These data, with the coordinates rounded to 3 decimal places, can be put into a CSV file (SevenSummits-9.csv) which has the following content:

```text
name,latitude,longitude
Aconcagua,-32.653,-70.012
Denali,63.07,-151.007
Elbrus,43.355,42.439
Everest,27.988,86.925
Kilimanjaro,-3.076,37.353
Kosciuszko,-36.456,148.264
Mont Blanc,45.833,6.865
Puncak Jaya,-4.079,137.158
Vinson,-78.526,-85.617
```

In the code below, we use Pandas to read in this CSV file, and then construct a `geopandas.GeoDataFrame` object specifying the geometry as points with x and y coordinates from longitude and latitude columns, and the coordinate reference system (crs) as "EPSG:4326", which is a widely used geographic coordinate system.

Next, we load the map data using the `geodatasets.read_file()` and `geodatasets.get_path()` functions. A world map is given by the file "naturalearth.land" from geodatasets.

For a cleaner look, we remove the tick marks and labels by passing an empty list to the functions for setting the axis ticks. Then we call the `plot()` function to plot the data points. Triangular markers are suitable for representing mountains, and we do this by using the marker type "^".

To annotate each mountain with its name, we iterate through the items of the columns simultaneously using the Python `zip()` function, and call the Matplotib `annotate()` function. In the resulting image, there is an issue with the text labels "Mont Blanc" and "Elbrus" overlapping. To fix this, we can change the location of the "Elbrus" text label.

```python
import pandas as pd
import geopandas
import geodatasets
import matplotlib.pyplot as plt

df = pd.read_csv("SevenSummits-9.csv")
gdf = geopandas.GeoDataFrame(
    df, 
    geometry=geopandas.points_from_xy(df.longitude, df.latitude), 
    crs="EPSG:4326"
)

world = geopandas.read_file(geodatasets.get_path("naturalearth.land"))

# plot the map
ax = world.plot(color="lightgray")
ax.set_xticks([])
ax.set_yticks([])

# plot the data points
gdf.plot(ax=ax, color="black", marker="^", markersize=10)

# annotate the data points with the names
for x, y, name in zip(gdf.geometry.x, gdf.geometry.y, df.name):
    xytext = (3, 3)
    # avoid overlapping text with "Elbrus"
    if name == "Elbrus":
        xytext = (3, -10)
    ax.annotate(name, xy=(x, y), xytext=xytext, 
                textcoords="offset points", size=7)

plt.tight_layout()
plt.show()

```

## 9.3 Example: Bird Migration Radars over the U.S

In the work of Horton et al., 2020 [4], weather surveillance radars over the continental U.S. were used to acquire data of bird migration activity along with the air surface temperature, and data analysis was performed to study the effects of climate change on the shifts over time of the peak migration dates. Their dataset is available at [4].

In this example, we will show how to use GeoPandas to plot the radar locations from the dataset over a map that is clipped to show only North America, and also to draw vertical lines denoting the flyway boundaries. See Figure 9.2.

![Figure 9.2](/bgdv-book/assets/images/book/plot_radars-flyways-over-map_lightgray-black_v02-a_crop.png)  
_Figure 9.2. Weather surveillance radars used to detect bird migration activity over the U.S., along the Western, Central, and Eastern flyways. Data source: Horton et al. [4]._

There are 143 radars used to record the spring and fall data. In the code below, we use Pandas to read in the CSV file for the spring data from the dataset [4], and drop the duplicate rows by keeping only the first row of each unique radar ID. As in the previous section, we construct a `geopandas.GeoDataFrame` object specifying the geometry as points with x and y coordinates from longitude and latitude columns, and the coordinate reference system (crs) as "EPSG:4326".

Next, we load the map data using the `geodatasets.get_path()` function. Since we want to crop our view to only show North America, we clip the map. Finally, we call the `plot()` function and add the axis labels.

To draw the dashed vertical lines representing the boundaries of the Western, Central, and Eastern flyways, we use the Matplotib `axvline()` function. The x-coordinates of the flyway boundaries are not given in the dataset file, but they can be determined to be approximately -103 and -90 using the information in the paper [4] that there are 40, 42, and 61 radars in the Western, Central, and Eastern flyways, respectively. See Exercise 9.1.

```python
df_spring = pd.read_csv("spring_data.csv")
df_unique = df_spring.drop_duplicates(subset=["radar_id"], keep="first")

gdf = geopandas.GeoDataFrame(
    df_unique, 
    geometry=geopandas.points_from_xy(df.LONGITUDE_W, df.LATITUDE_N), 
    crs="EPSG:4326"
)

world = geopandas.read_file(geodatasets.get_path("naturalearth.land"))

# restrict the view to North America
ax = world.clip([-150, 15, -45, 58]).plot(color="lightgray")

gdf.plot(ax=ax, color="black", markersize=3)
ax.set_xlabel("Longitude")
ax.set_ylabel("Latitude")

# draw vertical dashed lines for the flyways
# between Western and Central flyways:
plt.axvline(x=-103, color="gray", alpha=0.5, linestyle="--")
# between Central and Eastern flyways:
plt.axvline(x=-90, color="gray", alpha=0.5, linestyle="--")
```

Before moving on to the topic of choropleth maps, we can also make plots of the peak migration temporal data to look at the trends. The data for the peak migration (day of year) is in the column "q50", and the air surface temperature is in the column "air.sfc.mean". As we have done in previous chapters, we can calculate the 5-year moving averages and organize the plots in a grid as in Figure 9.3. We leave the details to the exercises. The plots indicate that over the 24 years of data from 1995 to 2018, the air surface temperature has increased, and the peak migration has come earlier in the year.

![Figure 9.3](/bgdv-book/assets/images/book/plot-2x2_years_peak_temp_cma-5_v03-a.png)  
_Figure 9.3. Bird migration activity over the U.S. (5-year moving average). Data source: Horton et al. [2]._

## Exercises

Ex.9.1. For the flyway boundaries, sort the radars by their longitude, then pick a number between the 40th and 41st radars and another number between the 82nd and 83rd radars, and check that these two numbers are approximately -103 and -90, respectively.

Ex.9.2. Make a plot of the bird migration radars (similar to Figure 9.2) with the map clipped to show a view of the continental U.S. instead of North America.

Ex.9.3. Make a 2x2 plot (similar to Figure 9.3) with columns for spring and fall and rows for peak migration and surface air temperature.
