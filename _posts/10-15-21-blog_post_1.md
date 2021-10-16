```python
import numpy as np
import seaborn as sns
import pandas as pd
import urllib
from matplotlib import pyplot as plt
import sqlite3

```

## §1. Create a Database

First, we open a database and name it `climate`. This database will be filled with data about temperatures, stations, and countries. 

First, create a database with three tables: temperatures, stations, and countries. Information on how to access country names and relate them to temperature readings is in this lecture. Rather than merging, as we did in the linked lecture, you should keep these as three separate tables in your database.

Make sure to close the database connection after you are finished constructing it.


```python
conn = sqlite3.connect("climate.db") # create a database in current directory called climate.db
```


```python
temps = pd.read_csv("/Users/qianzach/Desktop/PIC_16B/temps.csv") #load our temperatures data
temps.head()
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
  </tbody>
</table>
</div>



Next, we want to clean our data. We can use Professor Chodrow's helper function `prepare_df()` for this:


```python
def prepare_df(df):
    """
    This function helps change the indices and move our monthly temperatures from column values to a singular one
    from 1-12. Next, we also have to change the units of the temperatures. 
    """    df = df.set_index(keys=["ID", "Year"])
    df = df.stack()
    df = df.reset_index()
    df = df.rename(columns = {"level_2"  : "Month" , 0 : "Temp"})
    df["Month"] = df["Month"].str[5:].astype(int)
    df["Temp"]  = df["Temp"] / 100
    return(df)
```

Let's take a look at our newly prepared data:


```python
df = prepare_df(temps)
df.head()
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
      <th>Month</th>
      <th>Temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACW00011604</td>
      <td>1961</td>
      <td>1</td>
      <td>-0.89</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ACW00011604</td>
      <td>1961</td>
      <td>2</td>
      <td>2.36</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ACW00011604</td>
      <td>1961</td>
      <td>3</td>
      <td>4.72</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ACW00011604</td>
      <td>1961</td>
      <td>4</td>
      <td>7.73</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ACW00011604</td>
      <td>1961</td>
      <td>5</td>
      <td>11.28</td>
    </tr>
  </tbody>
</table>
</div>



Since we're dealing with large data, we can read iteratively with `chunksize` parameter in `read_csv()`!


```python
df_iter = pd.read_csv("temps.csv", chunksize = 100000) #choose selected chunk size
for df in df_iter: #iterate through 
    df = prepare_df(df)
    df.to_sql("temperatures", conn, if_exists = "append", index = False)
```

####  Load in Stations  and Countries Metadata
Now we can load into our database some data, specifically station and country data! These are relatively smaller data sets so we don't need to worry about reading it in by chunks like we do for `temps`. 


```python
#stations
url1 = "https://raw.githubusercontent.com/PhilChodrow/PIC16B/master/datasets/noaa-ghcn/station-metadata.csv"
stations = pd.read_csv(url1)
stations.to_sql("stations", conn, if_exists = "replace", index = False)
```


```python
#countries
url2  = "https://raw.githubusercontent.com/mysociety/gaze/master/data/fips-10-4-to-iso-country-codes.csv"
countries = pd.read_csv(url2)
#because there are spaces, we will simply delete them
countries.columns = countries.columns.str.replace(" ", "") #delete the whitespaces in column names
countries.to_sql("countries", conn, if_exists =  "replace", index = False)
```


```python
cursor = conn.cursor()
cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
print(cursor.fetchall())
```

    [('temperatures',), ('stations',), ('countries',)]


Since we're done, we can now close connection with our server.


```python
conn.close()  #close the connection
```

## §2. Create Query Function

Now, I will write a function called `query_climate_database()` which accepts four arguments:

- `country`, a string giving the name of a country for which data should be returned.
- `year_begin` and `year_end`, two integers giving the earliest and latest years for which should be returned.
- `month`, an integer giving the month of the year for which should be returned.

**Our output should have the following as columns:**

