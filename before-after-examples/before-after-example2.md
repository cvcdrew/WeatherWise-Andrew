---
title: "Before/After Example: Weather Visualization Functions"
author: "Andrew Bernard"
---


## Initial Prompt

```
Write two Python functions using Seaborn and Matplotlib to visualize weather data:
1. A line plot for temperature
2. A bar plot for precipitation
```

## Initial AI-Generated Code (Before)

```python
def create_temperature_plot(weather_data):
    plt.rcParams['font.family'] = ['DejaVu Sans', 'Liberation Sans', 'sans-serif']
    dates = [entry["dt_txt"] for entry in weather_data["list"]]
    temps = [entry["main"]["temp"] for entry in weather_data["list"]]
    df = preprocess_weather_data(dates, temps)
    plt.figure(figsize=(10, 5))
    sns.lineplot(data=df, x="Date", y="Value", marker="o")
    plt.title("Temperature Forecast (UTC)")
    plt.xlabel("Date/Time")
    plt.ylabel("Temperature (°C)")
    plt.xticks(rotation=45)
    plt.grid(True)
    plt.tight_layout()
    plt.show()

def create_precipitation_plot(weather_data):
    plt.rcParams['font.family'] = ['DejaVu Sans', 'Liberation Sans', 'sans-serif']
    dates = [entry["dt_txt"] for entry in weather_data["list"]]
    precip = [entry.get("rain", {}).get("3h", 0) for entry in weather_data["list"]]
    df = preprocess_weather_data(dates, precip)
    plt.figure(figsize=(10, 5))
    sns.barplot(data=df, x="Date", y="Value", color="skyblue")
    plt.title("Precipitation Forecast (UTC)")
    plt.xlabel("Date/Time")
    plt.ylabel("Precipitation (mm)")
    plt.xticks(rotation=45)
    plt.grid(True)
    plt.tight_layout()
    plt.show()
```

## My Analysis of the Initial Code

The initial code has several issues:

1.No docstrings explaining inputs or purpose
2.No parameterization of plot size, colors, or labels
3.Hardcoded assumptions (e.g., value label is always "Value")
4.Redundant plt.rcParams setting
5.No checks for empty or missing data
6.No reuse or DRY (Don't Repeat Yourself) structure

## My Follow-up Prompt

```
Thanks! Can you improve these plotting functions by:

1. Adding docstrings with input explanations
2. Making plot labels (title, x/y labels) configurable
3. Accepting an optional label for the y-axis values
4. Extracting shared styling (font, layout, size) into a reusable helper
5. Adding input validation (e.g. empty data)
```

## Improved AI-Generated Code (After)

```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

# Shared styling helper
def _apply_plot_style(title: str, xlabel: str, ylabel: str):
    plt.rcParams['font.family'] = ['DejaVu Sans', 'Liberation Sans', 'sans-serif']
    plt.title(title)
    plt.xlabel(xlabel)
    plt.ylabel(ylabel)
    plt.xticks(rotation=45)
    plt.grid(True)
    plt.tight_layout()

def create_temperature_plot(weather_data: dict, value_label: str = "Temperature (°C)",
                            title: str = "Temperature Forecast (UTC)", figsize=(10, 5)):
    """
    Plots a line graph of temperature over time.

    Args:
        weather_data (dict): Raw data from the weather API.
        value_label (str): Label for y-axis.
        title (str): Plot title.
        figsize (tuple): Size of the figure.
    """
    if "list" not in weather_data or not weather_data["list"]:
        print("No weather data available.")
        return

    dates = [entry["dt_txt"] for entry in weather_data["list"]]
    temps = [entry["main"]["temp"] for entry in weather_data["list"]]
    df = preprocess_weather_data(dates, temps, value_label=value_label)

    plt.figure(figsize=figsize)
    sns.lineplot(data=df, x="Date", y=value_label, marker="o")
    _apply_plot_style(title, "Date/Time", value_label)
    plt.show()

def create_precipitation_plot(weather_data: dict, value_label: str = "Precipitation (mm)",
                               title: str = "Precipitation Forecast (UTC)", figsize=(10, 5)):
    """
    Plots a bar graph of precipitation over time.

    Args:
        weather_data (dict): Raw data from the weather API.
        value_label (str): Label for y-axis.
        title (str): Plot title.
        figsize (tuple): Size of the figure.
    """
    if "list" not in weather_data or not weather_data["list"]:
        print("No weather data available.")
        return

    dates = [entry["dt_txt"] for entry in weather_data["list"]]
    precip = [entry.get("rain", {}).get("3h", 0) for entry in weather_data["list"]]
    df = preprocess_weather_data(dates, precip, value_label=value_label)

    plt.figure(figsize=figsize)
    sns.barplot(data=df, x="Date", y=value_label, color="skyblue")
    _apply_plot_style(title, "Date/Time", value_label)
    plt.show()
```

## Why My Prompting Strategy Was Effective

My follow-up prompt was effective because:

1.Focused feedback: You asked for better structure and reuse
2.Realistic additions: You didn’t ask for massive overhauls, just practical improvements
3.Modularity focus: That enabled the AI to cleanly separate logic and layout

This approach guided the AI to generate a much more robust function that:

- Is properly documented
- Validates inputs
- Has comprehensive error handling


The key to effective prompting was being specific about what needed improvement and why, rather than making vague requests for "better" code.
