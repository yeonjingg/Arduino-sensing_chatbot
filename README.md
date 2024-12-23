# Arduino-sensing_chatbot
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


