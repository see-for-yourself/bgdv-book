---
layout: post
title:  "Chapter 2: Plotting with Pyplot"
date:   2024-10-19
categories: book
TOC_order: 2
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

Pyplot is a module in the Matplotlib visualization library that provides a simple application programming interface (API) for making basic plots. It has many functions and can fill most needs. There are other visualization libraries for Python, and some of these are built on top of or employ Matplotib [1], and familiarity with the basics in Pyplot and Matplotlib can be valuable when using these other libraries.

In this chapter, we introduce Pyplot by plotting the climate data from the previous chapter. Then we show how to use smoothing to reduce the noise and to make the trends easier to see. After that, we plot some CO2 data that cover the same period as the temperature data, and make a plot with multiple subplots to facilitate visual comparison.

To run the code in this chapter, the Matplotlib and Pandas libraries need to be installed in your Python setup (see Appendix A).

## 2.1 Example: Plot Global Temperature Data

Let's plot the NASA global temperature anomaly data in the previous chapter. This can be done by using the Pyplot `plot()` function with the following code:

```python
import matplotlib.pyplot as plt

# Use temp_anoms from previous Chapter
# temp_anoms = [...]

# To get degrees Celsius, divide each data value by 100
y = [t/100 for t in temp_anoms]

x = range(1880, 2021)

plt.plot(x, y, color='black')
plt.xlabel('Years')
plt.ylabel('Temp Anomaly (Celsius)')
plt.show()
```

![Figure 2.1](/bgdv-book/assets/images/book/temp_anoms_v01-a_BW.png)  
_Figure 2.1. Plot of global temperature anomalies from 1880 to 2020. Data Source: NASA GISTEMP (Chapter 1 [7])._

## 2.2 Smoothing Data

The plot of the temperature anomaly data in Figure 2.1 looks noisy, which makes it somewhat difficult to see the trends. One way to deal with this is to smooth the data before plotting it. A common way to smooth data is to compute the _central moving average_.

To compute this, we can use the Pandas, which is an open source Python library for data analysis and manipulation [2]. In order to use Pandas, it needs to be installed in your Python setup (see Appendix A).

For the code, we add the library import and define the function:

```python
import pandas as pd

def central_moving_average(data_list, window):
    df = pd.DataFrame({'values': data_list})
    df['cma'] = df['values'].rolling(window, center=True, min_periods=1).mean()
    return df['cma']
```

This function converts the data from a Python list to a Pandas data frame object. (We will discuss data frames more in the next chapter.) Then it calls the rolling() function to compute the moving average, with the _window_ parameter specifying the window length.  For a central moving average, the _center_ parameter is set to True. Using an odd number _k_ for the window length will use _k/2_ points on each side of the current point _x_ (together with _x_) to compute the average.

The _min_periods_ parameter specifies how to handle the left and right ends of the interval: setting this to 1 will compute the average if there is at least 1 point in the window, and the result will have the same number of points as the the input data list.  Otherwise, if the optional _min_periods_ parameter is not set, the left and right ends of the result list will have _k/2_ values on each end set to _NaN_ (Not a Number).

Finally, add the line of code below to the code in Section 2.1, right below the line `y = ...` to replace those values with its smoothed values (using window length of 5).  The smoothed plot is shown in Figure 2.2.

```python
y = central_moving_average(y, window=5)
```

![Figure 2.2](/bgdv-book/assets/images/book/temp_anoms_cma-5_expand_v01-b_BW.png)  
_Figure 2.2. Plot of global temperature anomalies from 1880 to 2020, smoothed using a central moving average with window length 5._

## 2.3 Example: Plot CO2 Data

We can plot the amount of atmospheric carbon dioxide (CO2) by year, and compare this with the above plot of the temperature changes. To cover the same period as the temperature data, we can use data from these two sources: CDIAC Law Dome (for years up to 1978) [3] and NOAA GML (for years after 1979) [4].