- The station name.
- The latitude of the station.
- The longitude of the station.
- The name of the country in which the station is located.
- The year in which the reading was taken.
- The month in which the reading was taken.
- The average temperature at the specified station during the specified year and month. (Note: the temperatures in the raw data are already averages by month, so you don’t have to do any aggregation at this stage.)

First, let's relook at the data that our tables hold:


```python
stations.head()
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
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
      <th>STNELEV</th>
      <th>NAME</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACW00011604</td>
      <td>57.7667</td>
      <td>11.8667</td>
      <td>18.0</td>
      <td>SAVE</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AE000041196</td>
      <td>25.3330</td>
      <td>55.5170</td>
      <td>34.0</td>
      <td>SHARJAH_INTER_AIRP</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AEM00041184</td>
      <td>25.6170</td>
      <td>55.9330</td>
      <td>31.0</td>
      <td>RAS_AL_KHAIMAH_INTE</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AEM00041194</td>
      <td>25.2550</td>
      <td>55.3640</td>
      <td>10.4</td>
      <td>DUBAI_INTL</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AEM00041216</td>
      <td>24.4300</td>
      <td>54.4700</td>
      <td>3.0</td>
      <td>ABU_DHABI_BATEEN_AIR</td>
    </tr>
  </tbody>
</table>
</div>




```python
countries.head()
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
      <th>FIPS10-4</th>
      <th>ISO3166</th>
      <th>Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AF</td>
      <td>AF</td>
      <td>Afghanistan</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AX</td>
      <td>-</td>
      <td>Akrotiri</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AL</td>
      <td>AL</td>
      <td>Albania</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AG</td>
      <td>DZ</td>
      <td>Algeria</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AQ</td>
      <td>AS</td>
      <td>American Samoa</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.head()
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
      <th>Month</th>
      <th>Temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>USW00014924</td>
      <td>2016</td>
      <td>1</td>
      <td>-13.69</td>
    </tr>
    <tr>
      <th>1</th>
      <td>USW00014924</td>
      <td>2016</td>
      <td>2</td>
      <td>-8.40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>USW00014924</td>
      <td>2016</td>
      <td>3</td>
      <td>-0.20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>USW00014924</td>
      <td>2016</td>
      <td>4</td>
      <td>3.21</td>
    </tr>
    <tr>
      <th>4</th>
      <td>USW00014924</td>
      <td>2016</td>
      <td>5</td>
      <td>13.85</td>
    </tr>
  </tbody>
</table>
</div>



Ok! That means we get the station name, latitude, and longitude from `stations`, country name from `countries`, and the year/month/avg temperature in `temps`! Let's make a quick SQL query that accomplishes this: 


```python
def query_climate_database(country, year_begin, year_end, month):
    """
    This function takes these four parameters is a Pandas dataframe of temperature readings for the 
    specified country, in the specified date range, in the specified month of the year. 
    We Select based on the criteria given and perform a LEFT JOIN to merge the IDs of our temps df and our countries df by the FIPS 10-4 ID
    
    Lastly, we want to perform a left join that merges based on a match between the stations df ID and temps df ID.
    by ID similar to class.
    
    Then, using our parameters, we provide the country name, year between and the months for this query.
    
    
    """
    
    cmd = \
    """
    SELECT S.NAME, S.LATITUDE, S.LONGITUDE, C.Name, T.Year, T.Month, T.Temp
    FROM temperatures T
    LEFT JOIN countries C on C.'FIPS10-4' = SUBSTRING(T.ID, 1, 2)
    LEFT JOIN stations S ON S.ID = T.ID
    
    WHERE C.Name = ?
        AND T.YEAR BETWEEN ? AND ?
        AND T.MONTH = ?

    """
    conn = sqlite3.connect("climate.db") #connect to climate database
    df = pd.read_sql_query(cmd, conn, params=(country, year_begin, year_end, month)) #make call w/ db using given params
    df = df.rename(columns={"NAME":"STATION_NAME","Name":"Country"}) #rename column name of "Name" to Country
    conn.close() #close connection
    return df #return dataframe
    
```

