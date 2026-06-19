---
layout: post
title:  "Chapter 7: Polar Plots with Pyplot"
date:   2024-11-14
categories: book
TOC_order: 7
copyright: "Copyright © 2024-2026, P. L. Chiu. All Rights Reserved."
---

In this chapter, we show how to make polar plots with Pyplot, and do an example with the Covid-19 data to visualize seasonal patterns. We also show how transforming the data values can enhance the readability of the plot by making the values less compressed together in the radial direction.

## 7.1 Polar Coordinates in Pyplot

Circular plots can be made using Pyplot's support for polar coordinates in which a point is represented by (theta, r), which are its angle and radius. Pyplot also displays polar coordinate axes and labels that can be customized. To make polar plots with Pyplot, set the `"projection"` to `"polar"` in the function parameters for creating a figure.

The difference between using polar and Cartesian coordinates is shown in Figure 6.3, and the corresponding code is below.

![Figure 7.1](/bgdv-book/assets/images/book/archimedean_spiral_1x2_polar-cartesian_v01-a.png)  
*Figure 7.1. An Archimedean spiral in polar and in Cartesian coordinates.*

```python
# calculate coordinates for Archimedean spiral
nsegs = 370
b = 6
angle = 0.05  # radians

theta_vals, r_vals = [], []
x_vals, y_vals = [], []
for i in range(nsegs):
    theta = i * angle
    r = b * theta
    theta_vals.append(theta)
    r_vals.append(r)

    x = r * math.cos(theta)
    y = r * math.sin(theta)
    x_vals.append(x)
    y_vals.append(y)

fig = plt.figure(figsize=(8, 4))

# subplot in polar coordinates
ax_polar = fig.add_subplot(1, 2, 1, projection="polar")
ax_polar.plot(theta_vals, r_vals)

# subplot in Cartesian coordinates
ax_cartesian = fig.add_subplot(1, 2, 2)
ax_cartesian.set_aspect("equal")
ax_cartesian.grid(True)
ax_cartesian.set_xticks(np.arange(-120, 120, 20))
ax_cartesian.set_yticks(np.arange(-120, 120, 20))
ax_cartesian.plot(x_vals, y_vals)

fig.tight_layout()
plt.show()
```

## 7.2 Example: Seasonal Patterns in Covid-19 Dataset

In Chapter 5, we had aggregated the CSSE Covid-19 data and grouped them into years and months, calculated the monthly means, and saved the data to a CSV file. We had plotted the monthly means in Cartesian coordinates (Figure 5.4). In that plot, it is not so easy to see the seasonal patterns. Here in this chapter, we make a polar plot that reveals the seasonal patterns more clearly. See Figure 7.2. The plot shows that during the winter months of December to February in the US, the Covid-19 deaths spike up. It also shows the progression of the number of deaths as it winds up and down during the three years of the pandemic.

![Figure 7.2](/bgdv-book/assets/images/book/polar-plot_US_pointvals_monthly-mean_deaths_v02-a.png)  
*Figure 7.2. A polar plot of Covid-19 monthly mean deaths. Data source: Johns Hopkins CSSE [1].*

The first step is to load the CSV data file for the monthly Covid-19 deaths in the US, which we had created in Chapter 5:

```python
import pandas as pd
df = pd.read_csv("deaths/US_pointvals_monthly-mean.csv")
df_year = df["Year"].astype(int)
months = df.columns[1:13].tolist()
df_months = df[months].astype(float)
```

We will represent the data in polar coordinates (theta, r) where theta is the angle position of the month and r is the number of deaths. For the monthly polar plot layout, we can reuse the function `month_angles()` from the Turtle Climate Spiral program in the previous chapter. This calculates the sequence of month angles for a single year that places January at the 12 o'clock position, and makes the line plot revolve in the clockwise direction.

Next, we define a function `draw_year()` for drawing a year's path with a specified colormap value; the idea is to use a higher value for the more recent years. Unlike with the Turtle Climate Spiral program, there is no need to transform the polar coordinates to Cartesian coordinates.

