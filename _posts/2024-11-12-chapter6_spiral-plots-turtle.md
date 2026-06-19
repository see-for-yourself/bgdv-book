---
layout: post
title:  "Chapter 6: Drawing Spiral Plots with Turtle"
date:   2024-11-12
categories: book
TOC_order: 6
copyright: "Copyright © 2024-2026, P. L. Chiu. All Rights Reserved."
---

Circular plots are suitable for data that have cyclical structure or seasonal patterns. These are also called *spiral plots* when the visualization looks like a spiral, or *polar plots* when a polar coordinate system is used.

With Turtle, it is intuitive to draw circular paths by incrementing the turning angle at each step, and we can also alter this setup to plot in polar coordinates. In this chapter, we demonstrate drawing circles and spirals with Python Turtle, and give an example for making a Climate Spiral visualization with animation. In the next chapter, we will show how to create polar plots using Pyplot.

## 6.1 Drawing Circles and Spirals

Turtle can draw circular paths by incrementally turning and stepping. If at each step it moves forward by a constant distance and turns by a constant angle, it will go around in a circle (Figure 6.1-left). With a slight modification, moving by a distance that is a constant scale multiplied by the previous step's distance, it draws a *logarithmic spiral* (Figure 6.1-right). This is a beautiful shape that often appears in nature (e.g. nautilus shell).

![Figure 6.1](/bgdv-book/assets/images/book/turtle_circle_archimedean_logarithmic_v03-a_crop.png)  
*Figure 6.1. Drawn with Turtle graphics: circle, Archimedean spiral, logarithmic spiral.*

The code for drawing the logarithmic spiral using Turtle is very simple. Note that when the scale is set to 1.0, it draws a circle.

```python
import turtle as t
nsegs = 230  # number of segments
distance = 0.440  # initial step
angle = 5
scale = 1.015

for i in range(nsegs):
    t.forward(distance)
    t.left(angle)
    distance *= scale

t.hideturtle()
t.mainloop()
```

For the spiral in the middle of Figure 6.1, called an *Archimedean spiral*, it is more complicated to draw the path by turning and moving forward at each step, because the formula for the stepping distance is not obvious. See Exercise 6.1. Rather, it is easier to use a polar coordinate system with points represented by (theta, r), in which theta is the angle and r the radius. Then Turtle can draw a path specified by a sequence of points in polar coordinates by converting them to cartesian coordinates (x, y) and using the Turtle function `goto(x, y)`.

An Archimedean spiral can be defined by the polar coordinate equation r = b*θ, where b is a constant. The Turtle code for drawing an Archimedean spiral is:

```python
import math
nsegs = 370
b = 6
angle = 0.05  # radians

for i in range(nsegs):
    theta = i * angle
    r = b * theta
    x = r * math.cos(theta)
    y = r * math.sin(theta)
    t.goto(x, y)
```

In the next section, we will apply these basic ideas to real-world data and use Turtle to make a Climate Spiral visualization.

## 6.2 Example: Climate Spirals

We had mentioned in Chapter 1 that we can use Python Turtle to draw Climate Spirals. The Climate Spiral was developed by Ed Hawkins to visualize global warming over time [1]. The original graphic is an animated image. Using Turtle, we can also create an animated visualization to show the trend of global warming in a dynamic manner. Figure 6.2 shows a screenshot of our Turtle program.

![Figure 6.2](/bgdv-book/assets/images/book/climate_spiral_turtle_1880-2020_v02-a_60-40_viridis.png)  
*Figure 6.2. A Climate Spiral draw with Turtle, at the completion state. Data from 1880 to 2020 [2].*

The first step is to load the climate data from NASA GISTEMP [2]. We read its CSV file into a Pandas data frame and select the data for the years 1880-2020:

```python
import pandas as pd
df = pd.read_csv("GLB.Ts+dSST.csv", skiprows=1)

# select years 1880-2020
df_filtered = df[(df["Year"] >= 1880) & (df["Year"] <= 2020)]
df_year = df_filtered["Year"].astype(int)  # column with years
months = df.columns[1:13].tolist()
df_months = df_filtered[months].astype(float)  # values to plot
```

We will represent the data in polar coordinates (theta, r) where theta is the angle position of the month and r is the temperature anomaly. Furthermore, in the layout of the coordinate axes and labels, we would like to place January at the 12 o'clock position, and have the path revolve in the clockwise direction. To facilitate this, we define a function to calculate the sequence of month angles for the plot line path to traverse in a single year:

```python
def month_angles():
    # setup months along clockwise direction and with Jan at 12 o'clock position
    # angles for the months are in radians
    theta_months = [(2 * math.pi * i / 12) for i in range(12)]
    theta_months.reverse()
    theta_months = theta_months[-4:] + theta_months[:-4]  # rotate list

    # for closing the loop from current Dec to next Jan (if not at the last year)
    theta_months_close = theta_months.copy()
    theta_months_close.append(theta_months[0])
    return theta_months, theta_months_close
```

Next, we define a function `draw_year()` for drawing a year's path with a specified colormap value; the idea is to use a higher value for the more recent years. The data in polar coordinates (theta, r) are transformed to Cartesian coordinates (x, y) and these are used as absolute positions in Turtle's `goto(x, y)` function to draw the line segments of the path.

```python
def draw_year(theta_months, year_vals, color):
    # parameters to adjust the shape of the spiral in screen coordinates
    r0, scale = 60, 40
    t.pencolor(color[0], color[1], color[2])
    for j in range(len(theta_months)):
        r = r0 + scale * year_vals[j]
        x = r * math.cos(theta_months[j])
        y = r * math.sin(theta_months[j])

        if j == 0:
            t.teleport(x, y)  # set starting postion
        else:
            t.goto(x, y)
```

Then the rest of the code for drawing a Climate Spiral is:

```python
import matplotlib as mpl
colormap = mpl.colormaps["viridis"]
brightness_range = 1.0  # use a lower value for grayscale colormaps
t.speed(10)
t.pensize(1)

theta_months, theta_months_close = month_angles()
for i in range(df_year.size):
    value = i / (df_year.size - 1)
    color_value = value * brightness_range + (1.0 - brightness_range)
    color = colormap(color_value)

    year_vals = df_months.iloc[i].tolist()
    if i < (df_year.size - 1):
        year_vals.append(df_months.iloc[i + 1].iloc[0])
        draw_year(theta_months_close, year_vals, color)
    else:
        draw_year(theta_months, year_vals, color)

t.hideturtle()
t.mainloop()
```

---

## Exercises

\*Ex.6.1. Use Turtle to draw an Archimedean spiral (Figure 6.1-middle) by turning and moving forward at each step (not using `goto(x, y)`). *Hint*: The incremental stepping distance can be calculated using the identity: \\(ds = \sqrt{r(θ)^2 + (dr/dθ)^2} dθ\\).

Ex.6.2. In the Climate Spirals code, instead of the "viridis" colormap, use a sequential "Greys" colormap, and also adjust the brightness range so that the early years do not get washed out on the white background.

Ex.6.3. In the Climate Spirals code, instead of the viridis color scheme, use a diverging blue to red color scheme. *Hint*: Use the diverging Red-Blue ("RdBu") colormap with the scale flipped to Blue-Red (as we had done for the Climate Stripes in Chapter 1).