We put the data here into two separate Python lists, along with their year ranges. The global average concentration of carbon dioxide is measured in parts per million (ppm). The data values have been rounded off to zero decimal places.

```python
# subset of CDIAC Lawdome data from 1880 to its end at 1978
co2_vals_law = [
    291, 291, 292, 292, 293, 293, 293, 294, 294, 294, 294, 294, 294, 295, 295,
    295, 295, 295, 295, 296, 296, 296, 296, 297, 297, 298, 298, 298, 299, 299,
    300, 300, 300, 301, 301, 301, 302, 302, 302, 303, 303, 303, 304, 304, 304,
    305, 305, 306, 306, 307, 307, 308, 308, 309, 309, 309, 310, 310, 310, 310,
    310, 310, 310, 310, 310, 310, 310, 310, 310, 310, 311, 311, 312, 312, 312,
    313, 314, 314, 315, 316, 316, 317, 318, 318, 319, 320, 321, 322, 323, 324,
    325, 326, 327, 328, 329, 330, 332, 333, 334]
years_law = range(1880, 1979)
    
# subset of NOAA GML data from its beginning at 1980 up to 2020
co2_vals_gml = [
    339, 340, 341, 343, 344,
    346, 347, 349, 351, 353, 354, 355, 356, 357, 358, 360, 362, 363, 366, 368,
    369, 371, 373, 375, 377, 379, 381, 383, 385, 386, 389, 391, 393, 395, 397,
    400, 403, 405, 408, 410, 412]
years_gml = range(1980, 2021)
```

Plotting these CO2 data is similar to plotting the temperature data, exept the graph has two separate segments. We can distinguish the Lawdome and GML segments by using different line styles. For color, we use 'gray'. The code for plotting them is below, and the result is in Figure 2.3. It does not look too noisy, and we will not smooth them for now.

```python
plt.plot(years_law, co2_vals_law, color='gray', linestyle='dotted')
plt.plot(years_gml, co2_vals_gml, color='gray', linestyle='dashed')
```

![Figure 2.3](/bgdv-book/assets/images/book/CO2_law-gml_v02-a_BW.png)  
_Figure 2.3. Plot of CO2 from 1880 to 2020. Lawdome data is shown by the dotted line, GML data by the dashed line._

## 2.4 Multiple Plots in a Figure

We can visually compare the trends in temperature anomaly and in CO2 by looking at the plots in Figures 2.2 and 2.3. A good way to do this is to plot them next to each other in a single figure. The code to do this as follows:

```python
    x = range(1880, 2021)

    y1 = temp_anoms
    y1 = [t/100 for t in temp_anoms]
    y1 = central_moving_average(y1, window=5)

    # Create a figure with 2 rows and 1 column
    fig, axes = plt.subplots(2, 1)

    # Plot temp anoms
    axes[0].plot(x, y1, color='black')
    axes[0].set_ylabel('Temp Anomaly (Celsius)')

    # Plot CO2
    axes[1].plot(years_law, co2_vals_law, color='gray', linestyle='dotted')
    axes[1].plot(years_gml, co2_vals_gml, color='gray', linestyle='dashed')
    axes[1].set_ylabel('CO2 (ppm)')
    axes[1].set_xlabel('Years')

    plt.tight_layout()  # Adjust spacing
    plt.show()
```

![Figure 2.4](/bgdv-book/assets/images/book/plot-2x1_temp-anoms_CO2_law-gml_v02-a_BW.png)  
_Figure 2.4. A single figure containing multiple subplots._

---

## Exercises

Ex.2.1. Try smoothing with different window lengths when computing the central moving average of Figure 2.2.

*Ex.2.2. Another widely used smoothing algorithm is _Lowess (locally weighted scatterplot smoothing)_. It is available in the statsmodels library for Python. Plot the temperature anomaly data using Lowess smoothing.

Ex.2.3. Plot a variation of the CO2 data as in Figure 2.3 with the line segments smoothed.