Using the parameters that Professor Chodrow gave us as an example, let's try out our query!


```python
query_climate_database(country = "India", 
                       year_begin = 1980, 
                       year_end = 2020,
                       month = 1).head()
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
      <th>STATION_NAME</th>
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
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1980</td>
      <td>1</td>
      <td>23.48</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1981</td>
      <td>1</td>
      <td>24.57</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1982</td>
      <td>1</td>
      <td>24.19</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1983</td>
      <td>1</td>
      <td>23.51</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1984</td>
      <td>1</td>
      <td>24.81</td>
    </tr>
  </tbody>
</table>
</div>



It works! Now we have completed our query from the climate database.

## §3. Geographic Scatter Function for Yearly Temperature Increases
Our next goal is to write a function called `temperature_coefficient_plot()`. This function should accept five explicit arguments, and an undetermined number of keyword arguments.

`country`, `year_begin`, `year_end`, and `month` should be as in the previous part.
`min_obs`, the minimum required number of years of data for any given station. Only data for stations with at least `min_obs` years worth of data in the specified month should be plotted; the others should be filtered out. `df.transform()` plus filtering is a good way to achieve this task.
`**kwargs`, additional keyword arguments passed to `px.scatter_mapbox()`. These can be used to control the colormap used, the mapbox style, etc.

Therefore, our goal is: **make an interactive geographic scatterplot, constructed using Plotly Express, with a point for each station, such that the color of the point reflects an estimate of the yearly change in temperature during the specified month and time period at that station.**

Per Professor Chodrow, one way to do this is is to compute the first coefficient of a linear regression model at that station, which we have seen in lecture!


```python
from sklearn.linear_model import LinearRegression #import libraries
def coef(data_group):
    """
    This helper function allows us compute a simple estimate of the year-over-year average change in temperature in each month at each station
    """
    x = data_group[["Year"]] # 2 brackets because X should be a df
    y = data_group["Temp"]   # 1 bracket because y should be a series
    LR = LinearRegression()
    LR.fit(x, y)
    return LR.coef_[0]

```

Now, we want to implement `temperature_coefficient_plot()`


```python
from plotly import express as px
import calendar
def temperature_coefficient_plot(country, year_begin, year_end, month, min_obs, **kwargs):
    """
    Plots estimated yearly change in temperature for a given area  based on a general area.
    We must satisfy the following criteria:
    - The station name is shown when you hover over the corresponding point on the map.
    - The estimates shown in the hover are rounded to a sober number of significant figures.
    - The colorbar and overall plot have professional titles.
    - The colorbar is centered at 0, so that the “middle” of the colorbar (white, in this case) corresponds to a coefficient of 0.
    """
    #data manipulation/computation
    df = query_climate_database(country, year_begin, year_end, month) #make query
    coefs = df.groupby(["STATION_NAME", "Month"]).apply(coef) #compute coefficients

    df["obs"] = df.groupby(["STATION_NAME"])["Year"].transform(len) #create obs column to count number of obs per station
    df = df[df["obs"] >= min_obs] #subset based on the minimum observation threshold
    df = df.drop(["obs"], axis = 1) #delete the observation column
    df = df.reset_index()
    df = df.groupby(["STATION_NAME", "LATITUDE", "LONGITUDE"]).apply(coef).reset_index() #create coefficients for our instances
    df = df.rename(columns = {0: "Estimated_Yearly_Delta"}) #rename from 0 to a more useful columnn name
    df["Estimated_Yearly_Delta"] = round(df["Estimated_Yearly_Delta"],3) #round sigfigs to 3
    
    #plotting process
    main = "Estimated Yearly Change in Temperature for Stations in {0} from {1} to {2} in {3}".format(
        country, year_begin, year_end, calendar.month_name[month] ) #main title using params
    fig = px.scatter_mapbox(df, lat = "LATITUDE", lon="LONGITUDE", title=main, hover_name = "STATION_NAME",
                           color = "Estimated_Yearly_Delta", **kwargs)
    fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
    
    return fig
    
