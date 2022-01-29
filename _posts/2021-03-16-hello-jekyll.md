---
layout: post
title: Blog Post 1
---

# Interactive Data Graphics

## Introduction
Hiya! Welcome to another blog post! In this post we will be creating several interactive data graphics to analyze certain global warming data.

    Just like in blog post 0, let's start off by importing the necessary Python 
    libraries using the appropriate. sqlite3 is a module used to create, 
    manipulate, and query databases. LinearRegression is to fit a linear model. 
    plotly is a graphing library that makes interactive, publication-quality 
    graphs.


```python
import pandas as pd
from matplotlib import pyplot as plt
import numpy as np
import sqlite3

from sklearn.linear_model import LinearRegression
from plotly import express as px
```

 ## `Creating and Populating a Database`
 
    To use the sqlite3 module, we need to start by creating a Connection 
    object that represents the database. Here the data will be stored in 
    the climate.db file.


```python
conn = sqlite3.connect("climate.db")
```

    Before creating a database, we need to read three files into 
    separate dataframes. The three data sets separately contain a 
    record of surface temperatures of atmospheric measurement stations 
    across the globe; including country names and their internationally 
    standardized abbreviation.


```python
df = pd.read_csv("temps.csv")
stations = pd.read_csv("station-metadata.csv")
countries = pd.read_csv("countries.csv")
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>Year</th>
      <th>VALUE1</th>
      <th>VALUE2</th>
      <th>VALUE3</th>
      <th>VALUE4</th>
      <th>VALUE5</th>
      <th>VALUE6</th>
      <th>VALUE7</th>
      <th>VALUE8</th>
      <th>VALUE9</th>
      <th>VALUE10</th>
      <th>VALUE11</th>
      <th>VALUE12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACW00011604</td>
      <td>1961</td>
      <td>-89.0</td>
      <td>236.0</td>
      <td>472.0</td>
      <td>773.0</td>
      <td>1128.0</td>
      <td>1599.0</td>
      <td>1570.0</td>
      <td>1481.0</td>
      <td>1413.0</td>
      <td>1174.0</td>
      <td>510.0</td>
      <td>-39.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ACW00011604</td>
      <td>1962</td>
      <td>113.0</td>
      <td>85.0</td>
      <td>-154.0</td>
      <td>635.0</td>
      <td>908.0</td>
      <td>1381.0</td>
      <td>1510.0</td>
      <td>1393.0</td>
      <td>1163.0</td>
      <td>994.0</td>
      <td>323.0</td>
      <td>-126.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ACW00011604</td>
      <td>1963</td>
      <td>-713.0</td>
      <td>-553.0</td>
      <td>-99.0</td>
      <td>541.0</td>
      <td>1224.0</td>
      <td>1627.0</td>
      <td>1620.0</td>
      <td>1596.0</td>
      <td>1332.0</td>
      <td>940.0</td>
      <td>566.0</td>
      <td>-108.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ACW00011604</td>
      <td>1964</td>
      <td>62.0</td>
      <td>-85.0</td>
      <td>55.0</td>
      <td>738.0</td>
      <td>1219.0</td>
      <td>1442.0</td>
      <td>1506.0</td>
      <td>1557.0</td>
      <td>1221.0</td>
      <td>788.0</td>
      <td>546.0</td>
      <td>112.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ACW00011604</td>
      <td>1965</td>
      <td>44.0</td>
      <td>-105.0</td>
      <td>38.0</td>
      <td>590.0</td>
      <td>987.0</td>
      <td>1500.0</td>
      <td>1487.0</td>
      <td>1477.0</td>
      <td>1377.0</td>
      <td>974.0</td>
      <td>31.0</td>
      <td>-178.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1359932</th>
      <td>ZIXLT622116</td>
      <td>1966</td>
      <td>2180.0</td>
      <td>2040.0</td>
      <td>1840.0</td>
      <td>1690.0</td>
      <td>1430.0</td>
      <td>1280.0</td>
      <td>1210.0</td>
      <td>1460.0</td>
      <td>1770.0</td>
      <td>1980.0</td>
      <td>2090.0</td>
      <td>2110.0</td>
    </tr>
    <tr>
      <th>1359933</th>
      <td>ZIXLT622116</td>
      <td>1967</td>
      <td>2110.0</td>
      <td>1990.0</td>
      <td>1890.0</td>
      <td>1920.0</td>
      <td>1510.0</td>
      <td>1350.0</td>
      <td>1100.0</td>
      <td>1380.0</td>
      <td>1660.0</td>
      <td>2080.0</td>
      <td>1990.0</td>
      <td>1910.0</td>
    </tr>
    <tr>
      <th>1359934</th>
      <td>ZIXLT622116</td>
      <td>1968</td>
      <td>2180.0</td>
      <td>2000.0</td>
      <td>1930.0</td>
      <td>1820.0</td>
      <td>1560.0</td>
      <td>1080.0</td>
      <td>1370.0</td>
      <td>1630.0</td>
      <td>1760.0</td>
      <td>2180.0</td>
      <td>1840.0</td>
      <td>2070.0</td>
    </tr>
    <tr>
      <th>1359935</th>
      <td>ZIXLT622116</td>
      <td>1969</td>
      <td>2090.0</td>
      <td>2150.0</td>
      <td>1950.0</td>
      <td>1830.0</td>
      <td>1410.0</td>
      <td>1310.0</td>
      <td>1160.0</td>
      <td>1460.0</td>
      <td>1780.0</td>
      <td>2100.0</td>
      <td>2040.0</td>
      <td>1910.0</td>
    </tr>
    <tr>
      <th>1359936</th>
      <td>ZIXLT622116</td>
      <td>1970</td>
      <td>2070.0</td>
      <td>1990.0</td>
      <td>1930.0</td>
      <td>1720.0</td>
      <td>1560.0</td>
      <td>1330.0</td>
      <td>1330.0</td>
      <td>1540.0</td>
      <td>2040.0</td>
      <td>2030.0</td>
      <td>2130.0</td>
      <td>2150.0</td>
    </tr>
  </tbody>
</table>
<p>1359937 rows × 14 columns</p>
</div>



    Let's create a function to clean up the created dataframes before 
    incorporating the data into a database.


```python
def prepare_df(df):
    """
    Returns cleaned dataframe
    
    Parmeter:
        df: dataframe
    """
    
    
    df["FIPS 10-4"] = df["ID"].str[0:2] # adding a column to include country abbreviations
    #  converting the columns that will no be stacked into a multi-index for the data frame
    df = df.set_index(keys=["ID", "Year", "FIPS 10-4"])
    df = df.stack()                 # stacking" all of the data values on top of each other
    df = df.reset_index()           # reseting the index of the dataframe
    df = df.rename(columns = {"level_3"  : "Month" , 0 : "Temp"}) #renaming certain columns
    df["Month"] = df["Month"].str[5:].astype(int) # extracting the month
    df["Temp"]  = df["Temp"] / 100                # getting temperature in degrees C
    return(df)
