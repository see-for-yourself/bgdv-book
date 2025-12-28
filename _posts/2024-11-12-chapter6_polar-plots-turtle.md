---
layout: post
title:  "Chapter 6: Drawing Polar Plots with Turtle"
date:   2024-11-12
categories: book
TOC_order: 6
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

Polar plots (also called radial plots) are suitable for data that have periodic structure or seasonal patterns. Typically, the data is modeled in polar coordinates, and the path of the plot revolves around a center point. In this chapter, we demonstrate polar plotting with Python Turtle with the example of Climate Spirals.  In the next chapter, we will show how to make polar plots using Pyplot.

## 6.1 Drawing a Polar Plot

Turtle can draw a line segment by moving with respect to relative or absolute parameters. As a simple example of drawing a circle using relative parameters, we can tell Turtle to turn at a constant angle. To neatly close up the circle when it comes around, the number of segments is set to be a divisor of 360 degrees; then the turning angle is 360 divided by the number of segments. Using a larger the number of segments produces a smoother circle. The distance to move forward after each turn will determine the size of the circle. Here is the code and below is the resulting image:

```python
import turtle as t

segments = 120
angle = 360 / segments
for i in range(segments):
    t.left(angle)
    t.forward(5)

t.hideturtle()
t.mainloop()
```

![Figure 6.1](/bgdv-book/assets/images/book/turtle_polarplot_circle_v01-a.png)  
_Figure 6.1. A circle drawn with Turtle._

When the dataset values correspond to points for the Turtle to move to, we can use absolute parameters by using Turtle's `goto(x, y)` function.  If these data points are modeled in polar coordinates, they would need to be converted into Cartesian coordinates. An example will be illustrated in the next section.

## 6.2 Example: Climate Spirals

We had mentioned in Chapter 1 that we can use Python Turtle to draw Climate Spirals. The Climate Spiral was developed by Ed Hawkins to visualize global warming over time [1]. The original graphic is an animated image. Using Turtle, we can also create an animated visualization to show the trend of global warming in a dynamic manner. Fig. 6.2 shows the final state of the Turtle's path.

![Figure 6.2](/bgdv-book/assets/images/book/climate_spiral_turtle_1880-2020_v01-b_60-40_viridis.png)  
_Figure 6.2. A Climate Spiral draw with Turtle, at the completion state. Data from 1880 to 2020 [2]._

The code for using Turtle to draw a Climate Spiral is below. The data is from NASA GISTEMP [2], and as we did in in Chapter 3, we read its CSV file into a Pandas DataFrame and select the data for the years 1880-2020. We will represent the data in polar coordinates (r, theta) so that r is the temperature anomaly and theta is the angle position of the month.

We do some data manipulation to rotate and reverse each row of data values so that the months advance in the clockwise direction and Jan is placed at the 12 o'clock position. At the end of each year, except for the last year, we add a connecting line segment from the current year's Dec value to the next year's Jan value. The line color for each year is determined by a selected colormap. Using a viridis color scheme results in an image that looks similar to Hawkins's Climate Spiral [1]. We can also try using a sequential grayscale colormap (see Exercise 6.1).

For each year, the `draw_year()` function draws a path using a different colormap value, with a higher value in the more recent years. We also put in a parameter `brightness_range` so that the values at the top of the range do not look washed out on a white background; e.g. this is needed when using a grayscale color map. The data in polar coordinates (r, theta) is converted to Cartesian coordinates (x, y) and these are used as absolute parameters in Turtle's `goto(x, y)` function to draw the line segments of the path.

As the Turtle circles around the months and through the years from 1880 to 2020, it spirals outward, indicating the trend of global warming.

```python
import turtle as t
import matplotlib as mpl
import math
import numpy as np
import pandas as pd

def rotate(l, n):
    return l[n:] + l[:n]

def draw_year(theta_months, year_vals, r0, scale, color):
    t.pencolor(color[0], color[1], color[2])
    for j in range(len(theta_months)):
        r = r0 + scale * year_vals[j]
        x = r * math.cos(theta_months[j])
        y = r * math.sin(theta_months[j])

        if j == 0:
            t.teleport(x, y)  # set starting postion
        else:
            t.goto(x, y)

filepath = "GLB.Ts+dSST.csv"  # data file from NASA GISTEMP [4]
df = pd.read_csv(filepath, skiprows=1)

# select years 1880-2020
df_filtered = df[(df["Year"] >= 1880) & (df["Year"] <= 2020)]
months = df.columns[1:13].tolist()
df_months = df_filtered[months].astype(float)
df_year = df_filtered["Year"].astype(int)

theta_months = np.linspace(0, 2 * np.pi, len(months), endpoint=False).tolist()
# reverse and rotate months to advance clockwise 
# and to place Jan at the 12 o'clock position 
theta_months.reverse()
theta_months = rotate(theta_months, -4)

# close a loop from current Dec to next Jan (if not at the last year)
theta_months_close = theta_months.copy()
theta_months_close.append(theta_months[0])

width, height = 320, 320
screen = t.Screen()
screen.setup(width, height)
t.speed(10)
t.pensize(1)
# pen color:
colormap = mpl.colormaps["viridis"]
brightness_range = 1.0

# parameters for transformation to screen coordinates
r0, scale = 60, 40

for i in range(df_year.size):
    value = i / (df_year.size - 1)
    color_value = value * brightness_range + (1.0 - brightness_range)
    color = colormap(color_value)

    year_vals = df_months.iloc[i].tolist()

    if i < (df_year.size - 1):
        year_vals.append(df_months.iloc[i + 1].iloc[0])
        draw_year(theta_months_close, year_vals, r0, scale, color)
    else:
        draw_year(theta_months, year_vals, r0, scale, color)

t.hideturtle()
t.mainloop()
```

In the next chapter, we will use Pyplot to make polar plots that has a polar coordinates grid and axis labels. We will also draw a Climate Spiral using Pyplot.

---

## Exercises

Ex.6.1. In the Climate Spirals code, instead of the "viridis" colormap, use a sequential "Greys" colormap, and also adjust the brightness range.

Ex.6.2. In the Climate Spirals code, instead of the viridis color scheme, use a diverging blue to red color scheme. _Hint_: Use the diverging Red-Blue ("RdBu") colormap with the scale flipped to Blue-Red (as we had done for the Climate Stripes in Chapter 1).
