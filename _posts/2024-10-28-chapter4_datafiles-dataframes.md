---
layout: post
title:  "Chapter 4: Data Files and Data Frames"
date:   2024-10-28
categories: book
TOC_order: 4
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

In previous chapters, we included the small amounts of data that were used in the visualization examples. Now we will go into how to read the CSV data files for those examples. Once the data has been loaded, they can be programmatically manipulated using _data frame_ objects to extract the desired parts of the data. Subsets of a dataset can be visualized using the _small multiple_ design pattern of plots in a grid layout. We illustrate this with the NASA GISTEMP dataset.

To run the code in this chapter, the Matplotlib and Pandas libraries (which we had introduced in Chapter 2) need to be installed in your Python setup (see Appendix A).

## 4.1 Data Files

There are various formats for data files. One of the most widely adopted is the CSV format, which we will cover in this chapter. Other common formats are JSON, XML, Excel, etc.

_CSV (comma-separated values)_ is a file format the uses commas to separate the values on each line of text. In particular, tabular data is often put into CSV files. The original data may be stored in this format, or it may come from exporting the data from a spreadsheet or a database.

In addition to its simplicity, CSV files have the benefit of human readability. Because the file is just plain text, it can be opened and inspected and edited using any text or code editor like Notepad or VS Code.

For coding, there are file functions dedicated to handle CSV files. In Python, this is supported by the `csv` module which is included in the standard library. Moreover, special objects like data frames that are designed for data manipulation have functions to handle CSV files. The `DataFrame` object in the Pandas library for Python has support for CSV files.

To read a CSV file and to save out data to a CSV file, Python's `csv` module has the functions `csv.reader()` and `csv.writer()`, while Pandas `DataFrame` has the functions `read_csv()` and `to_csv()`.

## 4.2 Data Frames

A _data frame_ organizes data into rows and columns, similar to a spreadsheet or a database table. Typically, there are names for the columns, and the rows may be indexed by its row position or by its name (if available). Different implementations of data frames have been developed, notably in the R programming language for statistical computing and in the Python-based Pandas library for data manipulation and analysis.

For the purposes and examples in this book, data frames are not really necessary as it is straightforward to write the code in Python to manipulate the lists and dictionaries read in from the CSV data files. However, there are good reasons to use data frames. First, it is a fundamental data structure for data manipulation, analysis, and modeling, so it is worthwhile to learn it. Another reason is leveraging its design patterns, which enables more compact and readable code. Moreover, the data frame implementations are often optimized to do calculations efficiently, and to facilitate handling of large data files.

There are numerous functions for working with data frames and we will only touch on a small number of them in this book.

## 4.3 Data Types

An important detail is to ensure that the data values are in the correct data type for analysis and visualization. Having data values with the wrong type can lead to errors or cause bugs in the code.

Pandas supports the basic data types such as integer, float, string, datetime. (At a lower level, these types may be represented by Python built-in types or more efficient Numpy types.) The Pandas DataFrame `dtypes` property facilitates inspection of the data types of its columns. A column can be cast to a specified type using the `astype()` function.

When reading a CSV file into a DataFrame using the Pandas `read_csv()` function, it automatically attempts to infer the data types for each column, but sometimes the assigned data type is incorrect or not exactly what is desired. To ensure the correct data types, it is possible to explicitly specify the data type of a column or the conversion method using the various optional parameters in the `read_csv()` function.

## 4.4 Example: NASA GISTEMP dataset

