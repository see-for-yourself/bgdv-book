---
layout: post
title:  "Chapter 12: Animation with Pyplot"
date:   2026-05-28
categories: book
TOC_order: 12
copyright: "Copyright © 2024-2026, P. L. Chiu. All Rights Reserved."
---

Animation can be a compelling feature in visualizations by progressively revealing the data. It produces a cool effect, although it could be distracting due to the motion. A limitation is that it only works on screens and not on paper printouts.

Animation is the default mode for Turtle graphics. Pyplot can handle animation but the process is more complicated. To make a Pyplot animation using the `matplotlib.animation` module, we create a Pyplot figure and define a function which is called to update a frame during the animation. In this function, we do a plot for that frame. In the program, the figure and the function are passed to `animation.FuncAnimation()`, which has additional parameters to specify the time interval between frame updates, the range of frames to render, etc. Also, an initialization function can be optionally defined which is executed before the animation runs. For sharing or incorporating into digital documents, the `matplotlib.animation` module can output the animated plot in various formats such as GIF image files.

## 12.1 Example: Basketball Free Throw Trajectory

We begin with an example of a basketball trajectory for a free throw shot. The shooter releases the ball from a height of about 7 feet and the center of the basket is 10 feet high and 13.75 feet away. From basic physics, an object thrown travels along a parabolic path due to the force of gravity. Suppose we have the following data from a model (which will be discussed in the next section) for the horizontal distance and height (in feet) at time intervals of 0.1 seconds:

```text
x = [0.0, 1.2, 2.4, 3.6, 4.8, 6.0, 7.2, 8.4, 9.6, 10.8, 
     12.0, 13.2, 14.4, 15.6]
y = [7.0, 8.92, 10.52, 11.8, 12.75, 13.39, 13.71, 13.71, 13.39, 12.75, 
     11.78, 10.5, 8.9, 6.98]
```

To create an animation of the trajectory from these points, we proceed as follows. First, we setup the figure. We also draw two dashed lines that cross at the target point (13.75, 10). Then we create a line object by making an empty plot and specifying the plot parameters like line width and color -- this line object will be updated during the animation.

Next, we define the `init()` and `update_frame()` functions, and pass these to `animation.FuncAnimation()`, along with parameters to specify the number of frames and the time interval bewteen frames (in milliseconds). To render the animation, we call `plt.show()`. We also call the save function to output an animated GIF image. The code and the GIF image are below:

```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation

fig, ax = plt.subplots() 
ax.set_aspect("equal")
ax.set(xlim=[0, 15], ylim=[0, 16], xlabel="x (ft)", ylabel="y (ft)")

plt.axvline(x=13.75, color="lightgray", linestyle="--")
plt.axhline(y=10, color="lightgray", linestyle="--")

line, = ax.plot([], [], lw=2, color="tab:orange")

def init():
    # clear the line
    line.set_data([], [])
    return line,

def update_frame(i):
    # draw up to i points
    line.set_data(x[:i], y[:i])
    return line,

ani = animation.FuncAnimation(
    fig,
    update_frame,
    init_func=init,
    frames=len(t),
    interval=100,
)

plt.show()
ani.save(filename="freethrow_ani.gif", writer="pillow")
```

![Figure 12.1](/bgdv-book/assets/images/book/freethrow_err_ani_v02-a_w480_dt-0.1.gif)  
*Figure 12.1. Animated GIF image of a trajectory using data from a simple physics model of a basketball free throw.*
<!--
![Figure 12.1](/bgdv-book/assets/images/book/freethrow_err_ani_v02-a_w480_dt-0.1_last-frame.png)  
*Figure 12.1. Animated GIF image of a trajectory using data from a simple physics model of a basketball free throw.*
-->

## 12.2 Animating Line Collections

Matplotlib can handle the animation of a collection of line segments. In particular, this allows for progressively showing a bunch of line plots, or for breaking up a single line plot into subsegments which can be set with different properties. We illustrate the former in this section with the basketball free throw example, and the latter in the next section with the Covid-19 polar plot example.

First, we describe a basketball free throw model to generate different trajectories. A simple way to do this is to start with the basic physics equation for a parabolic trajectory and add an error term based on a Gaussian distribution:

&emsp; *y = ax² + bx + c + epsilon,*

where the coefficients *a* and *b* depend on the initial angle and initial velocity, and *c* is the initial height. The explicit formulas are in the code below, and the parameters are those for shooting a free throw. (Note that for simplicity, we are not directly taking into account air resistance, spin, etc. And we are only considering a 2D model.)

For the *epsilon* error term, we use a Gaussian random generator `random.gauss(mu, sigma)` in the code. Running the model equation multiple times will produce a set of different trajectories. For our model, we set the parameters `mu` (mean) and `sigma` (standard deviation) to some reasonable values: 0 for the mean and 0.3 (i.e. 4 inches) for the standard deviation. Real-world data can also be collected to compute the parameters in our model equation.

Here is the code for our model to generate a set of trajectories:

```python
# create time points for a duration of 2 sec with a step of 0.1 sec
t = np.linspace(0, 2, 20, endpoint=False)

# model trajectory by a parabola using a quadratic equation with an error term
# y = ax² + b + c + epsilon
g = 32   # acceleration due to gravity (ft/s²)
v0 = 24  # initial velocity
theta0 = math.radians(60)  # initial angle
y0 = 7   # release height

# calculate the x values
x = v0 * t * math.cos(theta0)

# generate a set of the y values, which is equivalent to:
# y = v0 * t * math.sin(theta0) - g * t**2 / 2 + y0 + epsilon
a = -0.5 * g / (v0 * math.cos(theta0))**2
b = math.tan(theta0)
mu, sigma = 0, 0.3

y_data = []
nrows = 20
for i in range(nrows):
    epsilon = random.gauss(mu, sigma)
    y = a * x**2 + b * x  + y0 + epsilon
    y_data.append(y)
```

If we progressively plot one trajectory per frame in an animation, it would look like Figure 12.2:

![Figure 12.2](/bgdv-book/assets/images/book/freethrow_LC_ani_v02-a_delay-200_sigma-0.3_s-42_n-20.gif)  
*Figure 12.2. Animated GIF image of 20 trajectories from our model of a basketball free throw (mu=0, sigma=0.3).*
<!-- 
![Figure 12.2](/bgdv-book/assets/images/book/freethrow_LC_ani_v02-a_delay-200_sigma-0.3_s-42_n-20_last-frame.png)  
*Figure 12.2. Animated GIF image of 20 trajectories from our model of a basketball free throw (mu=0, sigma=0.3).*
-->

To make this plot, we construct a `LineCollection` object `lc` with empty line segments and set its parameters for colors and for linewidths. In the animation `update_frame()` function, we set the line segments of `lc` to be plotted in that frame.

These line segments are structured as a nested list: it is a list that contains a list of line segments, and each line segment is a list of points, and each point is a list with two elements x and y. The elegant way to do this is with the Numpy `column_stack()` function:

```python
segments = []
for y in y_data:
    seg = np.column_stack((x, y))
    segments.append(seg)
```

Then the rest of the code for making the animation in Figure 12.2 is:

```python
from matplotlib.collections import LineCollection

tab_orange_alpha = '#ff7f0e77'
lc = LineCollection([], colors=tab_orange_alpha, linewidths=1)
ax.add_collection(lc)

def init():
    # clear the segments
    lc.set_segments([])
    return lc,

def update_frame(i):
    # draw up to i trajectories
    lc.set_segments(segments[: i + 1])
    return lc,

ani = animation.FuncAnimation(
    fig,
    update_frame,
    init_func=init,
    frames=len(segments),
    interval=200,
)

plt.show()
ani.save(filename="freethrow_LC_ani.gif", writer="pillow")
```

## 12.3 Example: Covid-19 Polar Plot with Animation

In this section, we create an animated polar plot for the Covid-19 data. In an earlier chapter, we had made a static polar plot of the Covid-19 deaths using Pyplot (Figure 7.2). This plot has a different color (grayscale value) for each year's line, which makes the animation more complex. See Figure 12.3. To construct the animated plot, we can use the `LineCollection` object like we did in the previous section, but breaking up the single line in the plot into subsegments by year and assigning a different color to each subsegment.

