from tkinter import *
import requests
import matplotlib.pyplot as plt
from datetime import datetime

class WeatherApp:
    def __init__(self, root):
        self.root = root
        self.root.geometry("400x600")
        self.root.resizable(0, 0)
        self.root.title("Weather Forecast Application")

        self.api_key = "d6473b61f30e673d8da312056a6d00d6"

        self.user_location = StringVar()

        Label(self.root, text="Enter City Name", font='Arial 12 bold').pack(pady=10)
        Entry(self.root, textvariable=self.user_location, width=24, font='Arial 14 bold').pack()

        Button(self.root, text="Check Weather", font="Roman", bg='black', fg='white', activebackground="red",
               command=self.get_weather).pack(pady=15)

        Button(self.root, text="Next 6 Days", font="Arial 10", bg='lightblue', fg='black', activebackground="green",
               command=self.get_and_plot_forecast).place(x=50, y=500)

        self.weather_now = Label(self.root, text="The Weather is:", font='arial 12 bold')
        self.weather_now.pack(pady=10)

        self.tfield = Text(self.root, width=46, height=10)
        self.tfield.pack()

    def kelvin_to_celsius(self, kelvin):
        return kelvin - 273.15

    def kelvin_to_fahrenheit(self, kelvin):
        return (kelvin - 273.15) * 9/5 + 32

    def get_weather(self):
        try:
            base_url = "http://api.openweathermap.org/data/2.5/weather"

            params = {
                "q": self.user_location.get(),
                "appid": self.api_key,
            }

            response = requests.get(base_url, params=params)
            data = response.json()

            self.tfield.delete("1.0", "end")

            if data.get("cod") == 200:
                weather_desc = data["weather"][0]["description"] if "weather" in data else "Unknown"
                temp_k = data["main"]["temp"] if "main" in data else "Unknown"

                temp_c = self.kelvin_to_celsius(temp_k)
                temp_f = self.kelvin_to_fahrenheit(temp_k)

                self.tfield.insert("insert", f"Weather in {self.user_location.get()}:\n")
                self.tfield.insert("insert", f"{weather_desc} and Temperature:\n")
                self.tfield.insert("insert", f"{temp_c:.2f}°C / {temp_f:.2f}°F\n")

            else:
                self.tfield.insert("insert", f"Error: Unable to fetch weather data for {self.user_location.get()}\n")

        except requests.exceptions.RequestException:
            self.tfield.insert("insert", "Error: NO INTERNET, Kindly check your internet connection\n")

    def plot_temperature_graph(self, dates, avg_temps):
        formatted_dates = [date.strftime('%Y-%m-%d') for date in dates]
        plt.figure(figsize=(10, 6))
        plt.plot(formatted_dates, avg_temps, marker='o')
        plt.title(f"6-Day Temperature Forecast of {self.user_location.get()}")
        plt.xlabel("Date")
        plt.ylabel("Average Temperature (°C)")
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.show()

    def get_6_day_forecast(self):
        try:
            base_url = "http://api.openweathermap.org/data/2.5/forecast"

            params = {
                "q": self.user_location.get(),
                "appid": self.api_key,
            }

            response = requests.get(base_url, params=params)
            data = response.json()

            if data.get("cod") == "200":
                forecasts = data.get("list", [])
                dates = [datetime.utcfromtimestamp(forecast["dt"]) for forecast in forecasts]
                avg_temps = [self.kelvin_to_celsius(forecast["main"]["temp"]) for forecast in forecasts]
                return dates, avg_temps

            else:
                self.tfield.delete("1.0", "end")
                self.tfield.insert("insert", f"Error: Unable to fetch 6-day forecast for {self.user_location.get()}\n")

        except requests.exceptions.RequestException:
            self.tfield.delete("1.0", "end")
            self.tfield.insert("insert", "Error: NO INTERNET, Kindly check your internet connection\n")

    def get_and_plot_forecast(self):
        weather_data = self.get_6_day_forecast()

        if weather_data is not None:
            dates, avg_temps = weather_data
            self.plot_temperature_graph(dates, avg_temps)


if __name__ == "__main__":
    root = Tk()
    app = WeatherApp(root)
    root.mainloop()
