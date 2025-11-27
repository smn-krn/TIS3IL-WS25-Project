# Weather Data Preprocessing & Cleaning — Michigan Daily Weather Data

## 1. Overview & Objectives

This notebook is responsible for downloading, inspecting, cleaning, and imputing Michigan daily weather data from the Meteostat API.

**Key objectives:**
- Load daily weather data for Michigan from 01.01.1980 – 31.12.2025
- Inspect dataset structure, column types, and missing values
- Visualizations of raw data to understand trends and anomalies
- Remove irrelevant or insufficient columns (tsun, wpgt, wdir)
- Impute missing values using appropriate methods:
    - tavg from tmin and tmax
    - climatology-based averages (pres, snow)
    - random Forest prediction (wspd)
- Save the cleaned dataset for subsequent model training

**Expected outputs:**
- Cleaned CSV file: processed_weather_data_michigan.csv
- Plots of raw and cleaned weather variables for sanity checks


## 2. Imports & Setup

~~~python
import sys
import subprocess

subprocess.check_call([sys.executable, "-m", "ensurepip", "--upgrade"])
~~~

- ensures pip is installed and upgraded to the latest version for installing Python packages
- sys.executable points to the current Python interpreter, so the correct pip is used

~~~python
!"{sys.executable}" -m pip install meteostat #install meteostat package
~~~

- installs meteostat python library - allowing us to access weather stations and their daily weather data

~~~python
from meteostat import Stations, Point, Daily
from datetime import datetime
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestRegressor
~~~

- Stations, Point, Daily are used to query weather stations and fetch daily data
- datetime for handling dates
- matplotlib.pyplot for plotting time series
- pandas for data manipulation and cleaning
- numpy for numerical operations
- RandomForestRegressor from scikit-learn for imputing missing data


## 3. Load Data from Website
- We selected Michigan as the city of focus.
- To gather sufficient data for making reliable predictions, we chose 8 weather stations.

### 3.1. Stations Data
~~~python
# Get nearby weather stations
stations = Stations()
stations = stations.nearby(44.182205, -84.506836)
station = stations.fetch(8)

# Print DataFrame
print(station)
~~~

- Stations() creates a query object for weather stations
- .nearby(lat, lon) filters stations near the Michigan coordinates (44.182205, -84.506836)
- .fetch(8) retrieves the 8 closest stations as a DataFrame
- print(station) displays metadata: station names, IDs, elevation, start/end dates of observations


### 3.2 Daily Weather Data
~~~python
# Set time period
start = datetime(1980, 1, 1)
end = datetime(2025, 11, 25)

# Create Point for Michigan
Michigan = Point(44.182205, -84.506836)

# Get daily data
data = Daily(Michigan, start, end)
data = data.fetch()
print(data.index.min(), data.index.max())
print(data)
~~~

- defines start and end dates for the dataset
    - start date: 01.01.1980
    - end date: 31.12.2025 (in the end)
- Point(lat, lon) specifies the location for weather data
- Daily(Point, start, end).fetch() downloads daily weather measurements for Michigan in the date range
- print(data.index.min(), data.index.max()) confirms the date range of the fetched data
- print(data) shows the raw dataset, including temperature, precipitation, wind, snow, pressure, and sunshine columns - acts as a sanity check


## 4. Inspect Data
### 4.1. Overview
- inspect station metadata and weather dataset
- station.describe() gives basic statistics of numeric fields
- station.info() shows data types and missing values
- data.describe() and data.info() provide similar summary for weather data

#### **Stations data**
- less important for our weather predictions

#### **Weather data**
- contains key information relevant to our predictions

Output for data.info() shows:
~~~txt
DatetimeIndex: 16768 entries, 1980-01-01 to 2025-11-27
Freq: D
Data columns (total 10 columns):
 #   Column  Non-Null Count  Dtype  
---  ------  --------------  -----  
 0   tavg    14413 non-null  Float64
 1   tmin    16767 non-null  Float64
 2   tmax    16767 non-null  Float64
 3   prcp    16767 non-null  Float64
 4   snow    15399 non-null  Float64
 5   wdir    0 non-null      Float64
 6   wspd    15611 non-null  Float64
 7   wpgt    5923 non-null   Float64
 8   pres    10977 non-null  Float64
 9   tsun    1285 non-null   Float64
~~~

The output shows that most columns are well-populated, though some contain missing data, particularly the wdir column, which is entirely empty. 


