# python-api-challenge
# Vacation and Weather Analysis

This project consists of two Python scripts, `Weather.py` and `Vacation.py`, designed to analyze weather data and vacation destinations. The scripts work together to filter and visualize cities based on weather conditions and find nearby hotels.

## Overview

### `Weather.py`

- **Purpose**: Fetches weather data for a list of cities and creates scatter plots to visualize the relationship between weather variables (temperature, humidity, cloudiness, and wind speed) and latitude. Computes linear regression to analyze these relationships.

- **Dependencies**:
  - `matplotlib` for plotting
  - `pandas` for data manipulation
  - `numpy` for numerical operations
  - `requests` for making API requests
  - `scipy` for linear regression
  - `citipy` for identifying cities based on latitude and longitude

- **Setup**:
  1. **API Key**: Ensure you have an OpenWeatherMap API key. Store it in a file named `api_keys.py` as `weather_api_key`.
  2. **Dependencies**: Install the required libraries using pip:
     ```bash
     pip install matplotlib pandas numpy requests scipy citipy
     ```
  3. **Data Generation**: The script generates random latitude and longitude coordinates to identify unique cities using the `citipy` library.

- **Script Flow**:
  1. **City Generation**: Generates a list of unique cities from random geographic coordinates.
  2. **Data Retrieval**: Fetches weather data from OpenWeatherMap for the generated cities.
  3. **Data Storage**: Saves the collected weather data into a CSV file.
  4. **Visualization**: Creates scatter plots and performs linear regression to analyze relationships between weather variables and latitude.

### `Vacation.py`

- **Purpose**: Filters cities based on ideal weather conditions, finds nearby hotels using the Geoapify API, and visualizes the results on an interactive map.

- **Dependencies**:
  - `pandas` for data manipulation
  - `requests` for making API requests
  - `hvplot` for interactive plotting

- **Setup**:
  1. **API Key**: Obtain a Geoapify API key and replace `"YOUR_GEOAPIFY_API_KEY"` in the script.
  2. **Dependencies**: Install the required libraries using pip:
     ```bash
     pip install pandas requests hvplot
     ```

- **Script Flow**:
  1. **Data Preparation**: Loads the filtered city data from a CSV file.
  2. **Hotel Search**: Uses the Geoapify API to find nearby hotels for each city.
  3. **Visualization**: Creates an interactive map displaying the cities and their nearest hotels.

## Example Code

### `Weather.py`

```python
# Import required libraries
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import time
from scipy.stats import linregress
from api_keys import weather_api_key
from citipy import citipy

# Generate list of cities
lat_lngs = [(lat, lng) for lat, lng in zip(np.random.uniform(-90, 90, size=1500), np.random.uniform(-180, 180, size=1500))]
cities = list(set(citipy.nearest_city(lat, lng).city_name for lat, lng in lat_lngs))

# Fetch weather data
city_data = []
for city in cities:
    city_url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={weather_api_key}&units=metric"
    try:
        city_weather = requests.get(city_url).json()
        city_data.append({
            "City": city,
            "Lat": city_weather["coord"]["lat"],
            "Lng": city_weather["coord"]["lon"],
            "Max Temp": city_weather["main"]["temp_max"],
            "Humidity": city_weather["main"]["humidity"],
            "Cloudiness": city_weather["clouds"]["all"],
            "Wind Speed": city_weather["wind"]["speed"],
            "Country": city_weather["sys"]["country"],
            "Date": city_weather["dt"]
        })
    except:
        pass
    time.sleep(1)

# Save and read data
city_data_df = pd.DataFrame(city_data)
city_data_df.to_csv("cities.csv", index_label="City_ID")
city_data_df = pd.read_csv("cities.csv", index_col="City_ID")

# Create scatter plots and linear regression
def plot_linear_regression(x_values, y_values, x_label, y_label, title, line_pos):
    slope, intercept, r_value, _, _ = linregress(x_values, y_values)
    plt.figure(figsize=(10, 6))
    plt.scatter(x_values, y_values, edgecolor="black", linewidths=1, marker="o", alpha=0.75)
    plt.plot(x_values, slope * x_values + intercept, color="red")
    plt.annotate(f"y = {slope:.2f}x + {intercept:.2f}\nR² = {r_value**2:.2f}", line_pos, fontsize=12, color="red")
    plt.title(title)
    plt.xlabel(x_label)
    plt.ylabel(y_label)
    plt.show()
    print(f"Equation of the line: y = {slope:.2f}x + {intercept:.2f}")
    print(f"The R² value is: {r_value**2:.2f}")

# Scatter plots and linear regression by hemisphere
northern_hemi_df = city_data_df[city_data_df["Lat"] >= 0]
southern_hemi_df = city_data_df[city_data_df["Lat"] < 0]
plot_linear_regression(northern_hemi_df["Lat"], northern_hemi_df["Max Temp"], "Latitude", "Max Temperature (°C)", "Northern Hemisphere Latitude vs. Max Temp", (10, -20))
plot_linear_regression(southern_hemi_df["Lat"], southern_hemi_df["Max Temp"], "Latitude", "Max Temperature (°C)", "Southern Hemisphere Latitude vs. Max Temp", (10, -20))


## Example Code

### `Vacation.py`
```python
import pandas as pd
import requests
import hvplot.pandas

# Load filtered city data
city_data_df = pd.read_csv('filtered_city_data.csv')

# Create hotel DataFrame
hotel_df = city_data_df.copy()
hotel_df['Hotel Name'] = ""

# Geoapify API parameters
api_key = "YOUR_GEOAPIFY_API_KEY"
radius = 10000
params = {
    "categories": "accommodation.hotel",
    "limit": 1,
    "apiKey": api_key
}

# Find hotels
for index, row in hotel_df.iterrows():
    lat = row["Lat"]
    lng = row["Lng"]
    params["filter"] = f"circle:{lng},{lat},{radius}"
    params["bias"] = f"proximity:{lng},{lat}"
    base_url = "https://api.geoapify.com/v2/places"
    response = requests.get(base_url, params=params)
    name_address = response.json()
    try:
        hotel_df.loc[index, "Hotel Name"] = name_address["features"][0]["properties"]["name"]
    except (KeyError, IndexError):
        hotel_df.loc[index, "Hotel Name"] = "No hotel found"

# Plot data
map_plot = hotel_df.hvplot.scatter(
    x="Lng",
    y="Lat",
    size="Humidity",
    color="Humidity",
    hover_cols=["City", "Country", "Hotel Name"],
    title="Cities and Nearby Hotels",
    xlabel="Longitude",
    ylabel="Latitude"
)

map_plot

## Troubleshooting
	•	API Errors: Ensure that the API keys are valid and correctly inserted into the scripts.
	•	Data Issues: Verify that the data files are correctly generated and paths are correctly set.

## Acknowledgements
This project was developed with the assistance of ChatGPT, a conversational AI developed by OpenAI. ChatGPT provided guidance and support in creating and structuring the code, as well as generating documentation. 