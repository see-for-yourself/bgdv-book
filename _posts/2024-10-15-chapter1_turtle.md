---
layout: post
title:  "Chapter 1: Drawing with Python Turtle"
date:   2024-10-15
categories: book
TOC_order: 1
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

A great place to begin learning about graphics and visualizatin is with _turtles_. Turtle graphics is a popular tool for computer education used in programming languages such as Scratch. It was developed as part of the Logo programming language in the 1960s by Wally Feurzeig, Seymour Papert and Cynthia Solomon [1, 2]. In Python, turtle graphics is included and there is no need to install extra libraries.

Drawing with a turtle is accomplished by giving it a sequence of steps specifying how far to move and how much to turn. When the program is run, the movements are animated. The process is like moving a pen around on a sheet of paper. It is highly intuitive and easy to use, yet it can draw quite sophisticated images.

In this chapter, we start with a simple Python turtle program to draw a staircase shape. Then we make up some data by using Python's random number generator to produce a more interesting staircase random walk. Moving on to real data, we use turtle to visualize the staircase-shaped trajectory of the Covid-19 vaccinations and cases in the US. As an example of creating more sophisticated visualizations with turtle, we show how to draw a "Climate Stripes" graphic of global warming data.

## 1.1 Hello Turtle

Let's begin by running a simple turtle program.

```python
import turtle as t

for i in range(12):
    t.forward(10)
    t.left(90)
    t.forward(10)
    t.right(90)

t.hideturtle()
t.mainloop()
```

To run this code, you need to have Python installed on your computer. See Appendix A for some information on installing Python. Since the turtle module is included in the Python standard library, you do not need to install additional packages.  However, if you don't have Python on your computer yet, you may want to install a Python distribution (like Anaconda) that includes other data science libraries that will be used in the later parts of this book.

This program draws a "staircase" figure with 12 steps by using the turtle commands to move forward by 10 units, and to turn left and right by 90 degrees. When the program is run, the drawing of the steps will be animated. The resulting image is shown in Figure 1.1.

![Figure 1.1](/assets/images/book/turtle_stairs_s12.png)  
_Figure 1.1. Screenshot of the finished state of our simple turtle program._

For data visualization, we obviously need data. One way to get data is to randomly generated it. Random data is very useful: It can be used for testing, benchmarking, performing experiments, checking statistical models, etc. In Python, the `random` module has functions to generate pseudo-random numbers. In particular, the function `random()` returns a number between 0 and 1 (inclusive of 0.0, exclusive of 1.0).

We can modified our program to draw a "staircase random walk" with random lengths for the steps. First, we add a line of code to import the `random` module, and we set the seed to a fixed number to produce the same results every time the program is run:

```python
import turtle as t
import random

random.seed(21)
...
```

Then we modify the calls to the turtle forward function to:

```python
t.forward(10*random.random())
```

To make the image look nicer, we increase the number of steps by setting the range parameter in the for loop to 24 instead of 12.

Run the program and the resulting image is shown in Figure 1.2.

![Figure 1.2](/assets/images/book/turtle_stairs_s12x2_rand-s21.png)  
_Figure 1.2. Screenshot of a staircase random walk with 24 steps._

## 1.2 Example: Trajectory of Covid-19 Vaccinations and Cases

Now let's look at some real data.

For the Covid-19 pandemic, there is a public dataset created by Johns Hopkins Univeristy (JHU) [3]. From cumulative data values at the end of each month, we can extract the monthly number of vaccinated people (with at least one dose) and the monthly number of cases. The data for vaccination began in December 2020, and the JHU dataset has month-end data up to February 2023. The World Health Organization (WHO) declared the end of COVID-19 as a global health emergency in May 2023 [4].

We can use Python turtle to sketch a trajectory of the monthly vaccinations and cases. For each successive month, we move the turtle horizontally by the number of vaccinations and vertically by the number of cases. When the program is run, the movement of the trajectory is animated. The resulting image is shown in Figure 1.3 (top).

