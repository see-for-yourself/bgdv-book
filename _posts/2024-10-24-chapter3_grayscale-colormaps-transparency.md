---
layout: post
title:  "Chapter 3: Grayscale, Colormaps, Transparency"
date:   2024-10-24
categories: book
TOC_order: 3
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

Grayscale images are versatile as they can be produced for a variety of digital and print media that may or may not support color. Color images have a certain appeal; however, they are more complicated to use effectively for data visualization. In this book we employ mostly grayscale images for their versatility, and the techniques discussed in this chapter will be used throughout the book.

For both grayscale and color, the aesthetic aspects are to a great extent subjective and personal. The perceptual aspects have been studied and there are design principles applicable to data visualization [1]. One important technique is the use of perceptually uniform colormaps. For numerical data, these map data values to color (or grayscale) values so that a difference on the data value scale is perceived as the same as a difference on the color value scale. It is notable that a sequential grayscale is more effective than a sequential color scale. For visualizing categorical variables, marker shapes and colors with the same level of saliency can be employed.

In this chapter, we begin with a discussion of grayscale and colormaps, and show an example of applying different colormaps to the Climate Stripes visualization. Then we go over transparency, followed by marker shapes and colors. We end the chapter with a longer example of visualing extreme rainfall events in Bengaluru.

## 3.1 Grayscale and Colormaps

A standard way to represent color on a computer is the RGB color model, with three components being red, green, and blue. A gray value has all three components equal (with the lowest value being black and the highest being white). The range of the component values can be set to either [0, 255] or [0.0, 1.0] depending on the mode.

The naive method of mapping data values to the grayscale values (or color values) is to use a linear map from the data coordinates to the RGB component values. A better method is to employ a perceptually uniform grayscale map (or color map). These maps are designed so that humans perceive an equal change in the grayscale space (or color space) for each step change in the data space.

We can compare a linear RGB mapping to a perceptally uniform grayscale sequential mapping by looking at them next to each other. See Figure 3.1. There is a slight but noticeable difference: the perceptual scale appears less dark on the left half and more balanced overall.

![Figure 3.1](/bgdv-book/assets/images/book/grayscale-stripes_wl-2x20_v02-c_crp.png)  
_Figure 3.1. Grayscale mappings over the interval [0.0, 1.0]. Top: Linear in RGB. Bottom: Sequential colormap ('Greys')._

With Python, there are various ways to specify a _color_ or to choose a _colormap_ [3]. A color can be specified by an RGB value or by a name. Colormaps provide a mapping from the data values to the color values, and a number of well-desgined grayscale and color mappings are available in the Matplotlib library. For a listing of the available colormaps, see [3].

To run the code in this chapter, the Matplotlib library needs to be installed in your Python setup (see Appendix A).

## 3.2 Example: Climate Stripes with Different Colormaps

We can try out different colormaps with the Climate Stripes visualization. A description of the visualization and the Python code is given in Chapter 1.3. In that code, the line

```python
colormap = mpl.colormaps['RdBu']
```

specifies by name the 'RdBu' colormap. The resulting image from Chapter 1.3 is reproduced here in Figure 3.2 top, and it looks like the original visualization by Ed Hawkins. Note that 'RdBu' is not a sequential colormap, rather it is a type of _diverging_ colormap. Diverging means that there is a critical point such that on the left it is one color and on the right it is another color. In Figure 3.2 top, the critical point corresponds to the middle value of the range of data values. Another sensible choice for the critical point is the zero data value (see Exercise 3.1).
  
Another popular choice for data visualization is the 'Viridis' colormap (Figure 3.2 middle); it is a perceptually uniform sequential colormap. For a grayscale Cimate Stripes, a perceptually uniform grayscale 'Greys' colormap is used in Figure 3.2 bottom. Studies have shown that sequential grayscale is better than sequential color in terms of accurate visual perception [1], although some might find it aesthetically less pleasing.

