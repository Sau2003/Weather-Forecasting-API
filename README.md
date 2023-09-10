from tkinter import *
import requests
import matplotlib.pyplot as plt
from datetime import datetime

# Your conversion functions and get_weather function remain unchanged
api_key = "d6473b61f30e673d8da312056a6d00d6"
def kelvin_to_celsius(kelvin):
    return kelvin - 273.15

def kelvin_to_fahrenheit(kelvin):
    return (kelvin - 273.15) * 9/5 + 32

def get_weather(api_key, location):
    try:
        base_url = "http://api.openweathermap.org/data/2.5/weather"

        params = {
            "q": location,
            "appid": api_key,
        }

        response = requests.get(base_url, params=params)
        data = response.json()

        tfield.delete("1.0", "end")

        if data.get("cod") == 200:
            weather_desc = data["weather"][0]["description"] if "weather" in data else "Unknown"
            temp_k = data["main"]["temp"] if "main" in data else "Unknown"

            temp_c = kelvin_to_celsius(temp_k)
            temp_f = kelvin_to_fahrenheit(temp_k)

            tfield.insert("insert", f"Weather in {location}:\n")
            tfield.insert("insert", f"{weather_desc} and Temperature:\n")
            tfield.insert("insert", f"{temp_c:.2f}°C / {temp_f:.2f}°F\n")

        else:
            tfield.insert("insert", f"Error: Unable to fetch weather data for {location}\n")

    except requests.exceptions.RequestException:
        tfield.insert("insert", "Error: NO INTERNET, Kindly check your internet connection\n")

def plot_temperature_graph(dates, avg_temps):
    formatted_dates = [date.strftime('%Y-%m-%d') for date in dates]
    plt.figure(figsize=(10, 6))
    plt.plot(formatted_dates, avg_temps, marker='o')
    plt.title(f"6-Day Temperature Forecast of {user_location.get()}")
    plt.xlabel("Date")
    plt.ylabel("Average Temperature (°C)")
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()
# ... (previous code)

def get_6_day_forecast(api_key, location):
    try:
        base_url = "http://api.openweathermap.org/data/2.5/forecast"

        params = {
            "q": location,
            "appid": api_key,
        }

        response = requests.get(base_url, params=params)
        data = response.json()

        if data.get("cod") == "200":
            forecasts = data.get("list", [])
            dates = [datetime.utcfromtimestamp(forecast["dt"]) for forecast in forecasts]
            avg_temps = [kelvin_to_celsius(forecast["main"]["temp"]) for forecast in forecasts]
            return dates, avg_temps

        else:
            tfield.delete("1.0", "end")
            tfield.insert("insert", f"Error: Unable to fetch 6-day forecast for {location}\n")

    except requests.exceptions.RequestException:
        tfield.delete("1.0", "end")
        tfield.insert("insert", "Error: NO INTERNET, Kindly check your internet connection\n")

# Modify the get_and_plot_forecast function

def get_and_plot_forecast():
    location = user_location.get()
    weather_data = get_weather(api_key, location)
    
    if weather_data is not None:
        dates, avg_temps = weather_data
        plot_temperature_graph(dates, avg_temps)

root = Tk()
root.geometry("400x600")
root.resizable(0, 0)
root.title("Weather Forecast Application")

user_location = StringVar()

Label(root, text="Enter City Name", font='Arial 12 bold').pack(pady=10)
Entry(root, textvariable=user_location, width=24, font='Arial 14 bold').pack()

api_key = "d6473b61f30e673d8da312056a6d00d6"

Button(root, text="Check Weather", font="Roman", bg='black', fg='white', activebackground="red",
       command=lambda: get_weather(api_key, user_location.get())).pack(pady=15)

Button(root, text="Next 6 Days", font="Arial 10", bg='lightblue', fg='black', activebackground="green",
       command=get_and_plot_forecast).place(x=50, y=500)

weather_now = Label(root, text="The Weather is:", font='arial 12 bold')
weather_now.pack(pady=10)

tfield = Text(root, width=46, height=10)
tfield.pack()

root.mainloop()




