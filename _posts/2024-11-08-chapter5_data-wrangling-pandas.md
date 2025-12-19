---
layout: post
title:  "Chapter 5: Data Wrangling with Pandas"
date:   2024-11-08
categories: book
TOC_order: 5
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---

In the previous chapter, we showed how to use Pandas to read CSV files into data frames, select rows and columns to get data values, and filtering the data based on specific criteria. Often the structure of the data is not in the form that we want to input into the functions for making the visualization. Sometimes there are missing or bad data values. Another common task is to aggregate data. Performing these kinds of data manipulation tasks is called _data wrangling_.

In this chapter, we go into some useful techniques for data wrangling using Pandas. First, we briefly discuss missing or bad data, and fixing them with data imputation methods. As an example, we work with data from the Johns Hopkins CSSE Covid-19 dataset. Then we show how to aggregate data by grouping and computing their monthly means. Another basic task is restructuring data, and we show how to restructure the Covid-19 data for the US into a table similar to the structure of the NASA GISTEMP dataset enountered in earlier chapters.

## 5.1 Missing or Bad Data

It is not unusual for a dataset to have missing or incomplete data. With CSV files, which consist of human-readable text, a person can easily look at the content and spot some of them. There are various ways to represent missing values: by an empty string between commas, by "NaN" (not a number), by "NULL", by a specified string such as "***" in the NASA GISTEMP data, etc.

The Pandas `read_csv()` function can handle missing values such as empty strings between commas and replaces them with the value NaN. For non-standard representations such as "***", it will read in the value as a string, and the program will need to detect and handle them if necessary.

Using quick and simple data visualization techniques can also reveal them. For example, the Pyplot function `plot(x, y)` will show gaps in a plotted line at the places where y has missing values. More involved data checking can be performed by writing custom scripts to detect missing or incomplete values that may potentially occur within a certain dataset. Once the missing values have been identified, there are _data imputation_ methods that can be applied, such as using simple interpolation or more sophisticated statistcal estimations.

Bad data often depends on the characteristics of a particular dataset. For instance, a sequence of cumulative values has to be monotonically increasing, and a bad value will violate this criterion. Pandas Series has a property `is_monotonic_increasing` that checks this condition. Directly related to the cumulative values are the point values (which are just the differences between successive cumulative values), and the corresponding check is to test for negative values. This can be done in Pandas with the statement `any(series < 0)` that returns whether any element in the series is less than zero.

Custom scripts can be written to detect the indices of the bad values, and each bad value can be replaced by a special placeholder value to mark these places; with Pandas the value `None` can be used. Another script can followed this to apply a data imputation technique to fill in the placeholder values. An example will be given in the next section.

## 5.2 Example: CSSE Covid-19 dataset