![Figure 3.2a](/bgdv-book/assets/images/book/climate-stripes_wl-2x80_cmap-RdBu_v01-c_crop.png)  
![Figure 3.2b](/bgdv-book/assets/images/book/climate-stripes_wl-2x80_cmap-viridis_v01-d_crop.png)  
![Figure 3.2c](/bgdv-book/assets/images/book/climate-stripes_wl-2x80_greys_v01-c_crop.png)  
_Figure 3.2. Climate stripes with different colormaps. Top: Diverging ('RdBu', reversed). Middle: Perceptually Uniform Sequential ('Viridis'). Bottom: Sequential ('Greys')._

## 3.3 Transparency and Alpha Channel

Applying transparency enables layering and helps deal with occlusions in visualizations. As an example, look at the two scatter plots of random points without and with transparency in Figure 3.3. Without transparency, overlapping points can turn into blobs that makes it harder to see the individual points. Transparency can also reveal points that are directly on top of one another. If you look carefully at the right plot in Figure 3.3, you can see that the point at (28, 17) is darker, which is caused by layering two transparent points with the same coordinates.

![Figure 3.3](/bgdv-book/assets/images/book/transparency_alpha-0.50_v01-a_crop.png)  
_Figure 3.3. Random points. Left: Gray. Right: Gray with alpha = 0.5._

The _alpha channel_ controls the transparency. For the RGB color model, the alpha channel is a fourth component that extends it to the RGBA model. Transparency is supported in Pyplot, but not currently in Turtle.

To use the alpha channel in Pyplot, set the `alpha` parameter in the `plot()` or `scatter()` functions. The `alpha` can be set between 0.0 and 1.0. In the example of Figure 3.3, the code is as folllows:

```python
# generate some random points
random.seed(42)  # a fixed seed to produce the same results
x, y = [], []
for i in range(100):
    x.append(random.randrange(100))
    y.append(random.randrange(100))

# create a figure with 1 row and 2 columns
fig, axes = plt.subplots(1, 2)

# plot left subplot
axes[0].scatter(x, y, color='gray')
axes[0].set_aspect('equal')

# plot right subplot
axes[1].scatter(x, y, color='gray', alpha=0.5)
axes[1].set_aspect('equal')
plt.show()
```  

## 3.4 Marker Shapes and Color

For visualizing groups or categories of data points, different marker shapes or different colors can be assigned to each group. There are multifarious design considerations when using marker shapes and colors. To illustrate some of the issues, we can make a plot using different shapes and a plot using different colors, with 4 groups of 10 random points. See Figure 3.4.

One issue is that the groups should have a similar level of saliency, unless you intend to highlight a specific group. For the shapes in Figure 3.4 left, the circles and polygons have similar saliency, while the X-shaped markers pop out. For the colors in Figure 3.4 right, the colors (blue, orange, green) from a colormap designed for categorical data have similar saliency, while the cyan color markers pop out.

![Figure 3.4](/bgdv-book/assets/images/book/marker-types_color_v04-a_crop.png)  
_Figure 3.4. Visualing 4 groups. Left: Different shapes in gray. Right: Different colors._

The code for Figure 3.4 is below. For the left plot, the shapes are specified by the line `marker_types`, and all markers are gray. There are other marker types available in Matplotlib; see [2]. For the right plot, the colors are specified by the line `colors`, and all the shapes are the same. The color prefix "tab:" stands for the Tableau Colors scheme (T10). For more details about specifying colors and about other colormaps schemes for categorical data, see [3].

Another issue to be mindful of is color blindness. About 10% of males and 1% of females are color blind [4]. It helps these people to use a color scheme that does not include both green and red [5].

```python
# generate groups of random points
random.seed(42)  # a fixed seed to produce the same results

n_groups = 4
groups_x, groups_y = [], []
for k in range(n_groups):
    x, y = [], []
    for i in range(10):
        x.append(random.randrange(100))
        y.append(random.randrange(100))
    groups_x.append(x)
    groups_y.append(y)

# create a figure with 1 row and 2 columns
fig, axes = plt.subplots(1, 2)

# plot left subplot
marker_types = ['o', 's', '^', 'x']
for k in range(n_groups):
    axes[0].scatter(groups_x[k], groups_y[k], marker=(marker_types[k]), 
                    color='gray')
axes[0].set_aspect('equal')

# plot right subplot
colors = ['tab:blue', 'tab:orange', 'tab:green', 'cyan']
for k in range(n_groups):
    axes[1].scatter(groups_x[k], groups_y[k], color=colors[k])
axes[1].set_aspect('equal')
plt.show()
```  