***Columns:***
| *Code* | *Meaning*             |
| -------- | ----------------------- |
| TAVG     | Average Temperature     |
| TMIN     | Minimum Temperature     |
| TMAX     | Maximum Temperature     |
| PRCP     | Total Precipitation     |
| WDIR     | Wind (From) Direction   |
| WSPD     | Average Wind Speed      |
| WPGT     | Wind Peak Gust          |
| PRES     | Sea-Level Air Pressure  |
| SNOW     | Snow Depth              |
| TSUN     | Total Sunshine Duration |


### 4.2. Plots of raw data
- To simplify and streamline all the plots in the script, a dedicated function was created.

~~~python
def Plot(data, category, Title):
    data = data.copy()
    data.plot(y=[category])
    plt.xlabel('Date')
    plt.ylabel(Title)
    plt.title(f"{Title} for Michigan (2000-2025)")
    plt.show()
~~~

- data.copy() ensures the original data is not modified
- data.plot(y=[category]) plots the selected column as a time series
- Labels and title added for readability
- plt.show() displays the plot


**Temperature** <br>
***Average*** <br>
<img src="image.png" alt="alt text" width="400">

- shows missing data for the years 1981 to 1983 and 1986 to 1988

***Maximum*** <br>
<img src="image-1.png" alt="alt text" width="400">

- shows no missing data

***Minimum*** <br>
<img src="image-2.png" alt="alt text" width="400">

- shows no missing data

**Precipitation** <br>
<img src="image-3.png" alt="alt text" width="400">

- shows no missing data

**Precipitation** <br>
<img src="image-4.png" alt="alt text" width="400">

- shows missing data between 1981 and 1984

**Wind Peak Gust** <br>
<img src="image-5.png" alt="alt text" width="400">

- shows missing data from 1996 to the present

**Wind from Direction** <br>
- no plot as there is no data available for this column

**Sea Level Air Pressure** <br>
<img src="image-6.png" alt="alt text" width="400">

- shows missing data from 1981 to 1996

**Snow Depth** <br>
<img src="image-7.png" alt="alt text" width="400">

- shows missing data from 1997 to 2000 and from 2005 to 2006 during winter

**Sunshine Duration** <br>
<img src="image-8.png" alt="alt text" width="400">

- shows consistently missing data except for the period from 2000 to 2005


## 5. Preprocessing
### 5.1. Removing Columns 
~~~python
cols_to_remove = ['tsun', 'wpgt', 'wdir']

data_remcol = data.drop(columns=[c for c in cols_to_remove if c in data.columns])

data_remcol
~~~

Drops columns with insufficient or missing data:
- tsun: too many missing values
- wpgt: historical wind gust data missing after 1976
- wdir: contains only null values


### 5.2. 
**Average Temperature**
~~~python
# fill missing tavg using vectorized operations
data_remcol['tavg'] = data_remcol['tavg'].fillna(
    (data_remcol['tmin'] + data_remcol['tmax']) / 2
)

# ensure numeric dtype
data_remcol['tavg'] = pd.to_numeric(data_remcol['tavg'], errors='coerce')

data_clear = data_remcol

# Plot as sanity check
Plot(data_clear, 'tavg', 'Average Temperature (°C) after Cleaning')
~~~

- computes missing tavg values as the mean of tmin and tmax
- ensures numeric dtype using pd.to_numeric
- Plot() visualizes the cleaned average temperature series and acts as a sanity check

<img src="image-9.png" alt="alt text" width="400">

- Now the plot displays a time series of average temperature, which appears reasonable and can be used for further analysis.

**Sea-Level Pressure**
~~~python
df = data_remcol.copy()

df['doy'] = df.index.dayofyear
df['year'] = df.index.year

# get years with sufficient data
valid_years = (
    df['pres']
    .notna()
    .groupby(df['year'])
    .sum()
)

usable_years = valid_years[valid_years > 300].index  # years with full coverage

print("Usable years for climatology:", usable_years.tolist())

# build climatology from usable years
clim_source = df[df['year'].isin(usable_years)]

pressure_clim = clim_source.groupby('doy')['pres'].mean()

# fill missing data using climatology
df['pres_filled'] = df.apply(
    lambda row: pressure_clim[row['doy']] if pd.isna(row['pres']) else row['pres'],
    axis=1
)

# smooth filled data with rolling mean
df['pres_filled'] = (
    df['pres_filled']
    .rolling(window=30, center=True, min_periods=1)
    .mean()
)