```


```python
df = prepare_df(df)
countries = countries.rename(columns = {"Name" : "Country"}) # renaming column
```

    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/core/generic.py:2872: UserWarning: The spaces in these column names will not be changed. In pandas versions < 0.14, spaces were converted to underscores.
      sql.to_sql(



```python
# writing to the specified table in the database
df.to_sql("temperatures", conn, if_exists = "append", index = False)
stations.to_sql("stations", conn, if_exists = "replace", index = False)
countries.to_sql("countries", conn, if_exists = "replace", index = False)
```

## `Queries`
    Now we have a database containing three tables named 'temperatures', 
    'stations', and 'countries'. Let's take a look at the database to 
    confirm.


```python
cursor = conn.cursor()
cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
print(cursor.fetchall())
```

    [('temperatures',), ('stations',), ('countries',)]



```python
def query_climate_database(country, year_begin, year_end, month):
    """
     Returns a Pandas dataframe of temperature readings for the specified country, 
     in the specified date range, in the specified month of the year
     
     Parameters:
         Each parameter is querying columns from the tables in the database
             country 
             year_begin
             year_end
             month
    """
    
    cmd = \
    """
    SELECT S.name, S.latitude, S.longitude, C.country, T.year, T.month, T.temp
    FROM temperatures T
    INNER JOIN stations S ON T.id = S.id 
    INNER JOIN countries C ON T.[FIPS 10-4] = C.[FIPS 10-4]
    WHERE C.country = '%s' AND (T.year >= '%s' OR T.year <= '%s') AND T.month = '%s'
    """
    # SELECT: controls which column(s) will be returned
    # FROM: which table to return columns from
    # INNER JOIN: incorporating relational information into our query
    # WHERE: only rows in which this criterion is satisfied will be returned
    
    # obtain the specified columns into a dataframe
    df = pd.read_sql_query(cmd %(country, year_begin, year_end, month), conn)
    
    return df