```

Now that we finished implementing our function, let's try an example plot!


```python
# assumes you have imported necessary packages
color_map = px.colors.diverging.RdGy_r # choose a colormap


fig = temperature_coefficient_plot("India", 1980, 2020, 1, 
                                   min_obs = 10,
                                   zoom = 2,
                                   mapbox_style="carto-positron",
                                   color_continuous_scale=color_map)

fig.show()

```


<div>                            <div id="f9e38b18-3cd2-4918-90af-b3baf7b4fa2f" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("f9e38b18-3cd2-4918-90af-b3baf7b4fa2f")) {                    Plotly.newPlot(                        "f9e38b18-3cd2-4918-90af-b3baf7b4fa2f",                        [{"hovertemplate":"<b>%{hovertext}</b><br><br>LATITUDE=%{lat}<br>LONGITUDE=%{lon}<br>Estimated_Yearly_Delta=%{marker.color}<extra></extra>","hovertext":["AGARTALA","AHMADABAD","AKOLA","AKOLA","ALLAHABAD","ALLAHABAD_BAMHRAULI","AMRITSAR","AURANGABAD_CHIKALTH","BALASORE","BANGALORE","BAREILLY","BEGUMPETOBSY","BELGAUM_SAMBRA","BHOPAL_BAIRAGARH","BHUBANESWAR","BHUJ_RUDRAMATA","BIKANER","BOMBAY_COLABA","BOMBAY_SANTACRUZ","CALCUTTA_ALIPORE","CALCUTTA_DUM_DUM","CHERRAPUNJI","CHERRA_POONJEE","CHITRADURGA","COIMBATORE_PEELAMED","CUDDALORE","DALTONGANJ","DEHRADUN","DIBRUGARH_MOHANBAR","DUMKA","DWARKA","GADAG","GANGANAGAR","GAUHATI","GAYA","GOA_PANJIM","GORAKHPUR","GUNA","GWALIOR","HISSAR","HONAVAR","IMPHAL","INDORE","JABALPUR","JAGDALPUR","JAIPUR_SANGANER","JAISALMER","JHARSUGUDA","JODHPUR","KAKINADA","KARAIKAL","KODAIKANAL","KOTA_AERODROME","KOZHIKODE","KURNOOL","LUCKNOW_AMAUSI","LUDHIANA","MACHILIPATNAM","MADRAS_MINAMBAKKAM","MANGALORE_BAJPE","MINICOY","MINICOYOBSY","MO_AMINI","MO_RANCHI","MUKTESWAR_KUMAON","NAGPUR_SONEGAON","NELLORE","NEW_DELHI_PALAM","NEW_DELHI_SAFDARJUN","NORTH_LAKHIMPUR","PAMBAN","PATIALA","PATNA","PBO_ANANTAPUR","PENDRA_ROAD","POONA","PORT_BLAIR","RAIPUR","RAJKOT","RAMGUNDAM","RATNAGIRI","SAGAR","SANDHEADS","SATNA","SHILONG","SHIMLA","SHOLAPUR","SILCHAR","SRINAGAR","SURAT","TEZPUR","THIRUVANANTHAPURAM","THIRUVANANTHAPURAM","TIRUCHCHIRAPALLI","TRIVANDRUM","UDAIPUR_DABOK","VARANASI_BABATPUR","VERAVAL","VISHAKHAPATNAM"],"lat":[23.883000000000003,23.066999999999997,20.7,20.7,25.441,25.5,31.71,19.85,21.517,12.967,28.366999999999997,17.45,15.85,23.283,20.25,23.25,28.0,18.9,19.117,22.533,22.65,25.25,25.25,14.232999999999999,11.033,11.767000000000001,24.05,30.316999999999997,27.483,24.267,22.3667,15.417,29.916999999999998,26.1,24.75,15.482999999999999,26.75,24.65,26.233,29.166999999999998,14.283,24.666999999999998,22.717,23.2,19.083,26.816999999999997,26.9,21.916999999999998,26.3,16.95,10.917,10.2333,25.15,11.25,15.8,26.75,30.9333,16.2,13.0,12.917,8.3,8.3,11.117,23.316999999999997,29.4667,21.1,14.45,28.566999999999997,28.583000000000002,27.233,9.267000000000001,30.333000000000002,25.6,14.583,22.767,18.533,11.667,21.217,22.3,18.767,16.983,23.85,20.85,24.566999999999997,25.6,31.1,17.667,24.82,34.083,21.2,26.616999999999997,8.467,8.482999999999999,10.767000000000001,8.5,24.616999999999997,25.45,20.9,17.717],"legendgroup":"","lon":[91.25,72.633,77.033,77.067,81.735,81.9,74.797,75.4,86.93299999999999,77.583,79.4,78.47,74.617,77.35,85.833,69.667,73.3,72.8167,72.85,88.333,88.45,91.7333,91.73,76.433,77.05,79.767,84.06700000000001,78.033,95.01700000000001,87.25,69.0833,75.633,73.917,91.583,84.95,73.817,83.367,77.317,78.25,75.733,74.45,93.9,75.8,79.95,82.03299999999999,75.8,70.917,84.083,73.017,82.23299999999999,79.833,77.4667,75.85,75.783,78.067,80.883,75.8667,81.15,80.183,74.883,73.15,73.0,72.733,85.31700000000001,79.65,79.05,79.983,77.117,77.2,94.117,79.3,76.467,85.1,77.633,81.9,73.85,92.71700000000001,81.667,70.783,79.433,73.333,78.75,88.25,80.833,91.89,77.167,75.9,92.83,74.833,72.833,92.78299999999999,76.95,76.95,78.717,77.0,73.883,82.867,70.367,83.23299999999999],"marker":{"color":[-0.006,0.007,-0.002,-0.006,-0.029,-0.015,-0.007,0.012,-0.009,0.036,-0.017,0.011,-0.004,-0.022,-0.016,0.07,0.017,0.011,0.035,-0.01,-0.016,0.047,-0.022,0.016,0.034,0.021,-0.005,0.044,0.052,-0.082,0.017,0.009,-0.008,0.014,-0.018,0.022,0.004,-0.009,-0.018,-0.003,-0.049,0.063,0.003,-0.025,-0.0,0.007,0.003,-0.018,0.01,0.026,0.021,0.069,-0.019,0.033,0.026,-0.037,0.132,0.012,0.024,0.023,0.025,0.014,-0.006,0.005,0.015,-0.025,0.023,0.006,-0.002,0.095,0.025,-0.012,-0.022,0.026,-0.01,-0.0,0.031,0.001,0.016,-0.009,0.003,0.002,-0.052,-0.023,0.013,0.004,0.004,0.016,0.049,0.023,0.039,0.009,0.021,0.015,0.023,0.072,-0.013,0.025,-0.034],"coloraxis":"coloraxis"},"mode":"markers","name":"","showlegend":false,"subplot":"mapbox","type":"scattermapbox"}],                        {"coloraxis":{"colorbar":{"title":{"text":"Estimated_Yearly_Delta"}},"colorscale":[[0.0,"rgb(26,26,26)"],[0.1,"rgb(77,77,77)"],[0.2,"rgb(135,135,135)"],[0.3,"rgb(186,186,186)"],[0.4,"rgb(224,224,224)"],[0.5,"rgb(255,255,255)"],[0.6,"rgb(253,219,199)"],[0.7,"rgb(244,165,130)"],[0.8,"rgb(214,96,77)"],[0.9,"rgb(178,24,43)"],[1.0,"rgb(103,0,31)"]]},"legend":{"tracegroupgap":0},"mapbox":{"center":{"lat":21.042010101010103,"lon":79.63467373737375},"domain":{"x":[0.0,1.0],"y":[0.0,1.0]},"style":"carto-positron","zoom":2},"margin":{"b":0,"l":0,"r":0,"t":0},"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"title":{"text":"Estimated Yearly Change in Temperature for Stations in India from 1980 to 2020 in January"}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('f9e38b18-3cd2-4918-90af-b3baf7b4fa2f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



```python
from plotly.io import write_html
write_html(fig, "/Users/qianzach/Desktop/PIC_16B/BP1_indian_stations.html") #save our graph
```

## §4. Create Two More Interesting Figures

Now, we want to make 2 more interesting and insightful visualizations. Therefore, let's pose these two questions:

**1. Is there a relationship between latitude and average temperature change over the years?**

**2. Is there a relationship between longitude and average temperature change over the years?**

### (1) Latitude vs. Estimated Yearly Change in Average Temperature

We first want to make an abridged version of our `query_climate_database()` function and make it generalizable to multiple countries.


```python
def query_climate_database_facet(year_begin, year_end):
    """
    This function works similarly to query_climate_database() except it accepts 3 params.
    
    More specifically, this allows for all countries to be accepted!
    
    """
    
    cmd = \
    """
    SELECT S.NAME, S.LATITUDE, S.LONGITUDE,C.Name, T.Year, T.Month, T.Temp
    FROM temperatures T
    LEFT JOIN countries C on C.'FIPS10-4' = SUBSTRING(T.ID, 1, 2)
    LEFT JOIN stations S ON S.ID = T.ID
    
    WHERE T.YEAR BETWEEN ? AND ?

    """
    conn = sqlite3.connect("climate.db") #connect to climate database
    df = pd.read_sql_query(cmd, conn, params=( year_begin, year_end)) #make call w/ db using given params
    df = df.rename(columns={"NAME":"STATION_NAME","Name":"Country"}) #rename column name of "Name" to Country
    conn.close() #close connection
    return df #return dataframe
    