df['pres'] = df['pres_filled']
df = df.drop(columns=['pres_filled', 'doy', 'year'])

data_clear = df

# Plot as sanity check
Plot(data_clear, 'pres', 'Sea-Level Air Pressure (hPa)')
~~~

- imputed using climatology: average value per day-of-year from years with sufficient data
- smoothed with 30-day rolling mean to reduce daily noise
- Plot(): acts as sanity check

<img src="image-10.png" alt="alt text" width="400">

- Although the plot covers the entire timeframe without any missing data, the results are not meaningful.


**Wind Speed**

~~~python
df = data_clear.copy()

predictors = ['tavg', 'tmin', 'tmax', 'pres', 'prcp', 'snow']

# ensure numeric
df[predictors] = df[predictors].apply(pd.to_numeric, errors='coerce')

# impute predictors first
df[predictors] = df[predictors].fillna(method='ffill').fillna(method='bfill')
df[predictors] = df[predictors].fillna(df[predictors].mean())

# split data
train = df[df['wspd'].notna()]
test  = df[df['wspd'].isna()]

# train model
model = RandomForestRegressor(n_estimators=300, random_state=42)
model.fit(train[predictors], train['wspd'])

# predict
df.loc[test.index, 'wspd'] = model.predict(test[predictors])

# light smoothing
df['wspd'] = df['wspd'].rolling(3, center=True, min_periods=1).mean()

data_clear = df

# Plot as sanity check
Plot(data_clear, "wspd", "Wind Speed (km/h) — Reconstructed")
~~~

- trains a Random Forest to predict missing wind speeds
- train contains rows with available wind speed, test rows with missing values
- predictions replace missing values, then smoothed using a 3-day rolling average

<img src="image-11.png" alt="alt text" width="400">

- Now the plot displays a time series of wind speed, which appears reasonable and can be used for further analysis.


**Snow Depth**

~~~python
df = data_clear.copy()
df.index = pd.to_datetime(df.index)

df['doy'] = df.index.dayofyear

snow_climatology = df.loc[df['snow'].notna(), ['snow', 'doy']].groupby('doy').mean()['snow']

missing_ranges = [
    ('1997-01-01', '2000-12-31'),
    ('2004-01-01', '2006-12-31')
]

df['snow_filled'] = df['snow']

for start, end in missing_ranges:
    mask = (df.index >= start) & (df.index <= end)
    df.loc[mask, 'snow_filled'] = df.loc[mask, 'doy'].apply(lambda d: snow_climatology[d])

df['snow_filled'] = df['snow_filled'].rolling(7, center=True, min_periods=1).mean()

df['snow'] = df['snow_filled']
df = df.drop(columns=['snow_filled', 'doy'])

data_clear = df

# Plot as sanity check
Plot(data_clear, 'snow', 'Snow Depth (cm)')
~~~

- imputed using climatology of valid years, filling missing ranges
- smoothed with a 7-day rolling average for continuity

<img src="image-12.png" alt="alt text" width="400">

- Now the plot displays a time series of snow depth, which appears reasonable and can be used for further analysis.

***Dropping Sea-Level Pressure***
~~~python
data_clear.drop(columns=['pres'], inplace=True)
data_clear
~~~

- column pres dropped because missing values were too extensive for reliable estimation


## 6. Export Processed Data
~~~python
data_clear.to_csv(
    r'C:\Users\Celina Binder\Documents\github respository\TIS3IL-WS25-Project\data\processed\processed_weather_data_michigan.csv'
)
~~~

- saves the cleaned, imputed dataset to a CSV file for use in modeling and visualization

## Summary
The resulting dataset consists of the following columns:
- Snow Depth
- Wind Speed
- Total Precipitation
- Average Temperature
- Maximum Daily Temperature
- Minimum Daily Temperature
- time (datetime) 

This preprocessed dataset contains no more missing values.

~~~txt
DatetimeIndex: 16768 entries, 1980-01-01 to 2025-11-27
Freq: D
Data columns (total 6 columns):
 #   Column  Non-Null Count  Dtype  
---  ------  --------------  -----  
 0   tavg    16768 non-null  Float64
 1   tmin    16768 non-null  Float64
 2   tmax    16768 non-null  Float64
 3   prcp    16768 non-null  Float64
 4   snow    16768 non-null  float64
 5   wspd    16768 non-null  float64
~~~