```python
def draw_year(theta_months, year_vals, color, linewidth, ax):
    ax.plot(theta_months, year_vals, color=color, linewidth=linewidth)
```

Then the rest of the code for making a polar plot of the Covid-19 data is:

```python
import matplotlib as mpl
fig, ax = plt.subplots(subplot_kw={'projection': 'polar'})
colormap = mpl.colormaps['Greys']  # Sequential 
brightness_range = 0.8  # use a lower value for grayscale colormaps

theta_months, theta_months_close = month_angles()
for i in range(df_year.size):
    value = i / (df_year.size - 1)
    color_value = value * brightness_range + (1.0 - brightness_range)
    color = colormap(color_value)

    year_vals = df_months.iloc[i].tolist()
    if i < (df_year.size - 1):
        year_vals.append(df_months.iloc[i + 1].iloc[0])
        draw_year(theta_months_close, year_vals, color, ax)
    else:
        draw_year(theta_months, year_vals, color, ax)

ax.set_xticks(theta_months)
ax.set_xticklabels(months)
ax.set_rticks([1000, 2000, 3000])
ax.set_rlabel_position(-80)

title = "Covid-19 in US\nmonthly average of deaths per day"
title += "\n1/22/20 to 3/9/23"
ax.set_title(title)

fig.tight_layout()
plt.show()
```

## 7.3 Transforming Data

Transforming data is an important process in data visualization and data science. It can be used to make the patterns easier to see in a visualization, to normalize the scale for statistical analysis, or to coerce the data to be more amenable to modeling with methods such as regression. Transforming data may involve using different coordinate systems, which we have been doing in this and the previous chapters with Cartesian and polar coordinates. A more common way to transform data is to apply a function to the data values; often used functions include logarithm, square, square root, etc.

As an example, we show how transforming the Covid-19 cases data can improve the visualization. Figure 7.3 shows the polar plots for linear, square root, and log (base 10) functions applied to the radial values. The graph is more legible in the square root plot than the other two, with the values less compressed together in the radial direction.

![Figure 7.3](/bgdv-book/assets/images/book/polar-plot_US_pointvals_monthly-mean_cases_transform-1x3_v02-a.png)  
*Figure 7.3. Different data transformations. Data source: Johns Hopkins CSSE [1].*

Making these polar plots with transformed values for the Covid-19 cases is similar to making the above polar plot for the deaths. For cases, we load the data file "cases/US_pointvals_monthly-mean.csv". To make it more convenient in the code to use different transformations, we assign a transformation function to a variable. As an example using the square root transformation, we proceed as follows:

```python
transform_func = math.sqrt
```

Then in the code in the previous section, insert the following line of code right above each of the two calls to draw_year():

```python
year_vals = [transform_func(x) for x in year_vals]
```

The ticks and tick labels are chosen manually so that the placement of the grid lines helps to convey the non-linear scale. Moreover, we try to avoid glaring overlaps with the plot line path. Note that the transformation function is applied on the tick values before setting them with `ax.set_rticks()`:

```python
rticks=[transform_func(200000), transform_func(400000), transform_func(600000)]
yticklabels = ['200000', '400000', '600000']
```

---

## Exercises

Ex.7.1. From the CSSE Covid-19 dataset, select a country from the southern hemisphere and make a polar plot as in Figure 7.2. Compare it to the plot of the US, which is country in the northern hemisphere with different months for the seasons.

Ex.7.2. Make polar plots with transformed values for the Covid-19 cases data, as shown in Figure 7.3, using the log (base 10) transformation function `math.log10`.

*Ex.7.3. Make a polar plot similar to the monthly mean Covid-19 deaths in Figure 7.2, but instead of with monthly resolution, do it with daily resolution using the point values (7-day central moving average) computed in Section 5.2. Note that the months have different number of days, and February also has different number of days because of leap years.

Ex. 7.4. Make a Climate Spiral with Pyplot. *Hint*: Since the structure of the NASA GISTEMP climate data file (GLB.Ts+dSST.csv) is essentially the same as for our Covid-19 monthly data, it is straightforward to modify the code above for plotting the Covid-19 data in polar coordinates to plot the climate data.
