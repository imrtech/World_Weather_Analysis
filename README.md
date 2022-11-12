# World_Weather_Analysis

## Project Overview

In this project we are testing new capabilitiles for the PlanMyTrip app. This information will be used to identify potential travel destinations and nearby hotels.

We will perform the following:
 - Add the weather description to the weather data and use OpenWeatherMap's API to retreive the latitude and longtitude, temperatures, and other weather conditions such as humidity, cloudiness, and windspeed.
 - Filter data by creating user inputs for weather preferences. 
 - Create a travel itinerary
 - We will use Google Maps Directions API to create a travel route between the four cities and a marker layer map.
 - We will use gmaps documentation extensively.

## Results

## Exploratory Analysis

###Deliverable 1: 

We performed analysis on weather data. We created a python file and imported these libraries:

```
# Import initial libraries
import numpy as np
import pandas as pd
from citipy import citipy
```

We used a np.random.uniform function to generate a new set of 2,000 random latitudes and 2,000 longitudes.

```
# Create a set of random latitude and longitude combinations
lats = np.random.uniform(low=-90.000, high=90.000, size=2000)
lngs = np.random.uniform(low=-180.000, high=180.000, size=2000)

# Use the zip function to create an iterator of tuples containing the latitude and longitude combinations
lat_lngs = zip(lats, lngs)
```
We added the results to a list.

```
# Add the latitudes and longitudes to a list
coordinates = list(lat_lngs)
```

We used a for loop to identify the nearest city to the current coordinates. The result was 790.

```
# Create an empty list for holding the cities
cities = []

# Use a for loop to identify nearest city for each latitude and longitude combination using the citipy module
for coordinate in coordinates:
    # Use the citipy module to identify the nearest city to the current coordinate
    city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name
    
    # If the city is unique, then add it to a our cities list
    if city not in cities:
        cities.append(city)

# Print the city count to confirm sufficient count
len(cities)
```

We imported OpenWeatherMap's API key and assembled the API call.
```
# Import the requests library
import requests

# Import the time library
import time

# Import the datetime module from the datetime library
from datetime import datetime

# Import the OpenWeatherMap's API key
from config import weather_api_key

# Assemble the OpenWeatherMap's API call
url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=" + weather_api_key
```

We created an empty list to hold the weather data for each city:
```
# Create an empty list to hold weather data for each city
city_data = []

# Print a message to indicate that the data retrieval starts
print("Beginning Data Retrieval     ")
print("-----------------------------")

# Create counters and set them to 1
record_count = 1
set_count = 1

# Loop through all the cities in our list to fetch weather data for each city
for i, city in enumerate(cities):
        
    # Group cities in sets of 50 for logging purposes
    if (i % 50 == 0 and i >= 50):
        set_count += 1
        record_count = 1
        time.sleep(60)

    # Create an endpoint URL for each city
    city_url = url + "&q=" + city.replace(" ","+")
    
    # Log the url, record, and set numbers
    print(f"Processing Record {record_count} of Set {set_count} | {city}")

    # Add 1 to the record count
    record_count += 1

    # Run an API request for each of the cities
    try:
        city_weather = requests.get(city_url).json()
        # Parse out the latitude, longitude, max temp, humidity, cloudiness, wind, country, and weather description
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]        
       # city_date = datetime.utcfromtimestamp(city_weather["dt"]).strftime('%Y-%m-%d %H:%M:%S')
        city_weather_description = city_weather["weather"][0]["description"]
        
        # Append the city information into the city_data list
        city_data.append({"City": city.title(),
                          "Lat": city_lat,
                          "Lng": city_lng,
                          "Max Temp": city_max_temp,
                          "Humidity": city_humidity,
                          "Cloudiness": city_clouds,
                          "Wind Speed": city_wind,
                          "Country": city_country,
                          "Current Description": city_weather_description})
                          #"Date": city_date})
    
    # If an error is experienced, skip the city
    except:
        print("City not found. Skipping...")
        pass

# Indicate that the data retrieval is complete 
print("-----------------------------")
print("Data Retrieval Complete      ")
print("-----------------------------")
```