```

Let's see the output!


```python
query_climate_database_facet(  year_begin = 1980, 
                       year_end = 1981).head()
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
      <th>STATION_NAME</th>
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
      <td>SAVE</td>
      <td>57.7667</td>
      <td>11.8667</td>
      <td>Antigua and Barbuda</td>
      <td>1980</td>
      <td>1</td>
      <td>-2.71</td>
    </tr>
    <tr>
      <th>1</th>
      <td>SAVE</td>
      <td>57.7667</td>
      <td>11.8667</td>
      <td>Antigua and Barbuda</td>
      <td>1980</td>
      <td>2</td>
      <td>-4.31</td>
    </tr>
    <tr>
      <th>2</th>
      <td>SAVE</td>
      <td>57.7667</td>
      <td>11.8667</td>
      <td>Antigua and Barbuda</td>
      <td>1980</td>
      <td>3</td>
      <td>0.34</td>
    </tr>
    <tr>
      <th>3</th>
      <td>SAVE</td>
      <td>57.7667</td>
      <td>11.8667</td>
      <td>Antigua and Barbuda</td>
      <td>1980</td>
      <td>4</td>
      <td>5.93</td>
    </tr>
    <tr>
      <th>4</th>
      <td>SAVE</td>
      <td>57.7667</td>
      <td>11.8667</td>
      <td>Antigua and Barbuda</td>
      <td>1980</td>
      <td>5</td>
      <td>11.40</td>
    </tr>
  </tbody>
