# Is it faster to walk through Spadina Station to the Bloor Line, or stay on the train and transfer at St. George?
*March 2, 2022*

There's a question that Torontians regularly ask when taking the Yonge-University subway southbouth and transfering to the Westbound Bloor Line: is it faster to get off at Spadina and walk, or to continue on the train and transfer at St. George?

The problem is that between the two lines at Spadina is a long walkway, and it takes several minutes to get to the Bloor Line platform. But at St. George Station, another transfer station, it's a 5 second walk up the stairs to get to the Bloor platform. But of course, you need to ride the train to St. George and then back to Spadina, so it adds time.

So: which way is faster?

Let's use [route and stop data](https://open.toronto.ca/dataset/ttc-routes-and-schedules/) provided by the City of Toronto to figure it out. We'll use pandas for analysis and Python's datetime module to handle datetime and timedelta objects.


```python
import pandas as pd
import datetime
```

There are many different datasets in the file we downloaded. We'll use these four for the analysis.


```python
stop_times = pd.read_csv("schedule_data/stop_times.txt")
routes = pd.read_csv("schedule_data/routes.txt")
stops = pd.read_csv("schedule_data/stops.txt")
trips = pd.read_csv("schedule_data/trips.txt")
```

The numbers we need to solve this problem ultimately are in the stop_times dataframe.


```python
stop_times.head(5)
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
      <th>trip_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
      <th>stop_id</th>
      <th>stop_sequence</th>
      <th>stop_headsign</th>
      <th>pickup_type</th>
      <th>drop_off_type</th>
      <th>shape_dist_traveled</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>45417218</td>
      <td>9:15:00</td>
      <td>9:15:00</td>
      <td>14155</td>
      <td>1</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>45417218</td>
      <td>9:16:20</td>
      <td>9:16:20</td>
      <td>3807</td>
      <td>2</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0.3546</td>
    </tr>
    <tr>
      <th>2</th>
      <td>45417218</td>
      <td>9:17:13</td>
      <td>9:17:13</td>
      <td>6904</td>
      <td>3</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0.5903</td>
    </tr>
    <tr>
      <th>3</th>
      <td>45417218</td>
      <td>9:18:36</td>
      <td>9:18:36</td>
      <td>1163</td>
      <td>4</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0.9613</td>
    </tr>
    <tr>
      <th>4</th>
      <td>45417218</td>
      <td>9:19:49</td>
      <td>9:19:49</td>
      <td>7723</td>
      <td>5</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>1.2849</td>
    </tr>
  </tbody>
</table>
</div>



The trip_id column is going to be important. Let's see what the dataframe looks like.


```python
trips.head(5)
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
      <th>route_id</th>
      <th>service_id</th>
      <th>trip_id</th>
      <th>trip_headsign</th>
      <th>trip_short_name</th>
      <th>direction_id</th>
      <th>block_id</th>
      <th>shape_id</th>
      <th>wheelchair_accessible</th>
      <th>bikes_allowed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>68631</td>
      <td>1</td>
      <td>45417228</td>
      <td>EAST - 10 VAN HORNE towards VICTORIA PARK</td>
      <td>NaN</td>
      <td>0</td>
      <td>1965187</td>
      <td>953035</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>68631</td>
      <td>1</td>
      <td>45417233</td>
      <td>EAST - 10 VAN HORNE towards VICTORIA PARK</td>
      <td>NaN</td>
      <td>0</td>
      <td>1965187</td>
      <td>953036</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>68631</td>
      <td>1</td>
      <td>45417223</td>
      <td>EAST - 10 VAN HORNE towards VICTORIA PARK</td>
      <td>NaN</td>
      <td>0</td>
      <td>1965187</td>
      <td>953036</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>68631</td>
      <td>1</td>
      <td>45417222</td>
      <td>EAST - 10 VAN HORNE towards VICTORIA PARK</td>
      <td>NaN</td>
      <td>0</td>
      <td>1965187</td>
      <td>953036</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>68631</td>
      <td>1</td>
      <td>45417221</td>
      <td>EAST - 10 VAN HORNE towards VICTORIA PARK</td>
      <td>NaN</td>
      <td>0</td>
      <td>1965187</td>
      <td>953036</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Here we have another interesting column, and that's route_id. Let's take a look at the route dataframe now!


```python
routes.head(5)
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
      <th>route_id</th>
      <th>agency_id</th>
      <th>route_short_name</th>
      <th>route_long_name</th>
      <th>route_desc</th>
      <th>route_type</th>
      <th>route_url</th>
      <th>route_color</th>
      <th>route_text_color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>68840</td>
      <td>1</td>
      <td>1</td>
      <td>LINE 1 (YONGE-UNIVERSITY)</td>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>D5C82B</td>
      <td>000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>68631</td>
      <td>1</td>
      <td>10</td>
      <td>VAN HORNE</td>
      <td>NaN</td>
      <td>3</td>
      <td>NaN</td>
      <td>FF0000</td>
      <td>FFFFFF</td>
    </tr>
    <tr>
      <th>2</th>
      <td>68632</td>
      <td>1</td>
      <td>100</td>
      <td>FLEMINGDON PARK</td>
      <td>NaN</td>
      <td>3</td>
      <td>NaN</td>
      <td>FF0000</td>
      <td>FFFFFF</td>
    </tr>
    <tr>
      <th>3</th>
      <td>68633</td>
      <td>1</td>
      <td>101</td>
      <td>DOWNSVIEW PARK</td>
      <td>NaN</td>
      <td>3</td>
      <td>NaN</td>
      <td>FF0000</td>
      <td>FFFFFF</td>
    </tr>
    <tr>
      <th>4</th>
      <td>68634</td>
      <td>1</td>
      <td>102</td>
      <td>MARKHAM RD.</td>
      <td>NaN</td>
      <td>3</td>
      <td>NaN</td>
      <td>FF0000</td>
      <td>FFFFFF</td>
    </tr>
  </tbody>
</table>
</div>



Taking a look at these three dataframes gives us a clue as to how the data is organized. We need to find the trips that the Yonge-University Line and the Bloor Line take between the stops of Spadina and St. George.

Let's find the route ids for our two TTC lines.


```python
ttc_routes = routes[routes["route_long_name"].str.contains("LINE 1|LINE 2")]

