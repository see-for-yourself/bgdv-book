---
layout: post
title:  "Chapter 8: Focus-plus-Context: Composite Plots"
date:   2025-03-14
categories: book
TOC_order: 8
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

A very effective visualization design is using _focus-plus-context_ to show a zoomed-in focus region that is connected to a context region. As an example, Figure 7.1 shows a visualization of the NASA GISTEMP temperature anomaly data with a focus region on a part of the time interval. In this chapter, we show how to use Pyplot to make composite plots with subplots that are connected together to construct a focus-plus-context plot. Then we apply this to an example with climate data to create the image in Figure 7.1. After that, we consider a more complex example with a Koch Snowflake, which uses Turtle to generate the Koch Snowflake data.

![Figure 8.1](/bgdv-book/assets/images/book/cutout-plot_mosaic_1933-1967_temp_anoms_v03-a_dpi-75.png)  
_Figure 8.1. Temperate anomaly (Celcius) with focus region from 1933-1967 and context region from 1880-2020. Data source: NASA GISTEMP [1]._

## 8.1 Constructing a Focus-plus-Context Visualization with Pyplot

In order to make subplots with different sizes, the layout must be designed and specified. A nice way to do this in Pyplot is to employ the mosaic layout feature, which supports designing layouts based on "ASCII Art". To create the layout in Figure 8.1, the ASCII Art design looks like this:

```text
.AAA.  
.AAA.  
BBBBB
```

which specifies a figure containing a rectangular subplot 'A' with some empty space '.' on the left and right, sitting above another rectangular subplot 'B' that is shorter and wider and spanning the width of the plot. Intuitively, one can see a blob of A's and a blob of B's, along with their shapes and placements.

The corresponding Python code is:

```python
fig, axes = plt.subplot_mosaic('.AAA.;.AAA.;BBBBB', figsize=(6, 4.5))

plt.tight_layout()
plt.show()
```

![Figure 8.2](/bgdv-book/assets/images/book/step1_create_mosaic_layout_0.4-0.6_v03-a_dpi-75.png)  
_Figure 8.2. Creating a mosaic layout._

The next step is to draw the connecting lines between the focus region 'A' and the context region 'B' as shown in Figure 8.3-left:

![Figure 8.3a](/bgdv-book/assets/images/book/step2_mosaic_draw_connecting_lines_0.4-0.6_v03-a_dpi-50.png)
![Figure 8.3b](/bgdv-book/assets/images/book/step3_mosaic_fill_patch_0.4-0.6_v03-a_dpi-50.png)  
_Figure 8.3. Left: Drawing the connecting lines. Right: Filling in the focus region._

First, we write a function that draws a single connecting line between two subplots in a figure specified by the two subplot axes and the two end points. It uses a `ConnectionPatch` object, with the `coordsA` and `coordsB` parameters specifying the use of data units with the coordinate systems. We also specify using a gray dotted line of width 1.

```python
import matplotlib.patches as patches

def draw_connecting_line_subplots(fig, axesA, axesB, xyA, xyB):
    # draw a line by creating a ConnectionPatch
    con = patches.ConnectionPatch(xyA=xyA, xyB=xyB, 
                        coordsA='data', coordsB='data',
                        axesA=axesA, axesB=axesB,
                        color='gray', linestyle=':', linewidth=1)
    
    # add the ConnectionPatch to the figure
    fig.add_artist(con)
```

Then we use this function to put in the connecting lines between the focus subplot 'A' and context subplot 'B', by adding the following code under the line of code for creating a mosaic layout in the program above.

```python
# specify the focus patch boundaries
patch_left, patch_right = 0.4, 0.6
patch_bottom, patch_top = 0.0, 1.0

# specify the context subplot boundaries that are needed
context_bottom, context_top = 0.0, 1.0

# set x and y limits on the focus subplot
axes['A'].set_xlim(patch_left, patch_right)
axes['A'].set_ylim(patch_bottom, patch_top)

# draw connecting lines between the patch in context subplot 
# and the focus subplot 
xyA = (patch_left, patch_bottom)
xyB = (patch_left, context_top)
draw_connecting_line_subplots(fig, axes['A'], axes['B'], xyA, xyB)
    
xyA = (patch_right, patch_bottom)
xyB = (patch_right, context_top)
draw_connecting_line_subplots(fig, axes['A'], axes['B'], xyA, xyB)
```

The final step (before plotting any data) is to fill in the focus region in the context subplot to highlight it as shown in Figure 8.3-right. This uses a `patches.Rectangle` object. We add the following code under the code for drawing the connecting lines.

```python
# fill patch in context subplot
xy = (patch_left, patch_bottom)
width, height = (patch_right - patch_left), (patch_top - patch_bottom)
rect = patches.Rectangle(xy, width, height, 
                         edgecolor='gray', facecolor='lightgray')
# add to the context axis
axes['B'].add_patch(rect)
```

Now the figure is ready for plotting the data in the subplots.

## 8.2 Example: Climate Data

Using the mosaic layout from the previous section with some minor enhancements, we plot the NASA GISTEMP temperature anomaly data to obtain the focus-plus-context visualization in Figure 8.1.