```


```python
#test function
query_climate_database(country = "India", 
                       year_begin = 1980, 
                       year_end = 2020,
                       month = 1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
      <th>Country</th>
      <th>Year</th>
      <th>Month</th>
      <th>Temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.5830</td>
      <td>77.6330</td>
      <td>India</td>
      <td>2011</td>
      <td>1</td>
      <td>23.48</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.5830</td>
      <td>77.6330</td>
      <td>India</td>
      <td>2013</td>
      <td>1</td>
      <td>25.20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.5830</td>
      <td>77.6330</td>
      <td>India</td>
      <td>2014</td>
      <td>1</td>
      <td>24.96</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.5830</td>
      <td>77.6330</td>
      <td>India</td>
      <td>2015</td>
      <td>1</td>
      <td>24.90</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.5830</td>
      <td>77.6330</td>
      <td>India</td>
      <td>2016</td>
      <td>1</td>
      <td>24.50</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>13192</th>
      <td>DIU</td>
      <td>20.7167</td>
      <td>70.9167</td>
      <td>India</td>
      <td>1955</td>
      <td>1</td>
      <td>22.75</td>
    </tr>
    <tr>
      <th>13193</th>
      <td>DIU</td>
      <td>20.7167</td>
      <td>70.9167</td>
      <td>India</td>
      <td>1956</td>
      <td>1</td>
      <td>22.50</td>
    </tr>
    <tr>
      <th>13194</th>
      <td>DIU</td>
      <td>20.7167</td>
      <td>70.9167</td>
      <td>India</td>
      <td>1957</td>
      <td>1</td>
      <td>22.20</td>
    </tr>
    <tr>
      <th>13195</th>
      <td>DIU</td>
      <td>20.7167</td>
      <td>70.9167</td>
      <td>India</td>
      <td>1958</td>
      <td>1</td>
      <td>22.85</td>
    </tr>
    <tr>
      <th>13196</th>
      <td>DIU</td>
      <td>20.7167</td>
      <td>70.9167</td>
      <td>India</td>
      <td>1960</td>
      <td>1</td>
      <td>21.70</td>
    </tr>
  </tbody>
</table>
<p>13197 rows × 7 columns</p>
</div>



## `Geographic Data Visualization`


```python
df = query_climate_database(country = "India", 
                           year_begin = 1980, 
                           year_end = 2020,
                           month = 1)

def coef(data_group):
    x = data_group[["Year"]]
    y = data_group["Temp"]
    LR = LinearRegression()
    LR.fit(x, y)
    return LR.coef_[0]

coefs = df.groupby(["NAME", "LATITUDE", "LONGITUDE", "Country", "Month"]).apply(coef)

coefs = coefs.reset_index()
coefs

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
      <th>Country</th>
      <th>Month</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AGARTALA</td>
      <td>23.8830</td>
      <td>91.2500</td>
      <td>India</td>
      <td>1</td>
      <td>0.009995</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AGRA</td>
      <td>27.1667</td>
      <td>78.0333</td>
      <td>India</td>
      <td>1</td>
      <td>-0.001788</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AHMADABAD</td>
      <td>23.0670</td>
      <td>72.6330</td>
      <td>India</td>
      <td>1</td>
      <td>-0.016162</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AJMERE</td>
      <td>26.4700</td>
      <td>74.6200</td>
      <td>India</td>
      <td>1</td>
      <td>-0.005517</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AKOLA</td>
      <td>20.7000</td>
      <td>77.0330</td>
      <td>India</td>
      <td>1</td>
      <td>0.001553</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>113</th>
      <td>UDAIPUR_DABOK</td>
      <td>24.6170</td>
      <td>73.8830</td>
      <td>India</td>
      <td>1</td>
      <td>0.156358</td>
    </tr>
    <tr>
      <th>114</th>
      <td>VARANASI_BABATPUR</td>
      <td>25.4500</td>
      <td>82.8670</td>
      <td>India</td>
      <td>1</td>
      <td>0.023353</td>
    </tr>
    <tr>
      <th>115</th>
      <td>VERAVAL</td>
      <td>20.9000</td>
      <td>70.3670</td>
      <td>India</td>
      <td>1</td>
      <td>0.002563</td>
    </tr>
    <tr>
      <th>116</th>
      <td>VISHAKHAPATNAM</td>
      <td>17.7170</td>
      <td>83.2330</td>
      <td>India</td>
      <td>1</td>
      <td>-0.004859</td>
    </tr>
    <tr>
      <th>117</th>
      <td>VIZAGAPATAM</td>
      <td>17.7000</td>
      <td>83.3700</td>
      <td>India</td>
      <td>1</td>
      <td>-0.011871</td>
    </tr>
  </tbody>
</table>
<p>118 rows × 6 columns</p>
</div>




```python
def temperature_coefficient_plot(country, year_begin, year_end, month, min_obs, **kwargs):

    df = query_climate_database(country, year_begin, year_end, month)
    
    def coef(data_group):
        x = data_group[["Year"]]
        y = data_group["Temp"]
        LR = LinearRegression()
        LR.fit(x, y)
        return LR.coef_[0]
    
    coefs = df.groupby(["NAME", "LATITUDE", "LONGITUDE", "Country", "Month"]).apply(coef)
    
    coefs = coefs.reset_index()
    
    fig = px.scatter_mapbox(coefs, 
                        lat = "LATITUDE",
                        lon = "LONGITUDE",
                        hover_name = "NAME",
                        color = "NAME",
                        height = 300,
                        **kwargs)

    fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
```


```python
# test function
# assumes you have imported necessary packages
color_map = px.colors.diverging.RdGy_r # choose a colormap

fig = temperature_coefficient_plot("India", 1980, 2020, 1, 
                                   min_obs = 10,
                                   zoom = 2,
                                   mapbox_style = "carto-positron",
                                   color_continuous_scale = color_map)

fig.show()
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    /var/folders/q6/10s9dnt11c37jlcj336y9r9h0000gn/T/ipykernel_65299/1746835992.py in <module>
          9                                    color_continuous_scale = color_map)
         10 
    ---> 11 fig.show()
    

    AttributeError: 'NoneType' object has no attribute 'show'



```python
from plotly.io import write_html
write_html(fig, "geo_scatter.html")
```

    It's good practice to close your database connection once you're done using it.


```python
conn.close()   #close database connection
```