</table>
</div>




```python
from plotly import express as px
import plotly.io as pio
import calendar
def lat_plot(country_list, year_begin, year_end, **kwargs):
    """
    Goal is to plot scatterplots with our explanatory variable being latitude and response variable as change in temperature
    """
    #data manipulation/computation
    df = query_climate_database_facet(year_begin, year_end) #make query
    coefs = df.groupby(["STATION_NAME", "Month"]).apply(coef) #compute coefficients

    df = df[df["Country"].isin(country_list)] #slice based on our country list
    df = df.groupby(["STATION_NAME", "LATITUDE", "LONGITUDE"]).apply(coef).reset_index() #create coefficients for our instances
    df = df.rename(columns = {0: "Estimated_Yearly_Delta"}) #rename from 0 to a more useful columnn name
    df["Estimated_Yearly_Delta"] = round(df["Estimated_Yearly_Delta"],3) #round sigfigs to 3
    
    pio.templates.default = "plotly_white"
    fig = px.scatter(data_frame = df, 
                 x = "LATITUDE", 
                 y = "Estimated_Yearly_Delta",
                 color = "Country",
                 hover_name = "Species",
                 hover_data = ["Year", "Month"],
                 width = 600,
                 height = 400,
                 opacity = 0.5,
                 marginal_y = "box",
                 marginal_x = "rug")
    return fig
    
    
```