```
# Print the length of the city_data list to verify how many cities you have
len(city_data)
```
Our city data resulted in 729 cities.

We then added the weather data to a new DataFrame
```

# Use the city_data list to create a new pandas DataFrame.
city_data_df = pd.DataFrame(city_data)

# Display sample data
city_data_df.head(10)
```
![This is an image](/Weather_Database/City_data.png)


```
# Display the DataFrame's column names using the columns Pandas function
city_data_df.columns

Index(['City', 'Lat', 'Lng', 'Max Temp', 'Humidity', 'Cloudiness',
       'Wind Speed', 'Country', 'Current Description'],
      dtype='object')
```

We created a list to reorder the column names an recreated the DataFrame using a new column order.

```# Create a list to reorder the column names as follows:
new_column_order = ["City", "Country", "Lat", "Lng", "Max Temp", "Humidity",  "Cloudiness", "Wind Speed",  "Current Description"]

# Recreate the DataFrame by using the new column order
city_data_df = city_data_df[new_column_order]

# Display sample data
city_data_df.head(10)
```

![This is an image](/Weather_Database/City_data_new.png)

```
# Display the data types of each column by using the dtypes Pandas function
city_data_df.dtypes
```

Finally, we exported the DataFrame as a .csv file.

```
# Set the output file name
output_data_file = "WeatherPy_Database.csv"

# Export the city_data DataFrame into a CSV file
city_data_df.to_csv(output_data_file, index_label="City_ID")
```

### Deliverable 2: 

For this deliverable we created a destination map. 

We imported the following libraries:

```
# Dependencies and Setup
import pandas as pd
import requests
import gmaps
import matplotlib.pyplot as plt
import numpy as np

# Import Google API key
from config import g_key

# Configure gmaps
gmaps.configure(api_key=g_key)
```

We created a new Pandas DataFrame using the .csv file we created from deliverable 1. 

```
# Set the file path to import the WeatherPy_database.csv file
file_path = "Weather_Database/WeatherPy_database.csv"

# Load the CSV file into a Pandas DataFrame
city_data_df = pd.read_csv("../Weather_Database/WeatherPy_database.csv")

# Display sample data
city_data_df.head()
```

![This is an image](/Vacation_Search/Pandas_DataFrame.png)


We created two input statements that would prompt the user to enter a desired minimum temperature and a maximum temperature.

```
# Prompt the user to enter the minimum temperature criteria
min_temp = float(input("What is the minimum temperature you would like for your trip? "))

# Prompt the user to enter the maximum temperature criteria
max_temp = float(input("What is the maximum temperature you would like for your trip? "))

What is the minimum temperature you would like for your trip? 60
What is the maximum temperature you would like for your trip? 88
```

Next, we created a new Pandas DataFrame that would filter the city data for the temperature criteria collected.
```
# Filter the city_data_df DataFrame to find the cities that fit the criteria using the loc Pandas function
preferred_cities_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & \
                                       (city_data_df["Max Temp"] >= min_temp)]
                                    
# Display sample data
preferred_cities_df.head(10)
```

![This is an image](/Vacation_Search/Temp_criteria.png)

We cleaned some of the data by runnin the dropna() function to remove any empty rows.

```
# Drop any empty rows in the preferred_cities_df DataFrame and create a new DataFrame.
clean_travel_cities_df = preferred_cities_df.dropna()

# Display sample data
clean_travel_cities_df
```

![This is an image](/Vacation_Search/clean_travel.png)

We created a new DataFrame copying specific columns from the clean_travel_cities DataFrame.

```
# Create DataFrame called hotel_df by copying some columns from the clean_travel_cities DataFrame.
hotel_df = clean_travel_cities_df[["City", "Country", "Max Temp", "Current Description", "Lat", "Lng"]].copy()

# Display sample data
hotel_df.head(10)
```

![This is an image](/Vacation_Search/hotel_df.png)

We added Hotel Name to the hotel_df DataFrame.