Covid-19 data was collected by Johns Hopkins Center for Systems Science and Engineering (CSSE) [1] from January 2020 to March 2023, essentially during the period of global emergency [2].  The dataset is available as CSV files that can be downloaded from the CSSE website [1]. [Or from this books's supplementary material.] We will work with the files time_series_covid19_deaths_global.csv and time_series_covid19_confirmed_global.csv, from which we can extract the cumulative daily deaths and cases for the country US.

These CSV files can be viewed in a text editor to familiarize ourselves with the row and column labels and the contents. We can use Pandas to read the CSV file for deaths, extract the data for US and save it out to a new CSV file. See the code is below. After reading in the CSV file as a DataFrame, we extract the data for the country 'US'. Next we use the `drop()` function to remove the columns that are no longer relevant. Then we create a new DataFrame with columns for the dates and for the corresponding values, and save it out to a CSV file.

```python
df = pd.read_csv('time_series_covid19_deaths_global.csv')
df_country = df.loc[df['Country/Region'] == 'US']
df_country = df_country.drop(
    columns=['Province/State', 'Country/Region', 'Lat', 'Long'])
df_new = pd.DataFrame(
    {'date': df_country.columns.values, 'deaths': df_country.iloc[0].values})
df_new.to_csv('US.csv', index=False)  
```

This CSV file can be read in, along with converting the 'date' column values to datetime objects, by using the Pandas `read_csv()` function with the `parse_dates` option:

```python
df = pd.read_csv('US.csv', parse_dates=['date'], date_format='%m/%d/%y')
```

Looking at the data, we notice that the values are cumulative deaths. We check for monotonicity with the Pandas `is_monotonic_increasing` property mentioned in the previous section:

```python
print(df['deaths'].is_monotonic_increasing)
```

This prints out False.

We can find bad values at the time points where the monotonicity condition fails and replace them with the placeholder value `None`:

```python
series = df['deaths'].copy()
for i in range(len(series)):
    if i == 0:
        continue
    if series.iloc[i-1] > series.iloc[i]:
        print('bad i =', i, ', value =', series.iloc[i])
        series.iloc[i] = None
```

This code prints out:

```Text
bad i = 890, value = 1017377
bad i = 943, value = 1040905.0
bad i = 1132, value = 1119582.0
bad i = 1138, value = 1122134.0
```

Furthermore, we can plot the neighborhoods around a bad value (Exercise 4.1). A plot of this is shown in Figure 5.1-top.

We can now fix the bad values by performing interpolation on the data Series at the placeholders with value None, and save out the result to a CSV file for later use. The default interpolation method is linear, which preserves monotonicity.  The other methods available (such as quadratic, spline, etc.) may not preserve monotonicity; see Excercise 5.2. A plot of the interpolated values is shown in Figure 5.1-bottom.

```python
series = series.interpolate()
df['deaths'] = series
df.to_csv('US_replace-bad-vals_interpolate.csv', index=False)  
```

![Figure 5.1](/bgdv-book/assets/images/book/US_plot_2x1_bad_interpolated_2022-06-30_v01-b.png)  
_Figure 5.1. Neighborhood around the bad value at 2022-06-30. Top: original data. Bottom: interpolated after removing the bad value._

Next, we compute the point values from the cumulative values. The Pandas `diff()` function calculates the difference between elements, with the default using the elements in the previous row. The first row has no previous row, and first point value is set to NaN.

The following code adds a new column with the point values, and replaces its first value (NaN) with the first cumulative value, checks for any negative values, and saves the result to a CSV file for later use.

```python
df['pointvals'] = df['deaths'].diff()
df['pointvals'] = df['pointvals'].fillna(df['deaths'].iloc[0])
print('\nany pointvals < 0? : ', any(df['pointvals'] < 0))

df.to_csv('US_replace-bad-vals_interpolate_pointvals.csv', index=False)  
```

We can plot the cumulative and the point values using code similar to that in Section 2.1 and 2.2. We leave this as exercise 5.3. These plots are shown in Figures 5.2 and 5.3.

For the point values plot, simply plotting these values in black will result in an oversaturated image. This is due to the relatively large daily fluctuations. Instead, it is better to plot the raw data (with the bad values having been replaced) using a light gray color, and over that plot the smoothed data (7-day central moving average) in black color. The code snippet for that is:

```python
x = df['date']
y = df['pointvals']
plt.plot(x, y, color='lightgray')
plt.plot(x, central_moving_average(y, window=7), color='black')
```

![Figure 5.2](/bgdv-book/assets/images/book/US_replace-bad-vals_interpolate_plot.png)  
_Figure 5.2. Cumulative values._

![Figure 5.3](/bgdv-book/assets/images/book/US_replace-bad-vals_interpolate_pointvals_plot_smooth.png)  
_Figure 5.3. Point values in light gray, and the 7-day central moving average in black._

## 5.3 Aggregating Data

We have seen above how smoothing can help cut through the noise and enable us to see the trends better for the Covid-19 data in Figure 5.3 and for the climate data in Fig. 2.2. Another way to deal with the noise is by grouping the data into disjoint subsets and computing the average for each subset. This can be performed using data aggregation functions.

As an example, for the Covid-19 dataset, we can group the number of daily deaths by month, and calculate the average number of deaths per day for each month.

To facilitate this, we write functions to group a data frame by year and by month, using the Pandas `groupby()` function:

```python
def groupby_year(df):
    # extract the year from the date into a new column
    df['year'] = df['date'].dt.year

    # create a dictionary of DataFrames for each year
    dfs_by_year = {}
    for year, group in df.groupby('year'):
        dfs_by_year[year] = group.drop(columns=['year'])
```

The code for the `groupby_month(df)` function is exactly like `groupby_year(df)`, with `'year'` replaced by `'month'`.

Next, we create a Numpy 2D array with number-of-years rows and 13 columns (first column is for the 'year' values plus 12 columns for the months' values), and initialize all the values in the array to NaN:

```python
arr_2d = np.empty((len(dfs_by_year), 1 + len(months)))
arr_2d.fill(np.nan)
```

Then we fill in the values in the 2D array with the monthly mean. In order to do this, we group the data frame with the Covid-19 daily point values (`df` from the preivous section) by year, and then further group those subgroups by month. The mean for each monthly subgroup is calculated simply by calling its `mean()` function.

```python
dfs_by_year = groupby_year(df)

idx_year = 0
for year, group in dfs_by_year.items():
    arr_2d[idx_year, 0] = year

    dfs_by_month = groupby_month(group)

    for month, month_group in dfs_by_month.items():
        mean = month_group['pointvals'].mean()
        mean = int(round(mean, 0))
        arr_2d[idx_year, month] = mean

    idx_year += 1
```

After that, we create a new data frame with the 2D array values. For the column names, we can use the similar column names in the NASA GISTEMP dataset. Then we save this to a CSV file.

```python
months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 
          'Oct', 'Nov', 'Dec']
col_names = months
col_names.insert(0, 'Year')
df_monthly_mean = pd.DataFrame(arr_2d, columns=col_names)    

# convert the values to integer type (optional)
df_monthly_mean['Year'] = df_monthly_mean['Year'].astype(int)

# save to .csv file
df_monthly_mean.to_csv('US_pointvals_monthly-mean.csv', index=False)  
```

The resulting CSV file looks like:

```Text
Year,Jan,Feb,Mar,Apr,May,Jun,Jul,Aug,Sep,Oct,Nov,Dec
2020,0,0,173,2043,1330,652,856,930,772,793,1335,2581
2021,3097,2329,1178,794,584,348,284,912,1971,1548,1151,1485
2022,2046,2193,1059,431,355,356,398,521,442,351,336,397
2023,514,401,435,,,,,,,,,
```

We can plot the monthly mean values, using round dots to emphasize the discreteness of monthly data points (Exercise 5.4). The result is shown in Figure 5.4.

Note that this plot looks very similar to the plot of the 7-day central moving average in Figure 5.3. The main difference between smoothing with a central moving average and aggregating into months along with calculating their means is that the averages are calculated in the former case over a sliding window and in the latter case over disjoint groups (or "windows").

![Figure 5.4](/bgdv-book/assets/images/book/US_plot_pointvals_monthly-mean_deaths_v01-b.png)  
_Figure 5.4. Monthly mean values._

---

## Exercises

Ex.5.1. Write code to plot the neighborhood around the bad values (places of non-monotonicity) for the CSSE Covid-19 data for the US deaths and cases. Try a neighborhood size of 3 below and above the bad value, as those shown in Figure 5.1. _Hints:_ User `marker='o'` parameter in the `plot()` function to show the points discretely in the line graph, and use the function `tick_params()` to rotate the tick labels 45 degrees.

*Ex.5.2. For the example with Covid-19 data in section 4.2, try other interpolation methods such as quadratic or spline (cubic of order 3). Do these give results that are monotonic?

Ex.5.3. For the Covid-19 data, plot the cumulative and the point values as shown in Figures 5.2 and 5.3, using code similar to that in Section 2.1 and 2.2, along with the code snippet in Section 5.2 for plotting the 7-day central moving average on top of the point values.

Make another plot of the 31-day central moving average. How does this look compared to the monthly mean calculated by aggregating the data into monthly groups (Figure 5.4)?

Ex.5.4. For the Covid-19 data, plot the monthly average values to obtain a plot that looks like Figure 5.4. _Hint:_ User `marker='o'` parameter in the `plot()` function to show the points discretely in the line graph.

Ex.5.5. Extreme rainfall events were visualized in Chapter 3 (Figuure 3.5 and 3.6) for the city of Bengaluru (Bangalore). From the NOAA website [3], download rainfall data for a city of your choice and make visualization like Figuure 3.5 and 3.6.
