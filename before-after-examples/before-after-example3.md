---
title: "Before/After Example: Interactive Weather Forecast Widgets"
author: "Andrew Bernard"
---


## Initial Prompt

```
Create an interactive user interface with IPython widgets that:
- Lets users input a city, number of forecast days, and unit type
- Lets users toggle between temperature, precipitation, or both
- Provides a field to ask an AI a weather-related question
- Shows corresponding forecast visualizations
```

## Initial AI-Generated Code (Before)

```python
# UI widgets
location_box = widgets.Text(description="City:", placeholder="e.g. London", value="London")
days_slider = widgets.IntSlider(description="Days:", min=1, max=5, value=3)
units_dropdown = widgets.Dropdown(options=["metric", "imperial", "standard"], value="metric", description="Units:")
vis_toggle = widgets.ToggleButtons(options=["Temperature", "Precipitation", "Both"], description="Visual:")
question_box = widgets.Text(description="Ask AI:", placeholder="e.g. Will it rain in Paris tomorrow?")
go_button = widgets.Button(description="Show Forecast", button_style="primary")
ask_ai_button = widgets.Button(description="Ask AI", button_style="info")
output = widgets.Output()

# Forecast button handler
def on_click(b):
    with output:
        clear_output()
        try:
            if question_box.value.strip():
                parsed = parse_weather_question(question_box.value)
                print(generate_weather_response(parsed))
                location = parsed["city"]
                day = parsed["day"]
                days = 1 if day == "today" else (2 if day == "tomorrow" else 3)
                vis = "Temperature"
                if parsed["condition"] in ["rain", "snow"]:
                    vis = "Both"
                question_box.value = ""
            else:
                location = location_box.value.strip() or "London"
                days = days_slider.value
                vis = vis_toggle.value
            units = units_dropdown.value
            weather_data = get_weather_data(location, days, units)
            if vis in ["Temperature", "Both"]:
                create_temperature_plot(weather_data)
            if vis in ["Precipitation", "Both"]:
                create_precipitation_plot(weather_data)
        except Exception as e:
            print(f"❌ Error: {e}")

go_button.on_click(on_click)

# Ask AI handler
def on_ask_ai_click(b):
    with output:
        clear_output()
        question = question_box.value.strip()
        if not question:
            print("⚠️ Please enter a question in the 'Ask AI' box.")
            return
        try:
            parsed = parse_weather_question(question)
            print(generate_weather_response(parsed))
            question_box.value = ""
        except Exception as e:
            print(f"❌ Error processing AI question: {e}")

ask_ai_button.on_click(on_ask_ai_click)

# Display layout
print("☁️ Welcome to the AI-Powered Weather Assistant!")
display(widgets.VBox([
    widgets.HBox([question_box, ask_ai_button]),
    widgets.HBox([location_box, days_slider]),
    widgets.HBox([units_dropdown, vis_toggle]),
    go_button,
    output
]))
```

## My Analysis of the Initial Code

The initial code has several issues:

1.No modular functions — all logic is inside handlers
2.No docstrings or comments for functions
3.Some logic is duplicated (e.g. parsing vs input handling)
4.No input validation abstraction
5.Harder to test or reuse components separately
6.No separation of UI construction vs event logic

## My Follow-up Prompt

```
Thanks! Can you refactor this into a modular, documented version with:
- Reusable functions for UI creation and layout
- Input validation logic separated
- Separate functions for forecast vs AI handling
- Clear docstrings and layout explanation
```

## Improved AI-Generated Code (After)

```python
from IPython.display import display, clear_output
import ipywidgets as widgets

# --- UI Creation Functions ---

def create_weather_widgets():
    """Creates and returns all widgets used in the weather assistant UI."""
    return {
        "location_box": widgets.Text(description="City:", placeholder="e.g. London", value="London"),
        "days_slider": widgets.IntSlider(description="Days:", min=1, max=5, value=3),
        "units_dropdown": widgets.Dropdown(options=["metric", "imperial", "standard"], value="metric", description="Units:"),
        "vis_toggle": widgets.ToggleButtons(options=["Temperature", "Precipitation", "Both"], description="Visual:"),
        "question_box": widgets.Text(description="Ask AI:", placeholder="e.g. Will it rain in Paris tomorrow?"),
        "go_button": widgets.Button(description="Show Forecast", button_style="primary"),
        "ask_ai_button": widgets.Button(description="Ask AI", button_style="info"),
        "output": widgets.Output()
    }

# --- Logic Functions ---

def handle_forecast(ui):
    """Handles 'Show Forecast' button logic."""
    with ui["output"]:
        clear_output()
        try:
            if ui["question_box"].value.strip():
                parsed = parse_weather_question(ui["question_box"].value)
                print(generate_weather_response(parsed))
                location = parsed["city"]
                day = parsed["day"]
                days = 1 if day == "today" else (2 if day == "tomorrow" else 3)
                vis = "Both" if parsed["condition"] in ["rain", "snow"] else "Temperature"
                ui["question_box"].value = ""
            else:
                location = ui["location_box"].value.strip() or "London"
                days = ui["days_slider"].value
                vis = ui["vis_toggle"].value

            units = ui["units_dropdown"].value
            weather_data = get_weather_data(location, days, units)

            if vis in ["Temperature", "Both"]:
                create_temperature_plot(weather_data)
            if vis in ["Precipitation", "Both"]:
                create_precipitation_plot(weather_data)

        except Exception as e:
            print(f"❌ Error: {e}")

def handle_ai_question(ui):
    """Handles 'Ask AI' button logic."""
    with ui["output"]:
        clear_output()
        question = ui["question_box"].value.strip()
        if not question:
            print("⚠️ Please enter a question in the 'Ask AI' box.")
            return
        try:
            parsed = parse_weather_question(question)
            print(generate_weather_response(parsed))
            ui["question_box"].value = ""
        except Exception as e:
            print(f"❌ Error processing AI question: {e}")

# --- Display Function ---

def display_weather_ui():
    """Displays the complete weather assistant UI."""
    ui = create_weather_widgets()

    # Wire up event handlers
    ui["go_button"].on_click(lambda b: handle_forecast(ui))
    ui["ask_ai_button"].on_click(lambda b: handle_ai_question(ui))

    # Layout the widgets
    layout = widgets.VBox([
        widgets.HBox([ui["question_box"], ui["ask_ai_button"]]),
        widgets.HBox([ui["location_box"], ui["days_slider"]]),
        widgets.HBox([ui["units_dropdown"], ui["vis_toggle"]]),
        ui["go_button"],
        ui["output"]
    ])

    print("☁️ Welcome to the AI-Powered Weather Assistant!")
    display(layout)

# Launch the UI
display_weather_ui()
```

## Why My Prompting Strategy Was Effective

My follow-up prompt was effective because:

1.Modularity: Clear boundaries between UI, logic, and event handling
2.Reusability: Easier to test components individually
3.Clean layout: More maintainable and extensible
4.Better UX: Resets fields and guides user if inputs are missing

This approach guided the AI to generate a much more robust function that:

- Is properly documented
- Validates inputs


The key to effective prompting was being specific about what needed improvement and why, rather than making vague requests for "better" code.