```
# Add a new empty column, "Hotel Name", to the hotel_df DataFrame
hotel_df["Hotel Name"] = ""

# Display sample data
hotel_df.head(10)
```

To populate the hotel column we used parameters to search for a hotel and ran a for loop to retreive the latitude and longitude of each city to find the nearest hotel based on the search parameters. If a hotel was not found, we would skip to the next city.

```
# Review the parameters to search for a hotel
params = {
    "radius": 5000,
    "type": "lodging",
    "key": g_key
} 


# Iterate through the hotel DataFrame 
for index, row in hotel_df.iterrows():
    # Fetch latitude and longitude from the DataFrame
    lat = row["Lat"]
    lng = row["Lng"]
    
    # Add the latitude and longitude as parameters to the params dictionary
    params["location"] = f"{lat},{lng}"
    
    # Set up the base URL for the Google Directions API to get JSON data
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"

    # Make an API request and retrieve the JSON data from the hotel search
    hotels = requests.get(base_url, params=params).json()
    
    # Get the first hotel from the results and store the name, if a hotel isn't found skip the city
    try:
        hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
    except (IndexError):
        print("Hotel not found... skipping.")    



# Display sample data
hotel_df.head(10)
   
```
![This is an image](/Vacation_Search/refined_hotel_df.png)

We dropped rows that did not contain a hotel name.

```
# Drop the rows where there is no Hotel Name.
clean_hotel_df = hotel_df.replace(r'^\s*$', np.nan, regex=True)
clean_hotel_df = clean_hotel_df.dropna()

# Display sample data
clean_hotel_df
```

We created a new .csv to store the clean_hotel_df DataFrame.

```
# Set the file name.
output_data_file = "WeatherPy_vacation.csv"

# Create a CSV file by using the clean_hotel_df DataFrame
clean_hotel_df.to_csv(output_data_file, index_label="City_ID")
```

We created an info_box_template to collect and present Hotel Name, City, Country, Current Weather and Max Temp.

```
# Review the formatting template provided
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Current Weather</dt><dd>{Current Description} and {Max Temp} °F</dd>
</dl>
"""
```
```
# Get the data from each row in the clean_hotel_df DataFrame, add it to the formatting template, and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in clean_hotel_df.iterrows()]

# Display sample data
hotel_info[:10]
```

We then retreived the city data from each row and added it to the exiting hotel_info list.

```
# Get the data from each row in the clean_hotel_df DataFrame, add it to the formatting template, and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in clean_hotel_df.iterrows()]

# Display sample data
hotel_info[:10]
```

We retreived the latitude and longitude from each row and created a the locations DataFrame.

```
# Get the latitude and longitude from each row and store in a new DataFrame.
locations = clean_hotel_df[["Lat", "Lng"]]

# Display sample data
locations.head(10)
```

![This is an image](/Vacation_Search/locations.png)


Finally, we refactored the previous marker layer map to include pop-up markers for each city on the map.

```
# Add a marker layer for each city to the map. 
fig = gmaps.figure()
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)

# Create a figure to add the Google map as a layer
fig.add_layer(marker_layer)

# Display the figure containing the map
fig
```
![This is an image](/Vacation_Search/WeatherPy_vacation_map.png)


### Deliverable 3: 

To help us map our travel route we selected four cities in Colombia, South America, El Retorno, Puerto Colombia, Dibulla, and Riohacha. We retrieved the latitude and longitude for each city and created a directions layer map to include waypoints between the cities and our mode of travel as "Driving." Finally, we created a marker layer for the four cities with pop-up markers that included the info_box_template from deliverable #2--Hotel name, the city, the country, and current weather description and the maximum temperature. 

We imported our dependencies.

```
# Dependencies and Setup
import pandas as pd
import requests
import gmaps
import matplotlib.pyplot as plt
import numpy as np

# Import API key
from config import g_key


# Configure gmaps
gmaps.configure(api_key=g_key)

```
We used the WeatherPY_vacation.csv from deliverable 2.