ttc_routes
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
      <th>route_id</th>
      <th>agency_id</th>
      <th>route_short_name</th>
      <th>route_long_name</th>
      <th>route_desc</th>
      <th>route_type</th>
      <th>route_url</th>
      <th>route_color</th>
      <th>route_text_color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>68840</td>
      <td>1</td>
      <td>1</td>
      <td>LINE 1 (YONGE-UNIVERSITY)</td>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>D5C82B</td>
      <td>000000</td>
    </tr>
    <tr>
      <th>52</th>
      <td>68841</td>
      <td>1</td>
      <td>2</td>
      <td>LINE 2 (BLOOR - DANFORTH)</td>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>008000</td>
      <td>FFFFFF</td>
    </tr>
  </tbody>
</table>
</div>



Now let's see the trips that these two routes take.


```python
ttc_trips = trips[trips["route_id"].isin(ttc_routes["route_id"].astype(int).to_list())]

ttc_trips.head(3)
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
      <th>route_id</th>
      <th>service_id</th>
      <th>trip_id</th>
      <th>trip_headsign</th>
      <th>trip_short_name</th>
      <th>direction_id</th>
      <th>block_id</th>
      <th>shape_id</th>
      <th>wheelchair_accessible</th>
      <th>bikes_allowed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>36644</th>
      <td>68840</td>
      <td>1</td>
      <td>45543404</td>
      <td>LINE 1 (YONGE-UNIVERSITY) towards VAUGHAN METR...</td>
      <td>NaN</td>
      <td>0</td>
      <td>1970948</td>
      <td>956429</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>36645</th>
      <td>68840</td>
      <td>1</td>
      <td>45543414</td>
      <td>LINE 1 (YONGE-UNIVERSITY) towards VAUGHAN METR...</td>
      <td>NaN</td>
      <td>0</td>
      <td>1970946</td>
      <td>956429</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>36646</th>
      <td>68840</td>
      <td>1</td>
      <td>45543426</td>
      <td>LINE 1 (YONGE-UNIVERSITY) towards VAUGHAN METR...</td>
      <td>NaN</td>
      <td>0</td>
      <td>1970945</td>
      <td>956429</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



Now let's take a look at the stops dataset. 


```python
stops.head(3)
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
      <th>stop_id</th>
      <th>stop_code</th>
      <th>stop_name</th>
      <th>stop_desc</th>
      <th>stop_lat</th>
      <th>stop_lon</th>
      <th>zone_id</th>
      <th>stop_url</th>
      <th>location_type</th>
      <th>parent_station</th>
      <th>stop_timezone</th>
      <th>wheelchair_boarding</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>262</td>
      <td>662</td>
      <td>Danforth Rd at Kennedy Rd</td>
      <td>NaN</td>
      <td>43.714379</td>
      <td>-79.260939</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>263</td>
      <td>929</td>
      <td>Davenport Rd at Bedford Rd</td>
      <td>NaN</td>
      <td>43.674448</td>
      <td>-79.399659</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>264</td>
      <td>940</td>
      <td>Davenport Rd at Dupont St</td>
      <td>NaN</td>
      <td>43.675511</td>
      <td>-79.401938</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



These are all the stops on both lines, but we're only really interested in a subset:
* The trips on Yonge-University Line from Spadina to St. George.
* The trips on Bloor Line from St. George to Spadina.

It's hard to know what we're looking at without stop names in the trips dataset, so let's join stop names by stop_ids onto our trips dataset.


```python
ttc_trip_ids = ttc_trips["trip_id"].unique()

ttc_stop_times = (stop_times[stop_times["trip_id"].isin(ttc_trip_ids)]
                    .set_index("stop_id")
                    .join(stops.set_index("stop_id")[["stop_name"]])
                    )

ttc_stop_times.head(5)
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
      <th>trip_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
      <th>stop_sequence</th>
      <th>stop_headsign</th>
      <th>pickup_type</th>
      <th>drop_off_type</th>
      <th>shape_dist_traveled</th>
      <th>stop_name</th>
    </tr>
    <tr>
      <th>stop_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14404</th>
      <td>45543404</td>
      <td>5:31:21</td>
      <td>5:31:21</td>
      <td>1</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>Finch Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14404</th>
      <td>45543414</td>
      <td>5:36:08</td>
      <td>5:36:08</td>
      <td>1</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>Finch Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14404</th>
      <td>45543419</td>
      <td>6:13:49</td>
      <td>6:13:49</td>
      <td>1</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>Finch Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14404</th>
      <td>45543420</td>
      <td>6:09:07</td>
      <td>6:09:07</td>
      <td>1</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>Finch Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14404</th>
      <td>45543421</td>
      <td>6:04:25</td>
      <td>6:04:25</td>
      <td>1</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>Finch Station - Southbound Platform</td>
    </tr>
  </tbody>
</table>
</div>



By taking a look at the stop_name column, we can see the naming convention for the stations, and can guess at what our stations will be called and can filter our stops list for those.


```python
ttc_stop_times = (ttc_stop_times[ttc_stop_times["stop_name"]
                                     .str.contains("St George Station - Southbound Platform|Spadina Station - Southbound Platform|St George Station - Westbound Platform|Spadina Station - Westbound Platform")]
                    )

