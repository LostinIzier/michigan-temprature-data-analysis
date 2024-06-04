# michigan-temprature-data-analysis
This project involves analyzing temperature data from 2005 to 2015, highlighting record high and low temperatures in 2015 compared to the previous decade. It visualizes the data by plotting temperature trends and identifying days in 2015 when temperature records were broken.
The provided code performs a series of steps to analyze and visualize temperature data. Here's a detailed breakdown of each part:

import pandas as pd
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt
plt.figure()

df = pd.read_csv('assets/BinSize_d400.csv')
station_locations_by_hash = df[df['hash'] == 'fb441e62df2d58994928907a91895ec62c2c42e6cd075c2700843b89']
lons = station_locations_by_hash['LONGITUDE'].tolist()
lats = station_locations_by_hash['LATITUDE'].tolist()

//The data from BinSize_d400.csv is read into a DataFrame.
//The DataFrame is filtered to include only rows with a specific hash value.
//Longitude and latitude values are extracted into lists for later use.

df = pd.read_csv('assets/fb441e62df2d58994928907a91895ec62c2c42e6cd075c2700843b89.csv')
df["Data_Value"] = df["Data_Value"].apply(lambda x: x / 10)
df['Date'] = pd.to_datetime(df['Date'])
df['Year'] = df['Date'].dt.year
df['Month'] = df['Date'].dt.month
df['Day'] = df['Date'].dt.day
df = df[~((df['Month'] == 2) & (df['Day'] == 29))]

//The temperature data is read from a CSV file.
//Temperature values are converted to Celsius (assuming the original values were in tenths of degrees).
//The date column is converted to a datetime format, and year, month, and day columns are extracted.
//Leap day (February 29) data is removed for consistency in yearly comparisons.

df_min = df[df["Element"] == "TMIN"]
df_max = df[df["Element"] == "TMAX"]

df_min = df_min.sort_values(by="Date").groupby(['Date']).min()
df_max = df_max.sort_values(by="Date").groupby(['Date']).max()

//The DataFrame is split into two: one for minimum temperatures (TMIN) and one for maximum temperatures (TMAX).
//Both DataFrames are sorted by date and grouped by date, keeping the minimum values for TMIN and maximum values for TMAX.

df_2015_min = df_min[df_min["Year"] == 2015].groupby(["Month", "Day"]).aggregate({"Data_Value": np.min})
df_2015_max = df_max[df_max["Year"] == 2015].groupby(["Month", "Day"]).aggregate({"Data_Value": np.max})
df_0514_min = df_min[~(df_min["Year"] == 2015)].groupby(["Month", "Day"]).aggregate({"Data_Value": np.min})
df_0514_max = df_max[~(df_max["Year"] == 2015)].groupby(["Month", "Day"]).aggregate({"Data_Value": np.max})

//Data for 2015 is separated and grouped by month and day, aggregating the minimum and maximum values for each day.
//Data from 2005 to 2014 (excluding 2015) is similarly processed.

special_min = df_2015_min[df_2015_min["Data_Value"] < df_0514_min["Data_Value"]]
special_max = df_2015_max[df_2015_max["Data_Value"] > df_0514_max["Data_Value"]]
broken_max = np.where(df_2015_max['Data_Value'] > df_0514_max['Data_Value'])[0]
broken_min = np.where(df_2015_min['Data_Value'] < df_0514_min['Data_Value'])[0]

//Record-breaking temperatures in 2015 are identified by comparing with the 2005-2014 data.
//Indices of the days where records were broken are stored for plotting.

plt.plot(df_0514_min.values, label="Min temp 2005-2024", linewidth=1, c="blue")
plt.plot(df_0514_max.values, label="Max temp 2005-2024", linewidth=1, c="red")
plt.gca().fill_between(range(len(df_0514_min)), df_0514_min['Data_Value'], df_0514_max['Data_Value'], facecolor="yellow", alpha=.8)
plt.scatter(broken_max, df_2015_max.iloc[broken_max], s=10, c="orange", label="High temp broken")
plt.scatter(broken_min, df_2015_min.iloc[broken_min], s=10, c="green", label="Low temp broken")
plt.legend(loc='best', title='Temperature', fontsize=8)

plt.xticks(np.linspace(0, 30 + 30*11, num=12), (r'Jan', r'Feb', r'Mar', r'Apr', r'May', r'Jun', r'Jul', r'Aug', r'Sep', r'Oct', r'Nov', r'Dec'), alpha=0.8)
plt.yticks(alpha=0.8)
plt.xlim(0, 365)
plt.xlabel('Months', alpha=0.8)
plt.ylabel('Temperature ($^\circ$C)', alpha=0.8)
plt.title('Temperature Plot', alpha=0.8)

plt.gca().spines['top'].set_visible(False)
plt.gca().spines['right'].set_visible(False)
plt.gca().spines['bottom'].set_alpha(0.3)
plt.gca().spines['left'].set_alpha(0.3)

//The temperature data from 2005-2014 is plotted, showing the minimum and maximum temperatures.
//A shaded area between the min and max values is filled for better visualization.
//Record-breaking high and low temperatures in 2015 are highlighted with scatter points.
//Labels, legends, and aesthetic customizations are added to the plot for better readability and presentation.
//This code effectively visualizes temperature trends and highlights significant deviations in 2015 compared to the previous decade, providing insights into //potential changes in climate patterns.
