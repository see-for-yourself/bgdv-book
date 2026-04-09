---
layout: post
title:  "Chapter 6: Drawing Polar Plots with Turtle"
date:   2024-11-12
categories: book
TOC_order: 6
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

Polar plots (also called radial plots) are suitable for data that have cyclical structure or seasonal patterns. Typically, the data is modeled in polar coordinates, and the path of the plot revolves around the origin of the coordinate system.

With Turtle, it is intuitive to draw circular paths by incrementing the turning angle and moving forward at each step, and we can alter this setup to plot in polar coordinates. In this chapter, we demonstrate polar plotting with Python Turtle, and give an example for making a Climate Spiral visualization with animation. In the next chapter, we will show how to make polar plots using Pyplot.

## 6.1 Drawing a Polar Plot

Turtle can draw circular paths by incrementally turning and stepping. If at each step it turns by a constant angle and moves forward by a constant distance, it will go around in a circle (Figure 6.1-left). With a slight modification, moving by a distance that is a constant scale multiplied by the previous step distance, it draws a _logarithmic spiral_ (Figure 6.1-right). This is a beautiful shape that often appears in nature (e.g. nautilus shell).

![Figure 6.1](/bgdv-book/assets/images/book/turtle_circle_archimedean_logarithmic_v03-a_crop.png)  
_Figure 6.1. Drawn with Turtle graphics: circle, Archimedean spiral, logarithmic spiral._

The code for drawing the logarithmic spiral using Turtle is very simple. Note that when the scale is set to 1.0, it draws a circle.

```python
import turtle as t
segments = 230
angle = 5
scale = 1.015
distance = 0.440  # initial step

for i in range(segments):
    t.forward(distance)
    t.left(angle)
    distance *= scale

t.hideturtle()
t.mainloop()
```

For the spiral in the middle of Figure 6.1, called an _Archimedean spiral_, it is more complicated to draw the path by turning and moving forward at each step, because the formula for the stepping distance is not obvious. See Exercise 6.1. Rather, it is easier to use a polar coordinate system with points represented by (r, θ). Then Turtle can draw a path specified by a sequence of points in polar coordinates by converting them to cartesian coordinates (x, y) and using the Turtle function `goto(x, y)`.

An Archimedean spiral can be defined by the polar coordinate equation r = b*θ, where b is a constant. Then the Turtle code for drawing an Archimedean spiral is:

```python
segments = 370
b = 6
angle = 0.05  # radians

for i in range(segments):
    theta = i * angle
    r = b * theta
    x = r * math.cos(theta)
    y = r * math.sin(theta)
    t.goto(x, y)
```

In the next section, we will apply these basic ideas to real-world data and use Turtle to make Climate Spiral visualizations.

## 6.2 Example: Climate Spirals

We had mentioned in Chapter 1 that we can use Python Turtle to draw Climate Spirals. The Climate Spiral was developed by Ed Hawkins to visualize global warming over time [1]. The original graphic is an animated image. Using Turtle, we can also create an animated visualization to show the trend of global warming in a dynamic manner. Figure 6.2 shows the final state of the Turtle's path.

![Figure 6.2](/bgdv-book/assets/images/book/climate_spiral_turtle_1880-2020_v02-a_60-40_viridis.png)  
_Figure 6.2. A Climate Spiral draw with Turtle, at the completion state. Data from 1880 to 2020 [2]._

The code for using Turtle to draw a Climate Spiral is below. The data is from NASA GISTEMP [2], and as we did in in Chapter 3, we read its CSV file into a Pandas DataFrame and select the data for the years 1880-2020. We will represent the data in polar coordinates (r, theta) so that r is the temperature anomaly and theta is the angle position of the month.

We do some data manipulation to rotate and reverse each row of data values so that the months advance in the clockwise direction and Jan is placed at the 12 o'clock position. At the end of each year, except for the last year, we add a connecting line segment from the current year's Dec value to the next year's Jan value. The line color for each year is determined by a selected colormap. Using a viridis color scheme results in an image that looks similar to Hawkins's Climate Spiral [1]. We can also try using a sequential grayscale colormap (see Exercise 6.2).

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

\*Ex.6.1. Use Turtle to draw an Archimedean spiral (Figure 6.1-middle) by turning and moving forward at each step (not using `goto(x, y)`). _Hint_: The incremental stepping distance can be calculated using the identity: \\(ds = \sqrt{r(θ)^2 + (dr/dθ)^2} dθ\\).

Ex.6.2. In the Climate Spirals code, instead of the "viridis" colormap, use a sequential "Greys" colormap, and also adjust the brightness range.

Ex.6.3. In the Climate Spirals code, instead of the viridis color scheme, use a diverging blue to red color scheme. _Hint_: Use the diverging Red-Blue ("RdBu") colormap with the scale flipped to Blue-Red (as we had done for the Climate Stripes in Chapter 1).
