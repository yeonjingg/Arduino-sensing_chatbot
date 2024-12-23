# Arduino-sensing_chatbot
# The primary objective of this project is..
 The primary objective of this project is to develop a chatbot that supports indoor environmental improvement. By using Arduino and various sensors to measure indoor air quality, and integrating outdoor air quality data through an OpenAPI, we aim to enable users to easily monitor both indoor and outdoor air conditions and receive personalized recommendations on optimal ventilation and other relevant actions.

# 📡 Sensor Overview  
## 🛠️ Using Sensors  

### 1. DHT11  
- **Description**: A digital sensor that measures temperature and humidity.  
- **Application**: Used for monitoring environmental conditions in indoor and outdoor settings.  
![image](https://github.com/user-attachments/assets/668a0fb8-b3ab-4e01-a72b-5e1f18a3520d)

---

### 2. PP-A422  
- **Description**: A particulate matter (PM) sensor used for detecting fine dust levels in air quality monitoring systems.  
- **Application**: Effective in air quality projects to monitor PM2.5 and PM10 levels.  
![image](https://github.com/user-attachments/assets/dbdc05e7-31f3-4f96-93f3-8e31fbd90402)

---

### 3. MQ-7  
- **Description**: A gas sensor designed to detect carbon monoxide (CO).  
- **Application**: Often used in gas leak detection, fire alarm systems, and air quality control.  
![image](https://github.com/user-attachments/assets/c22d7ba1-a9cf-458b-a830-707df7dd4056)

---

## 🛠️ 사용 센서 및 코드  

---

### 1. Temperature and Humidity – DHT11  

```cpp
#include <DHT11.h>  // Include the DHT11 library.

int pin = 2;  
DHT11 dht11(pin);  // Create a DHT11 object.

void setup()
{
  Serial.begin(9600);  // Set the serial communication speed to 9600 bps.
}

void loop()
{
  int err;          // Variable to store error codes.
  float temp, humi; // Variables to store temperature and humidity values.

  // Read data from the DHT11 sensor and check if successful.
  if ((err = dht11.read(humi, temp)) == 0)
  {
    // If the sensor data is read successfully.
    Serial.print("temperature:"); 
    Serial.print(temp);         
    Serial.print(" humidity:");  
    Serial.print(humi);         
    Serial.println();            
  }
  else
  {
    // If the sensor data could not be read, print the error code.
    Serial.println();
    Serial.print("Error No :");  
    Serial.print(err);           
    Serial.println();            
  }

  delay(DHT11_RETRY_DELAY); // Delay before the next read (1000 ms).
}
```
---

### 2. fine dust - PP-A422

```cpp
int pin = 8;                  
unsigned long duration;       
unsigned long starttime;     
unsigned long sampletime_ms = 1000;  
unsigned long lowpulseoccupancy = 0; 
float ratio = 0;             
float concentration = 0;     
float concentrations[30];    
int count = 0;                
bool finished = false;       

void setup() {
    Serial.begin(9600);       // Initialize serial communication
    pinMode(pin, INPUT);      
    starttime = millis();     
    Serial.println("Starting PM2.5 measurement...");
}

void loop() {
    if (finished) {
        return;  // Exit the loop if data collection and output are complete
    }

    duration = pulseIn(pin, LOW);       
    lowpulseoccupancy += duration;      // Accumulate the LOW signal duration

    if ((millis() - starttime) > sampletime_ms) {  // When the sampling period has elapsed
        ratio = lowpulseoccupancy / (sampletime_ms * 10.0);  
        concentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62; 

        concentrations[count] = concentration;    // Store the data in the array
        count++;

        if (count == 30) {  
            Serial.println("Data collected:");
            float sum = 0; 
            for (int i = 0; i < 30; i++) {
                Serial.print(concentrations[i]);
                if (i < 29) Serial.print(",");  
                sum += concentrations[i];
            }
            Serial.println();  

            // Calculate and print the average concentration
            float average = (sum / 30.0)/300;
            Serial.print("Average PM2.5 Concentration: ");
            Serial.print(average);
            Serial.println(" µg/m³");

            finished = true;  // Set the flag to exit the loop
        }

        // Reset variables for the next sampling period
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}
```
---

### 3. Carbon Monoxide(CO) - MQ-7

```cpp
int GasPin = A0;  
float sensorVoltage = 5.0;  
float RL = 10.0;  

void setup() {
  pinMode(GasPin, INPUT);  // Set analog pin A0 to input mode
  Serial.begin(9600);      
}

void loop() {
  int sensorValue = analogRead(GasPin);  // Read the analog value from the MQ-7 sensor

  // Convert the analog value to voltage
  float voltage = sensorValue * (sensorVoltage / 1023.0);

  // Calculate CO concentration (ppm)
  float ppm = calculatePPM(voltage);

  // Print sensor data to the serial monitor
  Serial.print("Analog Value: ");
  Serial.print(sensorValue);
  Serial.print("\tVoltage: ");
  Serial.print(voltage, 3);  // Display voltage with 3 decimal places
  Serial.print(" V\tCO Concentration: ");
  Serial.print(ppm, 3);  // Display ppm with 3 decimal places
  Serial.println(" ppm");

  delay(5000);  // Wait for 5 seconds before the next reading
}

float calculatePPM(float voltage) {
  // R0 is the sensor's resistance in clean air (calibration needed)
  float R0 = 2.0;  // Adjusted default value from 10.0 to 2.0 for better sensitivity

  // Calculate RS (sensor resistance)
  float RS = (sensorVoltage - voltage) * RL / voltage;
  Serial.print(RS);

  // Calculate the RS/R0 ratio (use this for sensor's data sheet curve)
  float ratio = RS / R0;
  Serial.print(ratio);

  // Convert the ratio to ppm using a logarithmic formula (based on data sheet graph)
  // Example formula: ppm = 10^((log10(ratio) - b) / m)
  float b = -0.3;  // Intercept adjusted for low-concentration sensitivity
  float m = -0.17;  // Slope adjusted based on data sheet characteristics
  float ppm = pow(10, (log10(ratio) - b) / m);
  Serial.print(ppm);

  return ppm;
}
```
---

### 4. Final sensor function

Final sensor function

```cpp
// If 30 data points are collected, calculate and display averages
if (count == 30) {
    Serial.println("Data collected:");

    // Calculate and print average CO concentration
    float coSum = 0;
    for (int i = 0; i < 30; i++) {
        coSum += coConcentrations[i];
    }
    float coAverage = coSum / 30.0;
    Serial.print("Average CO Concentration: ");
    Serial.print(coAverage, 2);  // Print to 2 decimal places
    Serial.println(" ppm");

    // Calculate and print average PM2.5 concentration
    float pmSum = 0;
    for (int i = 0; i < 30; i++) {
        pmSum += pmConcentrations[i];
    }
    float pmAverage = pmSum / 30.0;
    Serial.print("Average PM2.5 Concentration: ");
    Serial.print(pmAverage, 2);
    Serial.println(" µg/m³");

    // Calculate and print average temperature
    float tempSum = 0;
    for (int i = 0; i < 30; i++) {
        tempSum += temperatures[i];
    }
    float tempAverage = tempSum / 30.0;
    Serial.print("Average Temperature: ");
    Serial.print(tempAverage, 2);
    Serial.println(" °C");

    // Calculate and print average humidity
    float humiSum = 0;
    for (int i = 0; i < 30; i++) {
        humiSum += humidities[i];
    }
    float humiAverage = humiSum / 30.0;
    Serial.print("Average Humidity: ");
    Serial.print(humiAverage, 2);
    Serial.println(" %");

    // Mark data collection as complete
    finished = true;
    Serial.println("Measurement complete. Ending program.");
}
```
---

- **Description**: The result of arduino
  
![image](https://github.com/user-attachments/assets/d10cc943-27f0-4dec-b218-60772eb08e88)


---

### 5. Final sensor code

```cpp
import serial
import time

def read_arduino_averages_as_string():
    """
    Function to read average values from Arduino and return them as a string.
    Only processes lines containing the word "Average".

    Args:
        port (str): The port where Arduino is connected (e.g., "COM3" or "/dev/ttyUSB0").
        baud_rate (int): Serial communication speed (default: 9600).
        timeout (int): Timeout for the serial port (in seconds).
    
    Returns:
        str: A single string containing averages for CO, PM2.5, temperature, and humidity.
    """
    # Connect to the Arduino serial port
    port = "/dev/ttyACM0"
    baud_rate = 9600
    timeout = 2
    ser = serial.Serial(port, baud_rate, timeout=timeout)
    time.sleep(2)  # Wait for Arduino initialization

    print("Waiting for average data from Arduino...")

    averages = {}  # Dictionary to store average values

    try:
        while True:
            if ser.in_waiting > 0:  # Check if data is available to read
                line = ser.readline().decode('utf-8').strip()  # Read and decode data
                
                # Process only lines containing the word "Average"
                if "Average" in line:
                    print(f"Filtered Data: {line}")  # Print lines containing "Average"
                    
                    if "CO Concentration" in line:
                        averages["CO (ppm)"] = float(line.split(":")[1].split()[0])
                    elif "PM2.5 Concentration" in line:
                        averages["PM2.5 (µg/m³)"] = float(line.split(":")[1].split()[0])
                    elif "Temperature" in line:
                        averages["Temperature (°C)"] = float(line.split(":")[1].split()[0])
                    elif "Humidity" in line:
                        averages["Humidity (%)"] = float(line.split(":")[1].split()[0])
                    
                    # Return the formatted string when all values are collected
                    if len(averages) == 4:
                        return (
                            f"CO: {averages['CO (ppm)']:.2f} ppm, "
                            f"PM2.5: {averages['PM2.5 (µg/m³)']:.2f} µg/m³, "
                            f"Temperature: {averages['Temperature (°C)']:.2f} °C, "
                            f"Humidity: {averages['Humidity (%)']:.2f} %"
                        )
    except KeyboardInterrupt:
        print("Interrupted!")
        return None
    finally:
        ser.close()  # Close the serial port


# Read the average values
average_string = read_arduino_averages_as_string()

if average_string:
    print("Average Sensor Data (String):")
    print(average_string)

```
---

- **Description**: Final circuit diagram

![image](https://github.com/user-attachments/assets/71eacf57-1ec8-483f-8eac-3b9e3e1c3a95)

![image](https://github.com/user-attachments/assets/97d98a7b-d646-4362-8917-78bf307e4720)


---

## 🛠️ 실외 대기질 OpenAPI 

---

```cpp
import urllib.request
import json
from pandas import json_normalize
from urllib.parse import urlencode, quote_plus, unquote
import time
import math

def get_air_quality_data(station_name, data_term='3MONTH', num_of_rows=2200):
    # API endpoint for fetching air quality data
    api = 'http://apis.data.go.kr/B552584/ArpltnInforInqireSvc/getMsrstnAcctoRltmMesureDnsty'
    # API key (decoded for use in the query parameters)
    key = unquote('--')

    # Build the query parameters for the API request
    queryParams = '?' + urlencode({
        quote_plus('serviceKey'): key,  
        quote_plus('returnType'): 'json',  
        quote_plus('numOfRows'): str(num_of_rows),  
        quote_plus('pageNo'): '1',  
        quote_plus('stationName'): station_name,  
        quote_plus('dataTerm'): data_term,  
        quote_plus('ver'): '1.0'  
    })

    # Combine the API endpoint with the query parameters
    url = api + queryParams

    try:
        # Send a request to the API and decode the response
        response = urllib.request.urlopen(url).read().decode('utf-8')
        json_return = json.loads(response)  # Parse the JSON response
        # Extract the data from the JSON response
        data = json_return.get('response', {}).get('body', {}).get('items', [])
        if data:
            # Normalize the JSON data into a pandas DataFrame
            return json_normalize(data)
        else:
            # Print a message if no data is found
            print("No data found for the given station.")
            return None
    except Exception as e:
        # Handle and print any exceptions that occur during the API call
        print(f"Failed to fetch air quality data: {e}")
        return None

if __name__ == "__main__":
    # Define the name of the station to fetch data for
    station_name = '중구'
    # Call the function to fetch air quality data
    air_quality_df = get_air_quality_data(station_name)
    if air_quality_df is not None:
        # Print the first few rows of the DataFrame if data is retrieved
        print("Air Quality Data:")
        print(air_quality_df.head())

use_functions = [
    {
        "type": "function",  # Indicates that this entry describes a function
        "function": {
            "name": "get_stations_by_region",  # Name of the function
            "description": (
                "Retrieve a list of monitoring stations in a given region by filtering a DataFrame. "
                "The function looks for the specified region name in the '지역명' column and returns "
                "the corresponding station names from the '측정소명' column."
            ),
            "parameters": {  
                "region_name": {
                    "type": "string",  
                    "description": (
                        "The name of the region for which to retrieve monitoring stations."
                    )
                }
            },
            "returns": {  
                "type": "list",  # Type of the return value
                "description": (
                    "A list of station names in the specified region. "
                    "Returns an empty list if the region does not exist or an error occurs."
                )
            }
        }
    }
]

```

- **Description**: Use Example Results

  ![image](https://github.com/user-attachments/assets/09568bd2-7b0f-4847-8a1a-52c62a85cf62)


---

## 🛠️ 날씨 함수로부터 온습도 추출하는 함수

---

```cpp

import requests
import json

def get_current_weather(city):
    """
    Fetches the current weather for a given city using the OpenWeatherMap API.

    Args:
        city (str): The name of the city to fetch the weather for.

    Returns:
        str: A string describing the weather conditions, temperature, and humidity in the specified city.
             Returns an error message if the city is not found.
    """
    key = "---"  
    api = f"http://api.openweathermap.org/geo/1.0/direct?q={city}&limit=5&appid={key}"
    location_response = requests.get(api)  # Send a request to the geolocation API
    location = json.loads(location_response.text)  # Parse the JSON response

    # Check if the location data is empty
    if not location:
        return f"Error: No location found for city '{city}'"

    # Extract latitude and longitude from the location data
    lat = location[0]['lat']
    lon = location[0]['lon']

    # API endpoint to get weather data for the given latitude and longitude
    location_api = f"https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={key}"
    result_response = requests.get(location_api)  
    result = json.loads(result_response.text)  

    # Extract weather information
    weather = result['weather'][0]['main']  # Main weather condition
    temp = result['main']['temp']  # Temperature in Kelvin
    temp = round(temp - 273.15, 2)  # Convert temperature to Celsius and round to 2 decimal places
    hum = result['main']['humidity']  # Humidity percentage

    # Return the formatted weather data
    return f"City: {city}, Weather: {weather}, Temperature: {temp}°C, Humidity: {hum}%"

# Example usage
city_name = "대전"  # City name in Korean (Daejeon)
result = get_current_weather(city_name)  # Fetch the weather data
print(result)

```

- **Description**: Use Example Results

![image](https://github.com/user-attachments/assets/71509921-dc6b-49f6-9ffb-cec1c1590e6a)


```cpp
{
  "type": "function",
  "function": {
    "name": "get_current_weather",
    "description": "Fetch current weather information for a given city, including temperature, humidity, and general weather conditions. This city corresponds to the region name provided in the get_stations_by_region function.",
    "parameters": {
      "type": "object",
      "properties": {
        "city": {
          "type": "string",
          "description": "The name of the city for which to fetch weather information. This city corresponds to the region name in get_stations_by_region."
        }
      },
      "required": ["city"]
    },
    "returns": {
      "type": "object",
      "properties": {
        "city": {
          "type": "string",
          "description": "The name of the city."
        },
        "weather": {
          "type": "string",
          "description": "The general weather condition (e.g., Clear, Rain, Clouds)."
        },
        "temperature": {
          "type": "number",
          "description": "The temperature in Celsius."
        },
        "humidity": {
          "type": "number",
          "description": "The percentage of humidity."
        }
      },
      "description": "Current weather information for the specified city."
    }
  }
}
```
---

## 🛠️ 장소명 불러오는 함수

---

```cpp

import pandas as pd

# File path for the Excel file
file_path = '/content/station_list.xls'

# Load the Excel file into a DataFrame
df = pd.read_excel(file_path)

# Function to retrieve station names based on the specified region name
def get_stations_by_region(region_name):
    try:
        # Columns for filtering (retain the Korean column names)
        region_column = '지역명'  # Column containing region names
        station_column = '측정소명'  # Column containing station names

        # Filter the DataFrame for the given region name and extract station names
        filtered_stations = df[df[region_column] == region_name][station_column].tolist()
        return filtered_stations
    except KeyError as e:
        # Handle errors when specified columns are not found in the DataFrame
        print(f"Error: Column not found - {e}")
        return []

# Example usage
region = "대전"  
result = get_stations_by_region(region) 
print(result)  

```

- **Description**: Use Example Results

![image](https://github.com/user-attachments/assets/e6ac108b-223d-49e7-b454-544963d0e730)


- **Description**: Excel file
![image](https://github.com/user-attachments/assets/b50877c0-59fd-4efd-ba58-c51bb2994d3c)

```cpp
{
  "type": "function",
  "function": {
    "name": "get_stations_by_region",  // Function name
    "description": "Retrieve a list of monitoring stations in a specified region using a DataFrame.",
    "parameters": {
      "type": "object",  // Parameters should be an object
      "properties": {
        "region_name": {
          "type": "string",  // The region name is a string
          "description": "The name of the region for which to retrieve monitoring stations."
        }
      },
      "required": ["region_name"]  // 'region_name' is a required parameter
    },
    "returns": {
      "type": "string",  // The function returns a string
      "description": "A string containing the region name followed by a list of station names. Returns a message if the region does not exist or an error occurs."
    }
  }
}
```

---

## 🛠️ ChatGPTs 역할 부여

---

```cpp

import gradio as gr
import random
import os
from openai import OpenAI

# Initial system message for the AI assistant
messages = [
    {
        "role": "system",
        "content": (
            "You are an assistant specialized in analyzing indoor and outdoor air quality, "
            "including temperature and humidity, to recommend whether to ventilate by opening windows. "
            "Consider factors such as particulate matter, gas pollutants, temperature comfort levels, "
            "and relative humidity for comprehensive recommendations."
        )
    }
]

# Function to process user input and interact with OpenAI
def process(user_message, chat_history):
    # Calls the OpenAI API with the specified model and input
    proc_messages, ai_message = ask_openai("gpt-4o-mini", messages, user_message, functions=use_functions)
    # Appends user and AI messages to the chat history
    chat_history.append((user_message, ai_message))
    return "", chat_history

# Gradio interface
with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="채팅창")  # Chatbot interface
    user_textbox = gr.Textbox(label="입력")  # Textbox for user input
    # Submits user input and updates chat history
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])
    demo.launch(share=True, debug=True)  # Launch Gradio app with sharing and debug enabled

```

---

## 🛠️ 활용 방안

---

### 1. Vehicle Ventilation System
CO 100-200ppm causes minor headaches, fatigue,
CO 400 ppm or more can lead to drowsiness, dizziness, and, in severe cases, loss of consciousness
It is a system that automatically opens the window to ventilate when it exceeds this number

---

### 2. Hospital ventilation system
The hospital must have a relative humidity of 40 to 60%, the concentration of fine dust is 35 µg/m ³ or less, CO₂ shall be maintained at less than 1,000 ppm
This system can prevent the spread of infection in hospitals.

---

### 3. Smart Agricultural Ventilation System
Relative humidity below 70%, CO₂ concentration should be maintained below 1,200 ppm.
This system can prevent the occurrence of pests.