ttc_stop_times["stop_name"].unique()
```




    array(['Spadina Station - Southbound Platform',
           'St George Station - Southbound Platform',
           'St George Station - Westbound Platform',
           'Spadina Station - Westbound Platform'], dtype=object)



Now that we have a rough subset of the data that we're interested in, let's take a second to fix a problem we have in our arrival_time and departure_time columns: the timecode is a string. Pandas normally doesn't have a hard time handling this, but let's take a look at the error it throws if we try to convert.


```python
try: pd.to_datetime(ttc_stop_times["arrival_time"])
except Exception as e:
    print(e)
```

    hour must be in 0..23: 24:22:46
    

Hard to fault pandas...there is no hour 24 in a day. This hints at a strange way this TTC date is organized. Rather than report 2 a.m. as 02:00:00, it reports it as 26:00:00. The reason is the data wants to show a continous day's schedule as one unit. If it reported the time as 2 a.m., it would show at the start of the day, and if 2 a.m. was the last route run that day, it would then have a huge gap before the next day started.

We can fix this to better suit our purposes, though. Let's define a little function that we can apply to the column to turn the string into a datetime.


```python
def clean_time(time_string):
    
    # Take the time string in this row and split it by colons.
    # The first item in the list becomes the hour, the second the minute, and the third the seconds.
    time_list = time_string.split(":")
    
    # If the hour is less than 23...
    if int(time_list[0]) < 24:
        
        # Report the hour as-is.
        hour = int(time_list[0])
        
        # Assign an arbitrary but consisent day. In this case, we'll pretend the day is March 2, 2022.
        day = 2
        
    # If the hour is greater than 23...
    else:
        
        # Subtract 23 from it to get the hour we know it as.
        hour = int(time_list[0]) - 23
        
        # And instead report it as the next day.
        day = 3
    
    # Report the minute as an integer.
    minute = int(time_list[1])
    
    # Report the second as an integer.
    seconds = int(time_list[2])
    
    # Return the whole thing built as a datetime object.
    return datetime.datetime(year=2022, month=3, day=day, hour=hour, minute=minute, second=seconds)
```

Let's try it!


```python
ttc_stop_times["arrival_time"] = ttc_stop_times["arrival_time"].apply(lambda x: clean_time(x))
ttc_stop_times["departure_time"] = ttc_stop_times["departure_time"].apply(lambda x: clean_time(x))

ttc_stop_times.head(3)
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
      <th>trip_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
      <th>stop_sequence</th>
      <th>stop_headsign</th>
      <th>pickup_type</th>
      <th>drop_off_type</th>
      <th>shape_dist_traveled</th>
      <th>stop_name</th>
    </tr>
    <tr>
      <th>stop_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14444</th>
      <td>45543667</td>
      <td>2022-03-02 06:50:26</td>
      <td>2022-03-02 06:50:26</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45543669</td>
      <td>2022-03-02 06:44:46</td>
      <td>2022-03-02 06:44:46</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45543670</td>
      <td>2022-03-02 06:41:56</td>
      <td>2022-03-02 06:41:56</td>
      <td>8</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>9.3982</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
  </tbody>
</table>
</div>



Let's also take a peek at the end of the dataset here to make sure our early-morning hours went into the next day.


```python
ttc_stop_times.tail(3)
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
      <th>trip_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
      <th>stop_sequence</th>
      <th>stop_headsign</th>
      <th>pickup_type</th>
      <th>drop_off_type</th>
      <th>shape_dist_traveled</th>
      <th>stop_name</th>
    </tr>
    <tr>
      <th>stop_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14515</th>
      <td>45547524</td>
      <td>2022-03-03 02:44:00</td>
      <td>2022-03-03 02:44:00</td>
      <td>17</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>14.843</td>
      <td>Spadina Station - Westbound Platform</td>
    </tr>
    <tr>
      <th>14515</th>
      <td>45547525</td>
      <td>2022-03-03 02:50:00</td>
      <td>2022-03-03 02:50:00</td>
      <td>17</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>14.843</td>
      <td>Spadina Station - Westbound Platform</td>
    </tr>
    <tr>
      <th>14515</th>
      <td>45547526</td>
      <td>2022-03-03 02:56:00</td>
      <td>2022-03-03 02:56:00</td>
      <td>17</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>14.843</td>
      <td>Spadina Station - Westbound Platform</td>
    </tr>
  </tbody>
</table>
</div>



Success! Now that we have our subset with clean datetime columns, we can move forward with our analysis.

### Segment times

The way we solve this problem is by comparing two trips:
* Walking from Spadina (Yonge-University) to Spadina (Bloor) and waiting for the next train.
* Staying on the train and going from Spadina to St. George, waiting for the next train, then riding the train from St. George to Spadina.

A critical part of estimating the second trip is knowing how long the train takes to go between Spadina and St. George, then back to Spadina. Let's figure that out now.

We'll isolate one such trip in our dataset here.


```python
test_trip = ttc_stop_times[ttc_stop_times["trip_id"] == 45544278]

test_trip
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
      <th>trip_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
      <th>stop_sequence</th>
      <th>stop_headsign</th>
      <th>pickup_type</th>
      <th>drop_off_type</th>
      <th>shape_dist_traveled</th>
      <th>stop_name</th>
    </tr>
    <tr>
      <th>stop_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14444</th>
      <td>45544278</td>
      <td>2022-03-02 22:59:19</td>
      <td>2022-03-02 22:59:19</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14445</th>
      <td>45544278</td>
      <td>2022-03-02 23:00:47</td>
      <td>2022-03-02 23:00:47</td>
      <td>16</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>20.1374</td>
      <td>St George Station - Southbound Platform</td>
    </tr>
  </tbody>
</table>
</div>



How long does this trip take? We can subtract the departure_time in the first row from the arrival_time in the second row.