## 3.5 Example: Rainfall Data in Bengaluru

Let's look at an example with real data. In this example, we visualize extreme rainfall events at Bengaluru (also known as Bangalore). There is no single standard definition of an extreme rainfall event. In addition to the the amount of rain in a day, another important consideration is how it compares to the seasonal amount. A good benchmark is a month's worth of rain falling in one day in that season at that location (see [6]).

Figure 3.5 is a scatter plot that shows the top 1% of the days with precipitation recorded by a weather station (IN009010100) in Bengalaru (Bangalore) from 1901-2020, using data from NOAA [7]. Each day is represented by a triangular marker. The size of a triangle represents extremeness: it is proportional to the average daily amount of rain in that point's month. A triangular marker points up if it is considered an extreme event. By the benchmark above, an extreme event exceeds a month's worth of rain, which we calculate by taking the average rainfall of its month over the years in the dataset for Bengalaru.

Looking closer at Figure 3.5, the day with the second highest rainfall (its month is March when it rains only a little) has a much larger marker size than the day with the highest rainfall (its month is September when it rains a lot). Another interesting point is the highest downward pointing triangle at 1994-09-13. It has over 6 inches of rain -- this is a lot of rain in a single day! However, by the benchmark, it is not considered to be an extreme event. These and other similar points in the visualization show clearly that a day with a large amount of rain may not be as extreme as a day with less rain when they occur in differnet seasons.

![Figure 3.5](/bgdv-book/assets/images/book/plot_Bangalore_1901-2020_top_prcp_extreme-markers_alpha-0.25_gray_6x3_v03-a.png)  
_Figure 3.5. The size of a point is proportional to the average rainfall of its month. The upward pointing triangular markers means that they are extreme by the benchmark above. The markers are gray with alpha = 0.25. Data source: NOAA [7]_

Another view to delineate the extreme from non-extreme events can be created by plotting the ratio of a point's rainfall to its month's average rainfall. See Figure 3.6. If the ratio is greater than 1, it is an extreme event by the benchmark above. Using a log scale on the y-axis provides better spacing for the points below 1 and makes the visualization more legible. Using transparent points also works well here in terms of aesthetics as they are delightfully evocative of rain drops.

![Figure 3.6](/bgdv-book/assets/images/book/plot_Bangalore_1901-2020_top_extreme_prcp_alpha-0.25_gray_6x3_log-scale_v03-b.png)  
_Figure 3.6. The size of a point is proportional to its amount of rainfall. The markers are gray with alpha = 0.25. The y-axis is in log scale. Data source: NOAA [7]_

In order to create the visualizations in this example, we need to learn how to load datasets into our programs and manipulate the data for plotting. We will cover these topics in the next couple of chapters.

---

## Exercises

*Ex.3.1. For the climate stripes visualization (refer to Figure 3.2 top, and Chapter 1.3, Figure 1.4), the colormap is of type diverging (RdBu, reversed). Diverging means that there is a critical point such that on the left it is one color and on the right it is another color. In Figure 3.2 top, the critical point is the midpoint of the range of the data values for the temperature anomaly. Modify the code in Chapter 1.3 so that the midpoint corresponds to the temperature anomaly value of 0, and with the data value scale [-size, size] mapped to colormap scale [0.0, 1.0], where size is the smallest value such that [-size, size] contains all the data values. Compare the two visualizations with different critical points: Which is more informative of the data? Which is more aesthetically pleasing?

Ex.3.2. For applying transparency in the scatter plot Figure 3.3, modify the code and try using different alpha values, and different grayscale values for the `color` parameter. Also, try using a non-grayscale color value for the `color` parameter.

Ex.3.3. For visualizing groups in the scatter plot Figure 3.4, modify the code and try using different marker shapes; see Matplotlib's marker types in [2]. Also, try different color schemes for categorical data; see Matplotlib's "qualitative" colormaps in [3].
