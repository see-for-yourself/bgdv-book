---
layout: post
title:  "Chapter 7: Polar Plots with Pyplot"
date:   2024-11-14
categories: book
TOC_order: 7
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

In this chapter, we show how to make polar plots with Pyplot. Continuing with the example of Climate Sprials from the previous chapter, we show how to do it with Pyplot by making minor modifications to the Turtle code. We present another interesting example with the Covid-19 data to visualize seasonal patterns.

## 7.1 Polar Coordinates in Pyplot

Polar plots can be made using Pyplot's support for polar coordinates. Unlike turtle graphics which draws on a blank screen, Pyplot provides coordinate axes and labels. To make polar plots with Pyplot, set the `'projection'` to `'polar'` in the function parameters for creating a figure.

To illustrate with a simple example, a Fermat spiral is drawn by the following code and the result is shown in Figure 6.1. Fermat spirals can be used to model natural phenomena like sunflower florets [1], and for engineering applications such as efficient layout of mirrors in solar power plants [2].

```python
import matplotlib.pyplot as plt

turns = 3
n_steps = 100*turns
step = turns*2*math.pi/n_steps
angles, radii = [], []    
for i in range(n_steps):
    theta = i*step
    r = math.sqrt(theta)
    angles.append(theta)
    radii.append(r)        

fig, ax = plt.subplots(subplot_kw={'projection': 'polar'})
ax.plot(angles, radii, color='black')
plt.show()
```

![Figure 7.1](/bgdv-book/assets/images/book/fermat_spiral_turns-3_v01-b_dpi-75.png)  
_Figure 7.1. Fermat spiral._

## 7.2 Example: Climate Spirals Redux

In the previous chapter, we demonstrated how to use Turtle to draw Climate Spirals [3].

To use Pyplot to make a polar plot of a Climate Spiral, we can use Pyplot's polar projection. It can provide a polar coordinates grid with labels, so that the months and the radii of the plotted values can be read off by the user. The resulting plot is shown in Fig. 7.2.

![Figure 7.2](/bgdv-book/assets/images/book/climate_spiral_1880-2020_v01-e_viridis_dpi-75.png)  
_Figure 7.2. A Climate Spiral made with Pyplot, for data from 1880 to 2020. Data source: NASA GISTEMP [4]._

The code is similar to the Turtle code in the previous chapter, with a few minor modifications. Taking that code, add above the for loop:

```python
# create the polar plot
fig, ax = plt.subplots(subplot_kw={'projection': 'polar'})
```

Inside the for loop, modify the if-else statement:

```python
    if i < (df_year.size - 1):
        year_vals.append(df_months.iloc[i+1].iloc[0])
        ax.plot(theta_months_close, year_vals, color=color, linewidth=0.5)
    else:
        ax.plot(theta_months, year_vals, color=color, linewidth=0.5)
```

And below the for loop, add code to specify the grid ticks and labels:

```python
ax.set_rmin(-2)
ax.set_rmax(2)
ax.set_rticks([-2, 2])
ax.set_xticks(theta_months)
ax.set_xticklabels(months)
plt.show()
```

Finally, remove the code that calls the Turtle functions, and remove the function `draw_year()`.

## 7.3 Example: Seasonal Patterns in Covid-19 Data

In Chapter 5, we had aggregated the CSSE Covid-19 data and grouped them into years and months, calculated the monthly means, and saved the data to a CSV file. We had plotted the monthly means in Cartesian coordinates (Figure 5.4). In that plot, it is not so easy to see the seasonal patterns. Here, we make a polar plot that reveals the seasonal patterns more clearly. See Figure 7.3. The plot shows that during the winter months of December to February in the US, the Covid-19 deaths spike up. It also shows the progression of the pandemic as it winds up and down during the three years of the pandemic.

![Figure 7.3](/bgdv-book/assets/images/book/polar-plot_US_pointvals_monthly-mean_deaths.png)  
_Figure 7.3. A polar plot of Covid-19 monthly mean deaths. Data source: Johns Hopkins CSSE [5]._

Since our saved data file (US_pointvals_monthly-mean.csv) had been made so that it has the same structure as the NASA GISTEMP climate data file (GLB.Ts+dSST.csv), it is straightforward to modify the code above for plotting the climate data in polar coordinates to plot the Covid-19 data. We leave the details to Exercise 7.1.

## 7.4 Transforming Data

Transforming data is an important process in data visualization and data science. It can be used to make the patterns easier to see in a visualization, to normalize the scale for statistical analysis, or to coerce the data to be more amenable to modeling with methods such as regression. Transforming data may involve using different coordinate systems, which we have been doing in this and the previous chapters with Cartesian and polar coordinates. A more common way to transform data is to apply a function to the data values; often used functions include logarithm, square, square root, etc.

As an example, we show how transforming the Covid-19 cases data can improve the visualization. Figure 7.4 shows the polar plots for linear, square root, and log (base 10) functions applied to the radial values. The graph is more legible in the square root plot than the other two, with the values less compressed together in the radial direction.

Making these polar plots with transformed values for the Covid-19 cases is similar to making the above polar plot for the deaths. Judicious placement of the grid lines helps to convey the non-linear scale. This is done manually by choosing the tick values, and their tick label placements should also try to avoid overlapping with the graph. The tranformation function is applied on the tick values before setting them with `ax.set_rticks()`. The reader is encouraged to make these plots and try other transformation functions (Exercise 7.3).

![Figure 7.4](/bgdv-book/assets/images/book/polar-plot_US_pointvals_monthly-mean_cases_transform-1x3_800x350.png)  
_Figure 7.4. Different data transformations. Data source: Johns Hopkins CSSE [4]._

---

## Exercises

Ex.7.1. Create Figure 7.3. Modify the code in Chapter 5 to read in our data file with the Covid-19 monthly mean values (US_pointvals_monthly-mean.csv), and modify the code in this chapter to plot that data in polar coordinates. Also, use a sequential grayscale colormap 'Greys' instead of 'viridis' which does not look as good for this visualzation.

Ex.7.2. From the CSSE Covid-19 dataset, select a country from the southern hemispherea and make a polar plot as in Figure 7.3. Compare it to the plot of the US, which is country in the northern hemisphere with different months for the seasons.

Ex.7.3. Make a polar plot with transformed values for Covid-19 cases, using square root function. Try using a natural logarithm transformation function (`log(x)`) and compare it to the plot in Figure 7.4 with the logarithm base 10 (`log10(x)`); which is more human-readable in terms of the grid lines and grid labels?

*Ex.7.4. Make a similar polar plot to the monthly mean Covid-19 deaths in Figure 7.3, but instead of with monthly resolution, do it with daily resolution using the point values (7-day central moving average) computed in Section 5.2. Note that the months have different number of days, and February also has different number of days because of leap years.