It's worth noting here that in most (or maybe all) cases, the arrival_time and the departure_time are the same, and so we can probably just use arrival_time for both. Interesting that the schedules do not account for stopping to let people on. Our analysis will pretend that takes no time, as well. In reality it's likely just a few seconds it's stopped for, anyways.


```python
test_trip.iat[1,1] - test_trip.iat[0,2]
```




    Timedelta('0 days 00:01:28')



This test trip we pulled out takes about a minute and a half. It's probably safe enough to assume that every trip will take the same amount of time. These are schedules, after all, not actual times of arrival and departure. But to be safe, let's do this for all trips and average them out.


```python
frames = []

for trip_id in ttc_stop_times["trip_id"].unique():
    trip = ttc_stop_times[ttc_stop_times["trip_id"] == trip_id].sort_values("arrival_time")

    if len(trip) > 1:
        segment_duration = trip.iat[1,1] - trip.iat[0,2]

        segment_name = trip.iat[0,-1] + " to " + trip.iat[1,-1]

        df = pd.DataFrame({"segment_name": [segment_name], "duration": [segment_duration]})
        
        frames.append(df)
    else:
        pass
    
segment_times = pd.concat(frames).pivot_table(index="segment_name", values="duration", aggfunc="mean")

segment_times
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
      <th>duration</th>
    </tr>
    <tr>
      <th>segment_name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Spadina Station - Southbound Platform to St George Station - Southbound Platform</th>
      <td>0 days 00:01:41.585714285</td>
    </tr>
    <tr>
      <th>St George Station - Westbound Platform to Spadina Station - Westbound Platform</th>
      <td>0 days 00:01:00</td>
    </tr>
  </tbody>
</table>
</div>



These numbers actually give us a pretty good estimate of how long the second trip - not walking, but staying on the train - would take!


```python
segment_times["duration"].sum()
```




    Timedelta('0 days 00:02:41.585714285')



Of course, it's more complicated than this: what if you're waiting forever for that train at St. George? You do have to transfer, after all. So we have more work to do. But we'll refer to these calculations when we do our math down below.

### The math

We're going to split our dataset into three now: the stop times at each station: Spadina heading southbound, St. George heading west, and Spadina heading west.


```python
spadina_south = ttc_stop_times[ttc_stop_times["stop_name"] == "Spadina Station - Southbound Platform"].sort_values("arrival_time")

george_west = ttc_stop_times[ttc_stop_times["stop_name"] == "St George Station - Westbound Platform"].sort_values("arrival_time")

spadina_west = ttc_stop_times[ttc_stop_times["stop_name"] == "Spadina Station - Westbound Platform"].sort_values("arrival_time")
```


```python
trips[trips["trip_id"] == 45544882]
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
      <th>route_id</th>
      <th>service_id</th>
      <th>trip_id</th>
      <th>trip_headsign</th>
      <th>trip_short_name</th>
      <th>direction_id</th>
      <th>block_id</th>
      <th>shape_id</th>
      <th>wheelchair_accessible</th>
      <th>bikes_allowed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>95453</th>
      <td>68840</td>
      <td>3</td>
      <td>45544882</td>
      <td>LINE 1 (YONGE-UNIVERSITY) towards FINCH STATION</td>
      <td>NaN</td>
      <td>1</td>
      <td>1971048</td>
      <td>956608</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
spadina_south
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
      <th>trip_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
      <th>stop_sequence</th>
      <th>stop_headsign</th>
      <th>pickup_type</th>
      <th>drop_off_type</th>
      <th>shape_dist_traveled</th>
      <th>stop_name</th>
    </tr>
    <tr>
      <th>stop_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14444</th>
      <td>45543691</td>
      <td>2022-03-02 06:00:35</td>
      <td>2022-03-02 06:00:35</td>
      <td>3</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>2.0186</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45544229</td>
      <td>2022-03-02 06:03:18</td>
      <td>2022-03-02 06:03:18</td>
      <td>3</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>2.0186</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45545169</td>
      <td>2022-03-02 06:03:18</td>
      <td>2022-03-02 06:03:18</td>
      <td>3</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>2.0186</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45543703</td>
      <td>2022-03-02 06:06:56</td>
      <td>2022-03-02 06:06:56</td>
      <td>8</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>9.3982</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45544230</td>
      <td>2022-03-02 06:09:19</td>
      <td>2022-03-02 06:09:19</td>
      <td>8</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>9.3982</td>
      <td>Spadina Station - Southbound Platform</td>
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
    </tr>
    <tr>
      <th>14444</th>
      <td>45543851</td>
      <td>2022-03-03 02:32:46</td>
      <td>2022-03-03 02:32:46</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45544882</td>
      <td>2022-03-03 02:35:19</td>
      <td>2022-03-03 02:35:19</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45544438</td>
      <td>2022-03-03 02:35:19</td>
      <td>2022-03-03 02:35:19</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45545388</td>
      <td>2022-03-03 02:35:19</td>
      <td>2022-03-03 02:35:19</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
    <tr>
      <th>14444</th>
      <td>45543906</td>
      <td>2022-03-03 02:36:45</td>
      <td>2022-03-03 02:36:45</td>
      <td>15</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>19.4416</td>
      <td>Spadina Station - Southbound Platform</td>
    </tr>
  </tbody>
</table>
<p>980 rows × 9 columns</p>
</div>



We'll also store the times for each segment of trip #2 (not walking, staying on the train) that we found above into variables for use in a second.


```python
time_on_train_from_spadina_to_george = segment_times.at["Spadina Station - Southbound Platform to St George Station - Southbound Platform", "duration"]

time_on_train_from_george_to_spadina = segment_times.at["St George Station - Westbound Platform to Spadina Station - Westbound Platform", "duration"]
```

