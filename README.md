# weather.app"""
Simple Weather App using the OpenWeatherMap API.

Setup:
1. Sign up for a free API key at https://openweathermap.org/api
2. Replace API_KEY below with your key (or set it as an environment
   variable called OPENWEATHER_API_KEY).
3. Install the requests library if you don't have it:
       pip install requests
4. Run:
       python weather_app.py
"""

import os
import sys
import requests

# You can hardcode your key here, or set the OPENWEATHER_API_KEY env variable.
API_KEY = os.environ.get("OPENWEATHER_API_KEY", "YOUR_API_KEY_HERE")
BASE_URL = "https://api.openweathermap.org/data/2.5/weather"


def get_weather(city: str, units: str = "metric") -> dict:
    """
    Fetch current weather data for a given city.

    :param city: Name of the city, e.g. "London" or "London,GB"
    :param units: "metric" (Celsius), "imperial" (Fahrenheit), or "standard" (Kelvin)
    :return: Parsed JSON response as a dict
    :raises: requests.HTTPError if the request fails
    """
    params = {
        "q": city,
        "appid": API_KEY,
        "units": units,
    }
    response = requests.get(BASE_URL, params=params, timeout=10)
    response.raise_for_status()
    return response.json()


def display_weather(data: dict, units: str = "metric") -> None:
    """Pretty-print the weather data returned by the API."""
    unit_symbol = "°C" if units == "metric" else "°F" if units == "imperial" else "K"
    speed_unit = "m/s" if units != "imperial" else "mph"

    city = data.get("name", "Unknown")
    country = data.get("sys", {}).get("country", "")
    weather_desc = data["weather"][0]["description"].capitalize()
    temp = data["main"]["temp"]
    feels_like = data["main"]["feels_like"]
    temp_min = data["main"]["temp_min"]
    temp_max = data["main"]["temp_max"]
    humidity = data["main"]["humidity"]
    pressure = data["main"]["pressure"]
    wind_speed = data["wind"]["speed"]

    print("\n" + "=" * 40)
    print(f"Weather in {city}, {country}")
    print("=" * 40)
    print(f"Condition:    {weather_desc}")
    print(f"Temperature:  {temp}{unit_symbol} (feels like {feels_like}{unit_symbol})")
    print(f"Min / Max:    {temp_min}{unit_symbol} / {temp_max}{unit_symbol}")
    print(f"Humidity:     {humidity}%")
    print(f"Pressure:     {pressure} hPa")
    print(f"Wind speed:   {wind_speed} {speed_unit}")
    print("=" * 40 + "\n")


def main():
    if API_KEY == "YOUR_API_KEY_HERE":
        print("⚠️  Please set your OpenWeatherMap API key before running this script.")
        print("   Either edit the API_KEY variable in this file, or set the")
        print("   OPENWEATHER_API_KEY environment variable.")
        sys.exit(1)

    print("=== Simple Weather App ===")
    print("(Type 'quit' or 'exit' to stop)\n")

    # Let the user choose units once at startup
    unit_choice = input("Units - (m)etric °C, (i)mperial °F, or (k)elvin? [m]: ").strip().lower()
    units_map = {"m": "metric", "i": "imperial", "k": "standard", "": "metric"}
    units = units_map.get(unit_choice, "metric")

    while True:
        city = input("Enter city name: ").strip()
        if city.lower() in ("quit", "exit"):
            print("Goodbye!")
            break
        if not city:
            print("Please enter a valid city name.\n")
            continue

        try:
            data = get_weather(city, units)
            display_weather(data, units)
        except requests.exceptions.HTTPError as e:
            if e.response is not None and e.response.status_code == 404:
                print(f"City '{city}' not found. Please check the spelling.\n")
            elif e.response is not None and e.response.status_code == 401:
                print("Invalid API key. Please check your API_KEY setting.\n")
            else:
                print(f"HTTP error occurred: {e}\n")
        except requests.exceptions.RequestException as e:
            print(f"Network error occurred: {e}\n")


if __name__ == "__main__":
    main()