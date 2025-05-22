---
title: "Before/After Example: Weather Data Fetch Function "
author: "Andrew Bernard"
---


## Initial Prompt

```
Write a Python function that fetches weather forecast data from the OpenWeatherMap API for a given city, using parameters like number of forecast days and units (metric, imperial). Also create a helper function to preprocess the data for plotting.
```

## Initial AI-Generated Code (Before)

```python
def get_weather_data(location: str, forecast_days: int = 5, units: str = "metric") -> dict:
    params = {
        "q": location,
        "cnt": forecast_days * INTERVALS_PER_DAY,
        "units": units,
        "appid": API_KEY
    }
    res = requests.get(BASE_URL, params=params)
    if res.status_code != 200:
        raise Exception(f"Error fetching weather: {res.status_code} - {res.text}")
    return res.json()

# Helper function to preprocess weather data for plotting
def preprocess_weather_data(dates, values):
    # Create a pandas DataFrame with datetime and corresponding values (temperature or precipitation)
    df = pd.DataFrame({"Date": pd.to_datetime(dates), "Value": values})
    return df
```

## My Analysis of the Initial Code

The initial code has several issues:

1.No error handling for invalid locations or bad API keys
2.No timeout parameter—this could cause the program to hang
3.No docstrings or explanations
4.Assumes global variables like API_KEY, BASE_URL, and INTERVALS_PER_DAY exist
5.Returns full raw JSON, which might be too detailed for most use cases
6.Preprocessing function lacks flexibility (e.g., no label support or type validation)

## My Follow-up Prompt

```
Thanks! Can you improve the get_weather_data function by:

1. Adding error handling for invalid city names, timeouts, and bad API keys
2. Including a timeout parameter in the request
3. Accepting BASE_URL and API_KEY as arguments (for modular use)
4. Returning a more compact structure with just the needed fields
5. Adding docstrings with parameter and return explanations

And for preprocess_weather_data:

- Add optional label for the value column
- Validate that the input lists are the same length
- Include error handling and helpful error messages
```

## Improved AI-Generated Code (After)

```python
import requests
import pandas as pd

def get_weather_data(location: str, forecast_days: int = 5, units: str = "metric",
                     api_key: str = "", base_url: str = "https://api.openweathermap.org/data/2.5/forecast",
                     timeout: int = 10) -> dict:
    """
    Fetches weather forecast data from the OpenWeatherMap API.

    Args:
        location (str): City name
        forecast_days (int): Number of days to forecast (default is 5)
        units (str): Units of measurement - 'metric', 'imperial', or 'standard'
        api_key (str): Your OpenWeatherMap API key
        base_url (str): API endpoint URL
        timeout (int): Timeout for the request in seconds

    Returns:
        dict: A structured dictionary containing city info and list of forecast entries
    """
    if not api_key:
        raise ValueError("API key is required.")

    intervals_per_day = 8  # 3-hour intervals
    params = {
        "q": location,
        "cnt": forecast_days * intervals_per_day,
        "units": units,
        "appid": api_key
    }

    try:
        response = requests.get(base_url, params=params, timeout=timeout)
        response.raise_for_status()
        data = response.json()

        # Simplify and return relevant data
        return {
            "city": data.get("city", {}).get("name", "Unknown"),
            "country": data.get("city", {}).get("country", ""),
            "forecasts": data["list"]
        }

    except requests.exceptions.RequestException as e:
        raise Exception(f"Error fetching weather data: {e}")
    except (KeyError, TypeError, ValueError) as e:
        raise Exception(f"Unexpected data format: {e}")

def preprocess_weather_data(dates, values, value_label="Value"):
    """
    Converts raw date/value lists into a clean DataFrame for plotting.

    Args:
        dates (list): List of datetime strings
        values (list): Corresponding values (e.g., temperature or precipitation)
        value_label (str): Optional label for the 'Value' column

    Returns:
        pd.DataFrame: A DataFrame with columns 'Date' and custom 'Value'
    """
    if len(dates) != len(values):
        raise ValueError("Dates and values must be of the same length.")

    df = pd.DataFrame({
        "Date": pd.to_datetime(dates),
        value_label: values
    })
    return df
```

## Why My Prompting Strategy Was Effective

My follow-up prompt was effective because:

1. **it is Specific and actionable: I listed concrete flaws and exact enhancements (timeout, validation, modularity).
2. **User-centered: I requested outputs that would be easier for plotting or display.
3. **Realistic expectations: I didn’t ask for too much in one go—just enough for a solid improvement.

This approach guided the AI to generate a much more robust function that:

- Is properly documented
- Validates inputs
- Has comprehensive error handling


The key to effective prompting was being specific about what needed improvement and why, rather than making vague requests for "better" code.