```
# Read the WeatherPy_vacation.csv into a DataFrame
vacation_df = pd.read_csv("../Vacation_Search/WeatherPy_vacation.csv")

# Display sample data
vacation_df.head()
```

![This is an image](/Vacation_Intinerary/vacation_df.png)

We set up the pop-up markers and created a marker layer map on the vacation search results. This used the same code as Deliverable 2.

```
# Review the formatting template
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Current Weather</dt><dd>{Current Description} and {Max Temp} °F</dd>
</dl>
"""

# Get the data from each row and add it to the formatting template and store the data in a list
hotel_info = [info_box_template.format(**row) for index, row in vacation_df.iterrows()]

# Get the latitude and longitude from each row and store in a new DataFrame.
locations = vacation_df[["Lat", "Lng"]]


# Add a marker layer for each city to the map.
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
fig = gmaps.figure()
fig.add_layer(marker_layer)

# Display the figure
fig

```

We choose four cities from the same country.

```
# Create DataFrames for each city by filtering the 'vacation_df' using the loc method. 
# The starting and ending city should be the same city.

vacation_start = vacation_df.loc[vacation_df['City'] == 'El Retorno']
vacation_end = vacation_df.loc[vacation_df['City'] == 'El Retorno']
vacation_stop1 = vacation_df.loc[vacation_df['City'] == 'Puerto Colombia']
vacation_stop2 = vacation_df.loc[vacation_df['City'] == 'Dibulla']
vacation_stop3 = vacation_df.loc[vacation_df['City'] == 'Riohacha']
```
```
# Get the latitude-longitude pairs as tuples from each city DataFrame using the to_numpy function and list indexing.
start = vacation_start[["Lat", "Lng"]].to_numpy()[0]
end = vacation_end[["Lat", "Lng"]].to_numpy()[0]
stop1 = vacation_stop1[["Lat", "Lng"]].to_numpy()[0]
stop2 = vacation_stop2[["Lat", "Lng"]].to_numpy()[0]
stop3 = vacation_stop3[["Lat", "Lng"]].to_numpy()[0]
```

We created a directions layer map using the filtered 'vacation_df in the previous step.

```
# Define a new figure object
fig = gmaps.figure()

# Create a direction layer map using the start and end latitude-longitude pairs, and stop1, stop2, and stop3 as the waypoints.
# The travel_mode should be "DRIVING", "BICYCLING", or "WALKING".
vacation_itinerary = gmaps.directions_layer(
                    start, end, waypoints = [stop1, stop2, stop3], 
                    travel_mode="DRIVING")

# Add the layer to the map
fig.add_layer(vacation_itinerary)

# Display the map
fig
```

![This is an image](/Vacation_Intinerary/WeatherPy_travel_map.png)

To create an itinerary, we used the concat() function code snippet to combine the four cities in one DataFrame.

```
#  Combine the four city DataFrames into one DataFrame using the concat() function.
itinerary_df = pd.concat(
    [
        vacation_start,
        vacation_stop1,
        vacation_stop2,
        vacation_stop3
    ],
    ignore_index = True
)

# Display sample data
itinerary_df
```


We refactored the code to create a new marker layer map of the cities on the travel route.
![This is an image](/Vacation_Itinerary/WeatherPy_travel_map_markers.png)



## Summary

From what we have gathered, we can see that in rural areas, ride-sharing was low and only made up $4,327.93 in total fares. This could be attributed to the low number of drivers available or the high fares. We would need to analyze this further to see if there are any correlations and look at other factors that may contribute to low ridership, such as trip distance or other commutting options. 

Total fares amoung the three city types was $63,538.64. With urban areas generating the highest total fares at $39,854.38. An interesting discovery is that suburban drivers accounted for roughly 30% of the total fares, even though they only made up about 12.5% of all drivers. Additional analysis would be required to understand why this is the case. 

In general we see that that ridership is high in urban areas, where there are more drivers, and the fares are low. Analyzing distances traveled and the presence of other commutting options or lack thereof in those areas would be an interesting approach. 

- The code file can be found here:  [filename](/PyBer_Challenge.ipynb)