The climate data for temperature anomaly is available as a CSV file that can be downloaded from the NASA GISTEMP website [1]. [Or from this book's supplementary material.] The file GLB.Ts+dSST.csv contains global land-ocean mean temperature anomaly data for the years and months from 1880 to the present year.

You can look at the contents of this file by opening it with any text or code editor such as Notepad or VS Code. As you can see, the first line is a description, the second line is a header line with the column names, and the rest of the lines are rows of data values.

To construct the same `temp_anoms` variable used in the code in Chapters 1 and 2, the data that we want is in the column "J-D" which stands for the January to December calendar year annual averages. (The 'D-N' column is for the seasonal year from December to November.) And we want them for the years beginning at 1880 and up to 2020 (or to the most recent year if you prefer).

The code for doing this is:

```python
import pandas as pd

df = pd.read_csv(GLB.Ts+dSST.csv, skiprows=1)

# select years 1880-2020
df_filtered = df[(df['Year'] >= 1880) & (df['Year'] <= 2020)]
print(df_filtered.head())
print(df_filtered.tail())

# select column and put into a Python list
ann_means = df_filtered['J-D']
temp_anoms = ann_means.to_list()
temp_anoms = [round(float(x)*100) for x in temp_anoms]
```

In the line of code to read the file into a DataFrame object `df`, we specify `skiprows=1` to not read the first line which is just a description of the file contents. The second row with column names are automatically parsed and used to index the columns of the DataFrame. Then we filter the years from 1880 to 2020. A convenient way to inspect the values of a DataFrame and check our program is to use the `head()` and `tail()` functions to get the first five rows and the last five rows, and print them out.

Next, we select the column 'J-D' containing the annual means. This data is retrieved as a one-dimensional Pandas _Series_ object. Finally, we convert this Series to a Python list, and multiply the values by 100 to obtain an integer scale (representing 0.01 degrees Celsius) that was used in Chapters 1 and 2.

We can also work directly with the Pandas DataFrame and Series objects instead of selecting and converting data to Python lists. We will do this in the next section.

To save a DataFrame out to a CSV file, use its `to_csv()` function. This function has a parameter `index` which is True by default, and in this case will insert an extra column containing the row numbers; setting `index=False` will prevent this behavior.

We give an example of saving out to a CSV file by constructing a new DataFrame with two columns for the years and for the temperature anomalies, and outputting that to a CSV file. First, we make another list for the years, and use this to initialize a new DataFrame specifying the column names and the lists of values for the columns. Then we write the DataFrame out to a CSV file with a descriptive filename:

```python
years = df_filtered['Year'].to_list()
df_new = pd.DataFrame({'Year': years, 'J-D': temp_anoms})
df_new.to_csv('temp_anoms_0.01C_1880-2020_J-D.csv', index=False)
```

## 4.5 Small Multiple Visualization

_Small multiple_ is a design pattern for visualizing different subsets of a dataset in a grid layout. It facilitates comparison by having the subplots close together, and when appropriate by using the same axes scales and limits. It was popularized in the books by Edward Tufte [2].

We can make a small multiple visualization of the global temperature anomalies at the finer granularity of months by accessing the columns of the data frame `df_filtered` from above. First, we extract the months from the column names, then similar to the 2x1 plot in the previous chapter, we create a 4x3 plot using the code below.

Before, for simplicity we converted things in Pandas data structures to Python lists. Here, we will work directly with Pandas data structures. In this way, we extract the names of the months, the subset of the data frame for the months data, and the years. We need to be careful and cast the temperature anomaly values to float type and the year values to int type (see Exercise 4.3). Note that the Pyplot `plot()` function can handle these data structures extracted from Pandas.

In each month's subplot, we plot the raw values (in dark gray) and the smoothed values (in black). The smoothing is performed directly on a Pandas Series object, as in the line of code inside the function `central_moving_average()` from the Chapter 2 section on Smoothing Data (whose input takes a Python list). To set the same limits on the y-axis, we use the function `set_ylim()`. To reduce clutter and save space, we only put the y-axis labels on the first subplot of each row.

The result is in Figure 4.1. From the subplots in the figure, we can clearly see some differences; for example, June has less variation over the years than December.

```python
months = df.columns[1:13]
df_months = df_filtered[months].astype(float)
df_year = df_filtered['Year'].astype(int)

fig, axes = plt.subplots(4, 3)
for i in range(4):
    for j in range(3):
        # raw data
        temp_anoms_month = df_months[months[i*3 + j]]
        # central moving average
        smoothed = temp_anoms_month.rolling(window=5, center=True, min_periods=1).mean()

        axes[i, j].plot(df_year, temp_anoms_month, color='darkgray')
        axes[i, j].plot(df_year, smoothed, color='black')

        axes[i, j].set_ylim(-1.0, 1.5)

        axes[i, j].set_title(months[i*3 + j])
        if j % 3 == 0:
            axes[i, j].set_ylabel('Celsius')

plt.tight_layout()  # adjust spacing
plt.show()
```

![Figure 4.1](/bgdv-book/assets/images/book/plot-4x3_temp_anoms_v01-b.png)  
_Figure 4.1. Small multiple of temperature anomalies for each month from 1880 to 2020._

---

## Exercises

Ex.4.1. For the NASA GISTEMP data (Example 4.4), instead of using the Pandas DataFrame, use Python's csv module with the function csv.reader() to load the file, and write some code to extract the data to construct the list _temp_anoms_.

Ex.4.2. As an alternative to small multiple design for Figure 4.1, use a single plot and plot 12 lines for the 12 months on that plot.

*Ex.4.3. In the code for Figure 4.1, remove the astype() function call in the line to get df_months. What issues arise and why? _Hint:_ It has to do with the incomplete data.