![Figure 1.3](/assets/images/book/csse_us_turtle_vax1_cases_600x320_black_v02-b_crp.png)  
![Figure 1.3](/assets/images/book/csse_us_turtle_vax1_cases_slopes_600x320_black_v02-b_crp.png)  
_Figure 1.3. Trajectory of Covid-19 vaccinations and cases in the US for each month (2020-12 to 2023-02). Data Source: Johns Hopkins Univeristy [3]. Top: Horizontal segments represent the number of vaccinated people and vertical segments the number of cases. Bottom: Red segments connect the points at the end of each month._

The code including the data for making this visualization is as follows:

```python
import turtle as t
screen = t.Screen()
screen.setup(width=600, height=320) 
t.teleport(-280, -100)
t.speed(10)

# monthly data (in millions) for US, from 2020-12 to 2023-02
vax_monthly = [
    5.15, 24.41, 26.11, 50.69, 45.53, 21.54, 9.68, 9.67, 13.53, 8.69, 
    6.84, 11.5, 10.58, 7.33, 2.86, 1.59, 1.63, 1.3, 1.35, 1.71, 
    1.24, 1.44, 1.82, 1.34, 1.01, 0.61, 0.31]
cases_monthly = [   
    6.55, 6.14, 2.41, 1.82, 1.89, 0.92, 0.4, 1.33, 4.29, 4.14, 
    2.51, 2.55, 6.3, 20.34, 3.94, 1.04, 1.24, 2.86, 3.35, 3.67, 
    3.18, 1.83, 1.13, 1.32, 1.95, 1.6, 1.08]

scale = 2 # to fit in screen nicely
steps = len(vax_monthly)
for i in range(steps):
    t.forward(scale*vax_monthly[i])
    t.left(90)
    t.forward(scale*cases_monthly[i])
    t.right(90)

t.hideturtle()
t.mainloop()
```

In order to make enough space on the turtle screen to fit all the drawing steps, we set the turtle screen's width and height at the top of the program. The `teleport()` function is used to move the starting point to some distance on the lower left of the screen. Depending on your computer, these parameters can be adjusted to fit the visualizaton inside the turtle screen.

One enhancement is to add a trajectory line that connects the points at the end of each month (see Figure 1.3 bottom). We leave this as an execise (Ex.1.2). Looking at the figures, we see a "flattening" of the cases,  followed by a sharp rise, and then much fewer cases and vaccinations at the end of the dataset period on the right. This may lead the curious to search for explanations for the sharp rise, and learn about different Covid-19 variants and their transmissibility, about seasonal effects, etc. As for the association between the vaccinations and the cases, there may be many factors involved and it would be better to defer the interpretations to the experts.

## 1.3 Example: Climate Stripes

To raise awareness about climate change, compelling data visualizations have been designed for viewing by the general public. These info graphics have been widely used in print and online media and displayed in public spaces. One famous example is the _Climate Spiral_, which was featured in opening ceremonies of the Olympics in 2016. The Climate Spiral was developed by Ed Hawkins, who also designed the _Climate Stripes_ (or _Warming Stripes_); these visualize global warming over time [5, 6].

We can use Python turtle graphics to draw Climate Stripes and Climate Spirals. Figure 1.4 shows a Climate Stripes visualization drawn with turtle using the code below. The result looks similar to the Climate Stripes in [5]. Climate Spirals will be covered in a later chapter on radial plots.

![Figure 1.4](/assets/images/book/climate-stripes_wl-2x80_cmap-RdBu_v01-c_crop.png)  
_Figure 1.4. Climate Stripes from 1880 to 2020, rendered with a diverging blue-red color mapping. Data Source: NASA GISTEMP  [7]._

First, we need to get the climate data. One source is NASA GISTEMP [7]. This has Global Land-Ocean Temperature data. Specifically, we want the data for the combined land-surface air and sea-surface water temperature anomalies. The _temperature anomaly_ is defined as the temperature difference from a baseline period in degrees Celsius. We will visualize the data for the annual mean temperature anomalies for the years from 1880 to 2020. In the GISTEMP data, the baseline period is 1951-1980.

The NASA GISTEMP data is available on its website in CSV and TXT format. You can downloaded them as files onto your computer and view them using a text editor. Later, we will show how to read and extract selected data from CSV files using Python. Here, the temperature anomaly values (in 1/100 degrees Celsius) from 1880 to 2020 have been extracted and put into a Python list, under the import declarations:

```python
import turtle as t
import matplotlib as mpl

# subset of NASA GISTEMP data from 1880 to 2020
temp_anoms = [
    -18, -9, -11, -18, -29, -33, -32, -37, -18, -11, -36, -23, -28, -32, -31,
    -23, -12, -12, -28, -18, -9, -16, -29, -38, -48, -27, -23, -39, -43, -49,
    -44, -45, -36, -35, -15, -14, -36, -46, -30, -28, -27, -19, -28, -27, -27,
    -22, -11, -21, -20, -36, -15, -9, -15, -28, -12, -19, -14, -3, 0, -2,
    13, 19, 7, 9, 20, 9, -7, -3, -11, -11, -17, -7, 1, 8, -13,
    -14, -19, 5, 6, 3, -2, 6, 3, 5, -20, -11, -6, -2, -8, 5,
    3, -8, 1, 16, -7, -1, -10, 18, 7, 16, 26, 32, 14, 31, 16,
    12, 18, 32, 39, 27, 45, 41, 22, 23, 31, 45, 33, 47, 61, 38,
    39, 53, 63, 62, 53, 68, 64, 66, 54, 65, 72, 61, 65, 68, 75,
    89, 101, 92, 85, 97, 101]
```

Below this code containing the data, rest of the program for using the turtle to draw the Climate Stripes in Figure 1.4 is below:

```python
min_ta = min(temp_anoms)
max_ta = max(temp_anoms)
range_ta = max_ta - min_ta

# rescale the temperatue values [min_ta, max_ta] to [0.0, 1.0]
values = [(x - min_ta)/range_ta for x in temp_anoms]

screen = t.Screen()
screen.colormode(1) 
colormap = mpl.colormaps['RdBu']  # diverging Red-Blue

stripe_width, stripe_length = 2, 80
t.speed(10)
t.pensize(stripe_width)

# set starting postion to place figure at center of screen
t.teleport(-stripe_width *len(values)/2, -stripe_length/2)

for value in values:
    rgb = colormap(1.0 - value)  # flip the scale to Blue-Red
    # set postion and direction for next stripe
    t.penup()
    t.forward(stripe_width)
    t.left(90)
    t.pendown()

    # draw stripe
    t.pencolor(rgb[0], rgb[1], rgb[2])
    t.forward(stripe_length)

    # go back
    t.penup()
    t.left(180)
    t.forward(stripe_length)
    t.left(90)
    t.pendown()

t.hideturtle()
t.mainloop()

```

To map the temperature values onto a color scale, we import and use the `matplotlib` library module. The Python Matplotlib library requires installation (see Appendix A, or search online on how to install Matplotlib).

To map the temperature values onto a color scale, we choose the Diverging Red-Blue colormap using the code `colormap = mpl.colormaps['RdBu']`. We use color mode 1, which means that the input range for the colormap is from 0.0 to 1.0. This requires that we rescale the temperature values to the range from 0.0 to 1.0. An additional tweak is to flip the scale so that the higher temperatures are shown in red and the lower temperatures in blue.

Looking at the result in Figure 1.4, we see a trend in global warming from the stripes going from blue to red as the years go from 1880 to 2020.

---

## Exercises

Ex.1.1. Running the staircase random walk program in Chapter 1.1 will produce the same image each time since we had set the seed to a fixed number. Try running the program with different seeds to get different images.

Ex.1.2. For the trajectory of Covid-19 vaccinations and cases, use Python turtle to draw the visualization with the added lines that connect the points at the end of each month as in Figure 1.3 (bottom). _Hints_: The turtle functions `t.xcor()` and `t.ycor()` gets the current coordinates on the screen, and `t.goto(x, y)` moves to a specified point (x, y). The function `t.pencolor('red')` sets the pen color to red.

*Ex.1.3. Notice that the trajectory of Covid-19 vaccinations and cases in Figure 1.3 (top) is basically a staircase visualization. It has 27 steps. In the previous section, we drew random staircases with 24 steps -- do these look like the Covid-19 trajectory?  Draw several more random staircases with 27 steps, and compare these to the Covid-19 visualization. To what degree do you believe that the Covid-19 trajectory looks like it comes from random data?