For the focus region, we select a time interval (1933-1967) around the mid-twentieth century in which an interesting phenomenon has been noticed. During this period, there was a leveling off of temperatures, and climatologists have proposed various explanations for this phenomenon [2].

The other x and y limits of the subplots are specified based on the range of values in the dataset. This can be done manually by printing out the min and max values of the dataset, and trying out different values to determine what looks good. A choice of good values for the subplot y-limits are -0.6 and 1.1.

```python
x_vals = range(1880, 2021)
y_vals = temp_anoms  # from Chapter 1

# compute the min and max
xmin, xmax = min(x_vals), max(x_vals)
ymin, ymax = min(y_vals), max(y_vals)
print('xmin, xmax =', xmin, ',', xmax)
print('ymin, ymax =', ymin, ',', ymax)
```

Proceeding along, we modify the program for constructing a mosaic layout in the previous section to specify the limits of the focus and context regions:

```python
# specify focus patch and context limits
patch_left, patch_right = 1933, 1967  # mid-century
patch_bottom, patch_top = -0.6, 1.1

context_left, context_right = 1870, 2030
context_bottom, context_top = -0.6, 1.1

# set limits on the focus and context subplots
axesA.set_xlim(patch_left, patch_right)
axesA.set_ylim(patch_bottom, patch_top)

axesB.set_xlim(context_left, context_right)
axesB.set_ylim(context_bottom, context_top)
```

Next, we add the code below to plot the data. We also make one enhancement (see Figure 8.1) to help further convey the zooming effect by setting the context subplot to use a thinner line width and a smaller font size for its axis labels.

```python
# plot the data
axesA.plot(x_vals, y_vals, color='black')
axesB.plot(x_vals, y_vals, color='black', linewidth=0.75)
axesB.tick_params(axis='both', which='major', labelsize=8)
```

## 8.3 Example: Koch Snowflake

The Koch Snowflake is an interesting mathematical object that has a resemblance to real snowflakes. See Figure 8.4. It is a _fractal_ that is self-similar at different scales. This means that when you zoom in or zoom out, it looks similar. Other fractals resemble a wide variety of natural real-world shapes such as coastlines, mountains, and clouds [3].

There is no simple formula to define a Koch Snowflake; however, it is easy to construct via an iterative algorithmic procedure. This can be done using Turtle, and a screenshot of a drawing with five iterations in shown in Figure 8.4.

![Figure 8.4](/bgdv-book/assets/images/book/KochSnowflake_len-243_iter-5_v02-b_300x315.png)  
_Figure 8.4. A Koch Snowflake drawn with Turtle (iterations = 5)._

The code for constructing a Koch Snowflake is below. The number of iterations and the length determining the distances of the Turtle's steps are parameters that can be specified. At each step of the Turtle's traversal, we save the position's coordinates (x, y), and the resulting list contains the data for a Koch Snowflake.

```python
import turtle as t
points = []  # for saving the position points

t.speed(0)  # no animation, fastest run time
t.hideturtle()

def koch_curve(t, iterations, length):
    if iterations == 0:
        t.forward(length)
        points.append(t.position())
    else:
        for angle in [60, -120, 60, 0]:
            koch_curve(t, iterations - 1, length/3)
            t.left(angle)

def koch_snowflake(iterations, length):
    # Koch snowflake comprises three Koch curves
    for i in range(3):
        koch_curve(t, iterations, length)
        t.right(120)

points.append(t.position())  # initial position
koch_snowflake(iterations=5, length=243)
t.mainloop()
```

To make the focus-plus-context visualization of a Koch Snowflake in Figure 8.5, we use the mosaic layout:

```python
fig, axes = plt.subplot_mosaic('AAAAA;.BBB.;.BBB.', figsize=(6, 4.5))
```

The rest of the code is similar to the example with the Climate Data in the previous section, which we leave to the exercises.

![Figure 8.5](/bgdv-book/assets/images/book/cutout-plot_koch-snowflake_len-243_iter-5_v01-b.png)  
_Figure 8.5. Koch Snowflake and a piece of the Koch function._

The Koch Snowflake fractal provides an example of data that has detail at multiple resolutions. Geographic maps are another kind of shapes with fractal-like properties that can benefit from focus-plus-context visualization. We will cover geospatial visualization in the next two chapters, and one of the examples will be a focus-plus-context example with geographic data.

## Exercises

Ex.8.1. Make the focus-plus-context Koch Snowflake (Figure 8.5), using the Turtle generated data and the mosaic layout in Section 8.3.

*Ex.8.2. Observations for cherry blossom bloom have been recorded in Kyoto since the 9th century. Over this period, it is difficult to see a pattern for the day of year (DOY) of the first bloom. However, since the Industrial Revolution (c.1760–1840), the first bloom appears to have come earlier in the year. Using the data source below for cherry blossom bloom (812-2015), make a focus-plus-context plot of the data points, with the focus region on the years since the end of the Industrial Revolution. Also, plot a curve of the smoothed data in the focus region.

Cherry blossom phenology and temperature reconstructions at Kyoto. <http://atmenv.envi.osakafu-u.ac.jp/aono/kyophenotemp4> (also available at: cherry_blossoms <https://www.tensorflow.org/datasets/catalog/cherry_blossoms>)