![Figure 12.3](/bgdv-book/assets/images/book/polar-plot_ani_US_pointvals_monthly-mean_deaths_v02-a.gif)  
*Figure 12.3. Animated GIF image of a polar plot of Covid-19 deaths. Data source: Johns Hopkins CSSE [1].*
<!-- 
![Figure 12.3](/bgdv-book/assets/images/book/polar-plot_ani_US_pointvals_monthly-mean_deaths_v02-a_last-frame.png)  
*Figure 12.3. Animated GIF image of a polar plot of Covid-19 deaths. Data source: Johns Hopkins CSSE [1].*
-->

To make this animation shown in Figure 12.3, first we load the Covid-19 data that we had wrangled into monthly mean values and saved into a CSV file back in Section 5.3:

```python
df = pd.read_csv("deaths/US_pointvals_monthly-mean.csv")
df_year = df["Year"].astype(int)
months = df.columns[1:13].tolist()
df_months = df[months].astype(float)
```

Then we construct the segments and assign their colors by year to be used in the `LineCollection` object. These are actually the subsegments of the single line in the plot.

```python
theta_months, theta_months_close = month_angles()  # function from Chapter 6
segments, seg_colors = [], []
colormap = mpl.colormaps['Greys']
brightness_range = 0.75

for i in range(df_year.size):
    value = i / (df_year.size - 1)
    color_value = value * brightness_range + (1.0 - brightness_range)
    color = colormap(color_value)

    # make theta values decreasing through the years
    year_theta_vals = [(t - i * 2 * math.pi) for t in theta_months]
    year_vals = df_months.iloc[i].tolist()

    # connect Dec to Jan (of next year)
    if i < (df_year.size - 1):
        close_year_theta_val = year_theta_vals[0] - 2 * math.pi
        year_theta_vals.append(close_year_theta_val)

        close_year_val = round(float(df_months.iloc[i+1].iloc[0]), 1)
        year_vals.append(close_year_val)

    # create one segment for each month,
    # with the same color for each year
    for j in range(len(year_theta_vals) - 1):
        # create a segment for each month
        # by connecting it to the next month
        month_seg = [[year_theta_vals[j], year_vals[j]],
                     [year_theta_vals[j + 1], year_vals[j + 1]]]
        segments.append(month_seg)
        seg_colors.append(color)
```

After that, we setup the figure:

```python
fig = plt.figure()
ax = fig.add_subplot(111, projection="polar")

# set axis limits, ticks and tick labels
overall_max = df_months.max().max()
ax.set(ylim=[0, 1.05 * overall_max])

rticks=[1000, 2000, 3000]
rlabel_position=-80
ax.set_rticks(rticks) 
ax.set_rlabel_position(rlabel_position)

xticks, xticks_lbls = theta_months, months
ax.set_xticks(xticks, labels=xticks_lbls)
```

Finally, we setup the animation functions and render the plot, and also save it to an animated GIF image.

```python
lc = LineCollection([], linewidths=2)
ax.add_collection(lc)

def init():
    # clear the segments
    lc.set_segments([])
    return lc,

def update_frame(i):
    # draw up to i segments
    lc.set_segments(segments[: i + 1])
    lc.set_color(seg_colors[: i + 1])
    return lc,

ani = animation.FuncAnimation(
    fig,
    update_frame,
    init_func=init,
    frames=len(segments),
    interval=200,
)

plt.show()
ani.save(filename=filepath, writer="pillow")
```

---

## Exercises

*Ex.12.1. Under NBA regulations [1], the basketball rim's center is at x = 13.75 feet, and it's radius is 9 inches. The ball radius is 4.7 inches. Modify the code in Section 12.1 to compute whether the basketball is inside the rim at y = 10 feet. (Note with our simplified model the trajectory is in the 2D xy-plane.) Use different colors for the trajectory lines depending on whether it goes through inside or outside of the rim.

Ex.12.2. Make a Climate Spiral with animation, similar to the way we made the Covid-19 polar plot with animation (Figure 12.3). *Hint:* See Section 6.2 for how to get the climate data into the data structure that is used to make Figure 12.3.

Ex.12.3. Output the Climate Spiral animation (Figure 12.3) to an MP4 video file. *Hint:* Use the `animation.FFMpegWriter()` function.