Now we're going to loop through every departure time from Spadina going south, which represents a scenario where you make two choices. In every case, you'll be on one of these trains already (if you lived near Spadina station, for example, you would always go to the westbound Bloor Line station - why would you not? Therefore, you'll always be on a train when the scenario starts).

Your two choices are:
* Get off the train and walk to the westbound Bloor Line platform at Spadina.
* Stay on the train, take it to St. George, wait for a train at St. George, and take it back towards Spadina.

So for each time that the train leaves Spadina, we'll tally up the time each trip takes.


```python
# An empty list where we'll store data temporarily as we go through the loop.
data = []

# For every stop time in the spadina_south dataset:
for i, stop in spadina_south.iterrows():
    
    ## Scenario #1: Walking to Spadina westbound platform.
    
    # I went and walked this distance for this variable, which is the time it takes to walk that tunnel to the other platform
    # at Spadina station.
    time_to_walk = datetime.timedelta(minutes=3, seconds=0)
    
    # The time you arrive at the Westbound platform is equal to the time the train arrived and dropped you off,
    # plus the time it takes to walk.
    you_arrive_at_spadina_west = stop["arrival_time"] + time_to_walk
    
    # Now you're at the station, and we need to find the next train that comes by the westbound platform after you arrive.
    next_train_leaves_from_spadina_west = (spadina_west
                                           .loc[spadina_west["arrival_time"] >= you_arrive_at_spadina_west, "arrival_time"]
                                           .min()
                                           )
    
    # Now the total length of your trip is the time from when you get off your train at Spadina south
    # to the time your new train arrives at Spadina west.
    travel_time_if_walking = next_train_leaves_from_spadina_west - stop["arrival_time"]
    
    
    
    ## Scenario #2: Staying on the train
    
    # The time you arrive at St. George station is equal to the time you arrived at Spadina south
    # plus the time the trip to St. George takes
    # plus the 10 seconds it takes to walk up the stairs to the westbound platform.
    you_arrive_at_george = stop["arrival_time"] + time_on_train_from_spadina_to_george + datetime.timedelta(seconds=10)
    
    # Now you're at St. George West, and you need to catch the next train. Let's find out when that is.
    time_next_train_leave_george = (george_west
                                    .loc[george_west["arrival_time"] >= you_arrive_at_george, "arrival_time"]
                                    .min()
                                    )
    
    # The time you arrive at Spadina west is equal to the time you leave St. George on that next train
    # plus the travel time from St. George to Spadina.
    time_train_arrives_at_spadina_west = time_next_train_leave_george + time_on_train_from_george_to_spadina
    
    # The travel time is the time between arriving at Spadina South and arriving at Spadina West.
    travel_time_if_not_walking = time_train_arrives_at_spadina_west - stop["arrival_time"]
    
    
    
    ## Now we store the relevant information in a pandas dataframe.
    entry = (pd.DataFrame({
                            "time_arriving_at_spadina": [stop["arrival_time"]],
                            "travel_time_if_walking": [travel_time_if_walking],
                            "travel_time_if_not_walking": [travel_time_if_not_walking],
                           })
             )
    
    # And we append this dataframe -- really just one row -- to our list.
    data.append(entry)

# When the loop is done, concatenate all the data into one frame.  
table = pd.concat(data)

# Show the frame!
table.sample(5)
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
      <th>time_arriving_at_spadina</th>
      <th>travel_time_if_walking</th>
      <th>travel_time_if_not_walking</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-02 07:57:19</td>
      <td>0 days 00:03:21</td>
      <td>0 days 00:03:21</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 08:15:19</td>
      <td>0 days 00:03:42</td>
      <td>0 days 00:03:42</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:57:19</td>
      <td>0 days 00:03:50</td>
      <td>0 days 00:03:50</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 21:27:19</td>
      <td>0 days 00:04:41</td>
      <td>0 days 00:04:41</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 13:17:06</td>
      <td>0 days 00:04:19</td>
      <td>0 days 00:04:19</td>
    </tr>
  </tbody>
</table>
</div>



The bulk of the math is done! Now we just need to make sense of it all. We'll make a boolean column that says whether or not you should walk at Spadina.


```python
table.loc[table["travel_time_if_not_walking"] > table["travel_time_if_walking"] , "better_scenario"] = "Walk"
table.loc[table["travel_time_if_not_walking"] < table["travel_time_if_walking"] , "better_scenario"] = "Stay on Train"
table.loc[table["travel_time_if_not_walking"] == table["travel_time_if_walking"] , "better_scenario"] = "Doesn't matter"

table.head(3)
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
      <th>time_arriving_at_spadina</th>
      <th>travel_time_if_walking</th>
      <th>travel_time_if_not_walking</th>
      <th>better_scenario</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:00:35</td>
      <td>0 days 00:04:50</td>
      <td>0 days 00:04:50</td>
      <td>Doesn't matter</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:03:18</td>
      <td>0 days 00:04:02</td>
      <td>0 days 00:04:02</td>
      <td>Doesn't matter</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:03:18</td>
      <td>0 days 00:04:02</td>
      <td>0 days 00:04:02</td>
      <td>Doesn't matter</td>
    </tr>
  </tbody>
</table>
</div>



Now let's group by True or False on this new column, and report the results!


```python
summary = table.groupby("better_scenario")["time_arriving_at_spadina"].count()

summary
```




    better_scenario
    Doesn't matter    912
    Stay on Train      68
    Name: time_arriving_at_spadina, dtype: int64



In essence, it's never going to hurt to stay on the train - it'll always be faster or the same time as getting off and walking. Let's see what percentage of the time it's worth it to stay on the train versus getting off and walking.


```python
( summary.sum() - summary["Doesn't matter"] ) / summary.sum()
```




    0.06938775510204082



Now let's figure out *how much* time you save -- in the cases when you do save time -- by staying on the train.


```python
table["time_savings"] = abs(table["travel_time_if_walking"] - table["travel_time_if_not_walking"])

table.head()
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
      <th>time_arriving_at_spadina</th>
      <th>travel_time_if_walking</th>
      <th>travel_time_if_not_walking</th>
      <th>better_scenario</th>
      <th>time_savings</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:00:35</td>
      <td>0 days 00:04:50</td>
      <td>0 days 00:04:50</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:03:18</td>
      <td>0 days 00:04:02</td>
      <td>0 days 00:04:02</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:03:18</td>
      <td>0 days 00:04:02</td>
      <td>0 days 00:04:02</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:06:56</td>
      <td>0 days 00:04:29</td>
      <td>0 days 00:04:29</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:09:19</td>
      <td>0 days 00:03:10</td>
      <td>0 days 00:03:10</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
    </tr>
  </tbody>
</table>
</div>



How much time do you save when staying on the train is the right choice?


```python
table.loc[table["better_scenario"] == "Walk", "time_savings"].mean()
```




    NaT



And what's the most amount of time you could save?


```python
table.loc[table["better_scenario"] == "Walk", "time_savings"].max()
```




    NaT




```python
table[["time_arriving_at_spadina", "time_savings"]].hist()
```




    array([[<AxesSubplot:title={'center':'time_arriving_at_spadina'}>]],
          dtype=object)




    
![png](output_61_1.png)
    


### A twist - delays

There's another element to this that we can add in: delays.

For the most part, train delays affect each of our scenarios equally. If a trains are delayed on the Bloor Line and 


```python
raw_delays = pd.read_excel("schedule_data/2022-delays.xlsx")

raw_delays
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
      <th>Date</th>
      <th>Time</th>
      <th>Day</th>
      <th>Station</th>
      <th>Code</th>
      <th>Min Delay</th>
      <th>Min Gap</th>
      <th>Bound</th>
      <th>Line</th>
      <th>Vehicle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-01-01</td>
      <td>15:59</td>
      <td>Saturday</td>
      <td>LAWRENCE EAST STATION</td>
      <td>SRDP</td>
      <td>0</td>
      <td>0</td>
      <td>N</td>
      <td>SRT</td>
      <td>3023</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-01-01</td>
      <td>02:23</td>
      <td>Saturday</td>
      <td>SPADINA BD STATION</td>
      <td>MUIS</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>BD</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-01-01</td>
      <td>22:00</td>
      <td>Saturday</td>
      <td>KENNEDY SRT STATION TO</td>
      <td>MRO</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>SRT</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-01-01</td>
      <td>02:28</td>
      <td>Saturday</td>
      <td>VAUGHAN MC STATION</td>
      <td>MUIS</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>YU</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-01-01</td>
      <td>02:34</td>
      <td>Saturday</td>
      <td>EGLINTON STATION</td>
      <td>MUATC</td>
      <td>0</td>
      <td>0</td>
      <td>S</td>
      <td>YU</td>
      <td>5981</td>
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
    </tr>
    <tr>
      <th>19890</th>
      <td>2022-12-31</td>
      <td>01:20</td>
      <td>Saturday</td>
      <td>BAY STATION</td>
      <td>SUUT</td>
      <td>16</td>
      <td>22</td>
      <td>E</td>
      <td>BD</td>
      <td>5089</td>
    </tr>
    <tr>
      <th>19891</th>
      <td>2022-12-31</td>
      <td>01:31</td>
      <td>Saturday</td>
      <td>SHEPPARD WEST STATION</td>
      <td>EUAC</td>
      <td>23</td>
      <td>29</td>
      <td>N</td>
      <td>YU</td>
      <td>5656</td>
    </tr>
    <tr>
      <th>19892</th>
      <td>2022-12-31</td>
      <td>01:33</td>
      <td>Saturday</td>
      <td>BAY STATION</td>
      <td>MUPAA</td>
      <td>0</td>
      <td>0</td>
      <td>E</td>
      <td>BD</td>
      <td>5313</td>
    </tr>
    <tr>
      <th>19893</th>
      <td>2022-12-31</td>
      <td>01:49</td>
      <td>Saturday</td>
      <td>YONGE BD STATION</td>
      <td>MUPAA</td>
      <td>3</td>
      <td>9</td>
      <td>E</td>
      <td>BD</td>
      <td>5211</td>
    </tr>
    <tr>
      <th>19894</th>
      <td>2022-12-31</td>
      <td>12:52</td>
      <td>Saturday</td>
      <td>DON MILLS STATION</td>
      <td>TUSC</td>
      <td>0</td>
      <td>0</td>
      <td>W</td>
      <td>SHP</td>
      <td>6166</td>
    </tr>
  </tbody>
</table>
<p>19895 rows × 10 columns</p>
</div>




```python
raw_delays[raw_delays["Station"].str.contains("GEORGE")]["Station"].unique()
```




    array(['ST GEORGE YUS STATION', 'ST GEORGE BD STATION',
           'ST. GEORGE STATION', 'ST GEORGE STATION TO B',
           'ST GEORGE TO BROADVIEW', 'BROADVIEW TO ST GEORGE',
           'CHRISTIE - ST GEORGE S', 'ST GEORGE STATION'], dtype=object)




```python
delays = raw_delays.copy()
george_delays = (delays[delays["Station"].isin(["ST GEORGE BD STATION",
                                                "ST GEORGE YUS STATION",
                                                "ST GEORGE STATION",
                                                "ST. GEORGE STATION",
                                                ])]
                 )

george_delays
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
      <th>Date</th>
      <th>Time</th>
      <th>Day</th>
      <th>Station</th>
      <th>Code</th>
      <th>Min Delay</th>
      <th>Min Gap</th>
      <th>Bound</th>
      <th>Line</th>
      <th>Vehicle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27</th>
      <td>2022-01-01</td>
      <td>11:17</td>
      <td>Saturday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>TUO</td>
      <td>5</td>
      <td>10</td>
      <td>S</td>
      <td>YU</td>
      <td>5936</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2022-01-01</td>
      <td>18:12</td>
      <td>Saturday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>MUSAN</td>
      <td>7</td>
      <td>12</td>
      <td>S</td>
      <td>YU</td>
      <td>5666</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2022-01-01</td>
      <td>23:45</td>
      <td>Saturday</td>
      <td>ST GEORGE BD STATION</td>
      <td>SUDP</td>
      <td>0</td>
      <td>0</td>
      <td>W</td>
      <td>BD</td>
      <td>5181</td>
    </tr>
    <tr>
      <th>126</th>
      <td>2022-01-03</td>
      <td>08:20</td>
      <td>Monday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>PUOPO</td>
      <td>3</td>
      <td>6</td>
      <td>N</td>
      <td>YU</td>
      <td>6011</td>
    </tr>
    <tr>
      <th>128</th>
      <td>2022-01-03</td>
      <td>08:46</td>
      <td>Monday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>PUOPO</td>
      <td>0</td>
      <td>0</td>
      <td>N</td>
      <td>YU</td>
      <td>5521</td>
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
    </tr>
    <tr>
      <th>19764</th>
      <td>2022-12-29</td>
      <td>17:57</td>
      <td>Thursday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>SUAP</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>YU</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19801</th>
      <td>2022-12-30</td>
      <td>13:21</td>
      <td>Friday</td>
      <td>ST GEORGE BD STATION</td>
      <td>SUDP</td>
      <td>6</td>
      <td>9</td>
      <td>E</td>
      <td>BD</td>
      <td>5233</td>
    </tr>
    <tr>
      <th>19848</th>
      <td>2022-12-31</td>
      <td>10:00</td>
      <td>Saturday</td>
      <td>ST GEORGE BD STATION</td>
      <td>EUBO</td>
      <td>5</td>
      <td>10</td>
      <td>W</td>
      <td>BD</td>
      <td>5182</td>
    </tr>
    <tr>
      <th>19864</th>
      <td>2022-12-31</td>
      <td>15:44</td>
      <td>Saturday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>MUIRS</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>YU</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19869</th>
      <td>2022-12-31</td>
      <td>19:14</td>
      <td>Saturday</td>
      <td>ST GEORGE YUS STATION</td>
      <td>MUPAA</td>
      <td>0</td>
      <td>0</td>
      <td>S</td>
      <td>YU</td>
      <td>6066</td>
    </tr>
  </tbody>
</table>
<p>726 rows × 10 columns</p>
</div>




```python
idx = pd.date_range('01-01-2022', '12-31-2022')

daily_delays_count = george_delays.groupby("Date")["Min Delay"].count().to_frame()
daily_delays_duration = george_delays.groupby("Date")["Min Delay"].sum().to_frame()

daily_delays_count = daily_delays_count.join(daily_delays_duration, rsuffix="(Min)")

daily_delays_count = daily_delays_count.reindex(idx, fill_value=0)

daily_delays_count
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
      <th>Min Delay</th>
      <th>Min Delay(Min)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2022-01-01</th>
      <td>3</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2022-01-02</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2022-01-03</th>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2022-01-04</th>
      <td>4</td>
      <td>7</td>
    </tr>
    <tr>
      <th>2022-01-05</th>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2022-12-27</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2022-12-28</th>
      <td>3</td>
      <td>21</td>
    </tr>
    <tr>
      <th>2022-12-29</th>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2022-12-30</th>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2022-12-31</th>
      <td>3</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
<p>365 rows × 2 columns</p>
</div>




```python
operational_minutes = (george_west["arrival_time"].max() - george_west["arrival_time"].min()).seconds / 60
```


```python
daily_delays_count["delay_chance"] = daily_delays_count["Min Delay(Min)"] / operational_minutes
```


```python
daily_delays_count.sort_values("delay_chance", ascending=False)
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
      <th>Min Delay</th>
      <th>Min Delay(Min)</th>
      <th>delay_chance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2022-11-15</th>
      <td>4</td>
      <td>141</td>
      <td>0.112127</td>
    </tr>
    <tr>
      <th>2022-04-02</th>
      <td>2</td>
      <td>120</td>
      <td>0.095427</td>
    </tr>
    <tr>
      <th>2022-04-19</th>
      <td>3</td>
      <td>50</td>
      <td>0.039761</td>
    </tr>
    <tr>
      <th>2022-12-09</th>
      <td>2</td>
      <td>44</td>
      <td>0.034990</td>
    </tr>
    <tr>
      <th>2022-07-01</th>
      <td>7</td>
      <td>42</td>
      <td>0.033400</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2022-03-22</th>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2022-09-30</th>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2022-05-24</th>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2022-08-07</th>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2022-06-25</th>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
<p>365 rows × 3 columns</p>
</div>




```python
chance_of_delay_each_minute = daily_delays_count["Min Delay(Min)"].sum() / (365 * operational_minutes)
```


```python
avg_delay_length = george_delays["Min Delay"].mean()
```


```python
expected_delay = chance_of_delay_each_minute * avg_delay_length

expected_delay
```




    0.02305945151011738




```python
table_with_delays = table.copy()

table_with_delays["delay_if_not_walking"] = table_with_delays["travel_time_if_not_walking"] * expected_delay
table_with_delays["delay_if_walking"] = table_with_delays["travel_time_if_walking"] * expected_delay

table_with_delays["travel_time_if_not_walking"] = table_with_delays["travel_time_if_not_walking"] + table_with_delays["delay_if_not_walking"]
table_with_delays["travel_time_if_walking"] = table_with_delays["travel_time_if_walking"] + table_with_delays["delay_if_walking"]

table_with_delays.loc[table_with_delays["travel_time_if_not_walking"] > table_with_delays["travel_time_if_walking"] , "should_walk_at_spadina?"] = True
table_with_delays.loc[table_with_delays["travel_time_if_not_walking"] <= table_with_delays["travel_time_if_walking"] , "should_walk_at_spadina?"] = False
table_with_delays["should_walk_at_spadina?"] = table_with_delays["should_walk_at_spadina?"]

table_with_delays
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
      <th>time_arriving_at_spadina</th>
      <th>travel_time_if_walking</th>
      <th>travel_time_if_not_walking</th>
      <th>better_scenario</th>
      <th>time_savings</th>
      <th>delay_if_not_walking</th>
      <th>delay_if_walking</th>
      <th>should_walk_at_spadina?</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:00:35</td>
      <td>0 days 00:04:56.687240937</td>
      <td>0 days 00:04:56.687240937</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:06.687240937</td>
      <td>0 days 00:00:06.687240937</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:03:18</td>
      <td>0 days 00:04:07.580387265</td>
      <td>0 days 00:04:07.580387265</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:05.580387265</td>
      <td>0 days 00:00:05.580387265</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:03:18</td>
      <td>0 days 00:04:07.580387265</td>
      <td>0 days 00:04:07.580387265</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:05.580387265</td>
      <td>0 days 00:00:05.580387265</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:06:56</td>
      <td>0 days 00:04:35.202992456</td>
      <td>0 days 00:04:35.202992456</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:06.202992456</td>
      <td>0 days 00:00:06.202992456</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-02 06:09:19</td>
      <td>0 days 00:03:14.381295786</td>
      <td>0 days 00:03:14.381295786</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:04.381295786</td>
      <td>0 days 00:00:04.381295786</td>
      <td>False</td>
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
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-03 02:32:46</td>
      <td>0 days 00:05:21.240667774</td>
      <td>0 days 00:05:21.240667774</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:07.240667774</td>
      <td>0 days 00:00:07.240667774</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-03 02:35:19</td>
      <td>0 days 00:05:12.033132710</td>
      <td>0 days 00:05:12.033132710</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:07.033132710</td>
      <td>0 days 00:00:07.033132710</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-03 02:35:19</td>
      <td>0 days 00:05:12.033132710</td>
      <td>0 days 00:05:12.033132710</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:07.033132710</td>
      <td>0 days 00:00:07.033132710</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-03 02:35:19</td>
      <td>0 days 00:05:12.033132710</td>
      <td>0 days 00:05:12.033132710</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:07.033132710</td>
      <td>0 days 00:00:07.033132710</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2022-03-03 02:36:45</td>
      <td>0 days 00:03:44.050019880</td>
      <td>0 days 00:03:44.050019880</td>
      <td>Doesn't matter</td>
      <td>0 days</td>
      <td>0 days 00:00:05.050019880</td>
      <td>0 days 00:00:05.050019880</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>980 rows × 8 columns</p>
</div>




```python
table_with_delays["time_savings_by_not_walking"] =  table_with_delays["travel_time_if_walking"] - table_with_delays["travel_time_if_not_walking"]
```


```python
table_with_delays["time_savings_by_not_walking"].mean(), table["time_savings_by_not_walking"].mean()
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    c:\Users\dexmc\anaconda3\lib\site-packages\pandas\core\indexes\base.py in get_loc(self, key, method, tolerance)
       3360             try:
    -> 3361                 return self._engine.get_loc(casted_key)
       3362             except KeyError as err:
    

    c:\Users\dexmc\anaconda3\lib\site-packages\pandas\_libs\index.pyx in pandas._libs.index.IndexEngine.get_loc()
    

    c:\Users\dexmc\anaconda3\lib\site-packages\pandas\_libs\index.pyx in pandas._libs.index.IndexEngine.get_loc()
    

    pandas\_libs\hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()
    

    pandas\_libs\hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()
    

    KeyError: 'time_savings_by_not_walking'

    
    The above exception was the direct cause of the following exception:
    

    KeyError                                  Traceback (most recent call last)

    ~\AppData\Local\Temp/ipykernel_3884/3718692207.py in <module>
    ----> 1 table_with_delays["time_savings_by_not_walking"].mean(), table["time_savings_by_not_walking"].mean()
    

    c:\Users\dexmc\anaconda3\lib\site-packages\pandas\core\frame.py in __getitem__(self, key)
       3456             if self.columns.nlevels > 1:
       3457                 return self._getitem_multilevel(key)
    -> 3458             indexer = self.columns.get_loc(key)
       3459             if is_integer(indexer):
       3460                 indexer = [indexer]
    

    c:\Users\dexmc\anaconda3\lib\site-packages\pandas\core\indexes\base.py in get_loc(self, key, method, tolerance)
       3361                 return self._engine.get_loc(casted_key)
       3362             except KeyError as err:
    -> 3363                 raise KeyError(key) from err
       3364 
       3365         if is_scalar(key) and isna(key) and not self.hasnans:
    

    KeyError: 'time_savings_by_not_walking'



```python
table_with_delays.groupby("should_walk_at_spadina?")["time_savings_by_not_walking"].count()
```




    should_walk_at_spadina?
    False    980
    Name: time_savings_by_not_walking, dtype: int64




```python
hist = table_with_delays[["time_arriving_at_spadina", "time_savings_by_not_walking"]].set_index("time_arriving_at_spadina").resample("h").mean()

hist["time_savings_by_not_walking"] = hist["time_savings_by_not_walking"].dt.seconds

hist.to_clipboard()
```