```python
countries = ["China", "India"]
fig2 = lat_plot(countries, 1980, 1981)
fig2.show()
```


    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-125-8d18a6d920c0> in <module>
          1 countries = ["China", "India"]
    ----> 2 fig2 = lat_plot(countries, 1980, 1981)
          3 fig2.show()


    <ipython-input-119-1078cdecd573> in lat_plot(country_list, year_begin, year_end, **kwargs)
          8     #data manipulation/computation
          9     df = query_climate_database_facet(year_begin, year_end) #make query
    ---> 10     coefs = df.groupby(["STATION_NAME", "Month"]).apply(coef) #compute coefficients
         11 
         12     df = df[df["Country"].isin(country_list)] #slice based on our country list


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/groupby/groupby.py in apply(self, func, *args, **kwargs)
        857         with option_context("mode.chained_assignment", None):
        858             try:
    --> 859                 result = self._python_apply_general(f, self._selected_obj)
        860             except TypeError:
        861                 # gh-20949


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/groupby/groupby.py in _python_apply_general(self, f, data)
        890             data after applying f
        891         """
    --> 892         keys, values, mutated = self.grouper.apply(f, data, self.axis)
        893 
        894         return self._wrap_applied_output(


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/groupby/ops.py in apply(self, f, data, axis)
        177         ):
        178             try:
    --> 179                 result_values, mutated = splitter.fast_apply(f, sdata, group_keys)
        180 
        181             except libreduction.InvalidApply as err:


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/groupby/ops.py in fast_apply(self, f, sdata, names)
        963         # must return keys::list, values::list, mutated::bool
        964         starts, ends = lib.generate_slices(self.slabels, self.ngroups)
    --> 965         return libreduction.apply_frame_axis0(sdata, f, names, starts, ends)
        966 
        967     def _chop(self, sdata: DataFrame, slice_obj: slice) -> DataFrame:


    pandas/_libs/reduction.pyx in pandas._libs.reduction.apply_frame_axis0()


    <ipython-input-36-97b9033d28ae> in coef(data_group)
          7     y = data_group["Temp"]   # 1 bracket because y should be a series
          8     LR = LinearRegression()
    ----> 9     LR.fit(x, y)
         10     return LR.coef_[0]


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/sklearn/linear_model/_base.py in fit(self, X, y, sample_weight)
        504         n_jobs_ = self.n_jobs
        505         X, y = self._validate_data(X, y, accept_sparse=['csr', 'csc', 'coo'],
    --> 506                                    y_numeric=True, multi_output=True)
        507 
        508         if sample_weight is not None:


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/sklearn/base.py in _validate_data(self, X, y, reset, validate_separately, **check_params)
        430                 y = check_array(y, **check_y_params)
        431             else:
    --> 432                 X, y = check_X_y(X, y, **check_params)
        433             out = X, y
        434 


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/sklearn/utils/validation.py in inner_f(*args, **kwargs)
         70                           FutureWarning)
         71         kwargs.update({k: arg for k, arg in zip(sig.parameters, args)})
    ---> 72         return f(**kwargs)
         73     return inner_f
         74 


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/sklearn/utils/validation.py in check_X_y(X, y, accept_sparse, accept_large_sparse, dtype, order, copy, force_all_finite, ensure_2d, allow_nd, multi_output, ensure_min_samples, ensure_min_features, y_numeric, estimator)
        800                     ensure_min_samples=ensure_min_samples,
        801                     ensure_min_features=ensure_min_features,
    --> 802                     estimator=estimator)
        803     if multi_output:
        804         y = check_array(y, accept_sparse='csr', force_all_finite=True,


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/sklearn/utils/validation.py in inner_f(*args, **kwargs)
         70                           FutureWarning)
         71         kwargs.update({k: arg for k, arg in zip(sig.parameters, args)})
    ---> 72         return f(**kwargs)
         73     return inner_f
         74 


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/sklearn/utils/validation.py in check_array(array, accept_sparse, accept_large_sparse, dtype, order, copy, force_all_finite, ensure_2d, allow_nd, ensure_min_samples, ensure_min_features, estimator)
        505             from pandas.api.types import is_sparse
        506             if (not hasattr(array, 'sparse') and
    --> 507                     array.dtypes.apply(is_sparse).any()):
        508                 warnings.warn(
        509                     "pandas.DataFrame with sparse columns found."


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/series.py in apply(self, func, convert_dtype, args, **kwds)
       4205             return self._constructor_expanddim(pd.array(mapped), index=self.index)
       4206         else:
    -> 4207             return self._constructor(mapped, index=self.index).__finalize__(
       4208                 self, method="apply"
       4209             )


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/series.py in __init__(self, data, index, dtype, name, copy, fastpath)
        325                     data = data.copy()
        326             else:
    --> 327                 data = sanitize_array(data, index, dtype, copy, raise_cast_failure=True)
        328 
        329                 data = SingleBlockManager.from_array(data, index)


    ~/opt/anaconda3/envs/PIC16B/lib/python3.7/site-packages/pandas/core/construction.py in sanitize_array(data, index, dtype, copy, raise_cast_failure)
        510                 subarr = np.array(data, dtype=object, copy=copy)
        511 
    --> 512         if is_object_dtype(subarr.dtype) and not is_object_dtype(dtype):
        513             inferred = lib.infer_dtype(subarr, skipna=False)
        514             if inferred in {"interval", "period"}:


    KeyboardInterrupt: 


### (2) Longitude vs. Estimated Yearly Change in Average Temperature


```python
def long_plot(country_list, year_begin, year_end, **kwargs):
    """
    Goal is to plot scatterplots with our explanatory variable being longitude and response variable as change in temperature
    """
    #data manipulation/computation
    df = query_climate_database_facet(year_begin, year_end) #make query
    coefs = df.groupby(["STATION_NAME", "Month"]).apply(coef) #compute coefficients

    df = df[df["Country"].isin(country_list)] #slice based on our country list
    df = df.groupby(["STATION_NAME", "LATITUDE", "LONGITUDE"]).apply(coef).reset_index() #create coefficients for our instances
    df = df.rename(columns = {0: "Estimated_Yearly_Delta"}) #rename from 0 to a more useful columnn name
    df["Estimated_Yearly_Delta"] = round(df["Estimated_Yearly_Delta"],3) #round sigfigs to 3
    
    pio.templates.default = "plotly_white"
    fig = px.scatter(data_frame = df, 
                 x = "LONGITUDE", 
                 y = "Estimated_Yearly_Delta",
                 color = "Country",
                 hover_name = "Species",
                 hover_data = ["Year", "Month"],
                 width = 600,
                 height = 400,
                 opacity = 0.5,
                 marginal_y = "box",
                 marginal_x = "rug")
    return fig
    
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
      <th>STATION_NAME</th>
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
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1980</td>
      <td>1</td>
      <td>23.48</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1981</td>
      <td>1</td>
      <td>24.57</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1982</td>
      <td>1</td>
      <td>24.19</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1983</td>
      <td>1</td>
      <td>23.51</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PBO_ANANTAPUR</td>
      <td>14.583</td>
      <td>77.633</td>
      <td>India</td>
      <td>1984</td>
      <td>1</td>
      <td>24.81</td>
    </tr>
  </tbody>
</table>
</div>




```python
countries = ["China", "India"]
fig3 = lat_plot(countries, 1990, 1991)
fig3.show()
```
