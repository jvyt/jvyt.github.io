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


```python
df_iter = pd.read_csv("temps.csv", chunksize = 100000)
stations = pd.read_csv("station-metadata.csv")
countries = pd.read_csv("countries.csv")

df = df_iter.__next__()
```


```python
def prepare_df(df):
    df["FIPS 10-4"] = df["ID"].str[0:2]
    df = df.set_index(keys=["ID", "Year", "FIPS 10-4"])
    df = df.stack()
    df = df.reset_index()
    df = df.rename(columns = {"level_3"  : "Month" , 0 : "Temp"})
    df["Month"] = df["Month"].str[5:].astype(int)
    df["Temp"]  = df["Temp"] / 100
    return(df)
```


```python
df = prepare_df(df)
df.to_sql("temperatures", conn, if_exists = "replace", index = False)

df_iter = pd.read_csv("temps.csv", chunksize = 100000)
for df in df_iter:
    df = prepare_df(df)
    df.to_sql("temperatures", conn, if_exists = "replace", index = False)
```

    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/core/generic.py:2872: UserWarning: The spaces in these column names will not be changed. In pandas versions < 0.14, spaces were converted to underscores.
      sql.to_sql(



```python
stations.to_sql("stations", conn, if_exists = "replace", index = False)
countries.to_sql("countries", conn, if_exists = "replace", index = False)
```

## `Queries`


```python
cursor = conn.cursor()
cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
print(cursor.fetchall())
```

    [('temperatures',), ('stations',), ('countries',)]



```python
def query_climate_database(country, year_begin, year_end, month):
    cmd = \
    """
    SELECT S.name, S.latitude, S.longitude, C.name, T.year, T.month, T.temp
    FROM temperatures T
    INNER JOIN stations S ON T.id = S.id 
    INNER JOIN countries C ON T.[FIPS 10-4] = C.[FIPS 10-4]
    WHERE C.name = country AND (T.year >= year_begin OR T.year <= year_end) AND T.month = month
    """
    
    df = pd.read_sql_query(cmd, conn)
    
    return df
```


```python
#test function
query_climate_database(country = "India", 
                       year_begin = 1980, 
                       year_end = 2020,
                       month = 1)
```


    ---------------------------------------------------------------------------

    OperationalError                          Traceback (most recent call last)

    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/io/sql.py in execute(self, *args, **kwargs)
       2055         try:
    -> 2056             cur.execute(*args, **kwargs)
       2057             return cur


    OperationalError: no such column: country

    
    The above exception was the direct cause of the following exception:


    DatabaseError                             Traceback (most recent call last)

    /var/folders/q6/10s9dnt11c37jlcj336y9r9h0000gn/T/ipykernel_51741/3060603923.py in <module>
          1 #test function
    ----> 2 query_climate_database(country = "India", 
          3                        year_begin = 1980,
          4                        year_end = 2020,
          5                        month = 1)


    /var/folders/q6/10s9dnt11c37jlcj336y9r9h0000gn/T/ipykernel_51741/3688955949.py in query_climate_database(country, year_begin, year_end, month)
          9     """
         10 
    ---> 11     df = pd.read_sql_query(cmd, conn)
         12 
         13     return df


    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/io/sql.py in read_sql_query(sql, con, index_col, coerce_float, params, parse_dates, chunksize, dtype)
        434     """
        435     pandas_sql = pandasSQL_builder(con)
    --> 436     return pandas_sql.read_query(
        437         sql,
        438         index_col=index_col,


    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/io/sql.py in read_query(self, sql, index_col, coerce_float, params, parse_dates, chunksize, dtype)
       2114 
       2115         args = _convert_params(sql, params)
    -> 2116         cursor = self.execute(*args)
       2117         columns = [col_desc[0] for col_desc in cursor.description]
       2118 


    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/io/sql.py in execute(self, *args, **kwargs)
       2066 
       2067             ex = DatabaseError(f"Execution failed on sql '{args[0]}': {exc}")
    -> 2068             raise ex from exc
       2069 
       2070     @staticmethod


    DatabaseError: Execution failed on sql '
        SELECT S.name, S.latitude, S.longitude, C.name, T.year, T.month, T.temp
        FROM temperatures T
        INNER JOIN stations S ON T.id = S.id 
        INNER JOIN countries C ON T.[FIPS 10-4] = C.[FIPS 10-4]
        WHERE C.name = country AND (T.year >= year_begin OR T.year <= year_end) AND T.month = month
        ': no such column: country


## `Geographic Data Visualization`


```python
def temperature_coefficient_plot(country, year_begin, year_end, month, min_obs, **kwargs):

    def coef(data_group):
        x = data_group[["Year"]] # 2 brackets because X should be a df
        y = data_group["Temp"]   # 1 bracket because y should be a series
        LR = LinearRegression()
        LR.fit(x, y)
        return LR.coef_[0]
    
    coefs = df.groupby(["NAME", "Month"]).apply(coef)

    coefs = coefs.reset_index()
    
    coords = pd.DataFrame({
    "lon" : [-118.44300984639733], 
    "lat" : [34.0696449790177]
    })
    
    fig = px.scatter_mapbox(coords, 
                        lat = "lat",
                        lon = "lon",
                        zoom = 15,
                        height = 300,
                        mapbox_style="open-street-map")

    fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
```


```python
# test function
# assumes you have imported necessary packages
color_map = px.colors.diverging.RdGy_r # choose a colormap

fig = temperature_coefficient_plot("India", 1980, 2020, 1, 
                                   min_obs = 10,
                                   zoom = 2,
                                   mapbox_style="carto-positron",
                                   color_continuous_scale=color_map)

fig.show()
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    /var/folders/q6/10s9dnt11c37jlcj336y9r9h0000gn/T/ipykernel_51741/482935465.py in <module>
          3 color_map = px.colors.diverging.RdGy_r # choose a colormap
          4 
    ----> 5 fig = temperature_coefficient_plot("India", 1980, 2020, 1, 
          6                                    min_obs = 10,
          7                                    zoom = 2,


    /var/folders/q6/10s9dnt11c37jlcj336y9r9h0000gn/T/ipykernel_51741/3270307080.py in temperature_coefficient_plot(country, year_begin, year_end, month, min_obs, **kwargs)
         10         return LR.coef_[0]
         11 
    ---> 12     coefs = df.groupby(["NAME", "Month"]).apply(coef)
         13 
         14     coefs = coefs.reset_index()


    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/core/frame.py in groupby(self, by, axis, level, as_index, sort, group_keys, squeeze, observed, dropna)
       7629         # error: Argument "squeeze" to "DataFrameGroupBy" has incompatible type
       7630         # "Union[bool, NoDefault]"; expected "bool"
    -> 7631         return DataFrameGroupBy(
       7632             obj=self,
       7633             keys=by,


    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/core/groupby/groupby.py in __init__(self, obj, keys, axis, level, grouper, exclusions, selection, as_index, sort, group_keys, squeeze, observed, mutated, dropna)
        887             from pandas.core.groupby.grouper import get_grouper
        888 
    --> 889             grouper, exclusions, obj = get_grouper(
        890                 obj,
        891                 keys,


    /opt/anaconda3/envs/PIC16B/lib/python3.8/site-packages/pandas/core/groupby/grouper.py in get_grouper(obj, key, axis, level, sort, observed, mutated, validate, dropna)
        860                 in_axis, level, gpr = False, gpr, None
        861             else:
    --> 862                 raise KeyError(gpr)
        863         elif isinstance(gpr, Grouper) and gpr.key is not None:
        864             # Add key to exclusions


    KeyError: 'NAME'



```python
conn.close()   #close database connection
```
