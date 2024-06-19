# IotPark
This repository implements a user-friendly and cost-effective smart parking system using an ESP32 development board, MicroPython for programming, and Wi-Fi for data transmission. It leverages ultrasonic sensors to detect vehicle presence in parking spots and displays real-time availability information.
---

# Parking Spot Monitoring System

## Overview

The Parking Spot Monitoring System is designed to help users easily find available parking spots in a parking lot. This system uses IR sensors to detect the presence of vehicles in parking spots and an ESP32 microcontroller to send real-time data to a server. The data is then displayed on a web interface, which can be accessed via any device connected to the same Wi-Fi network.

## Components and Tools

### Hardware
- ESP32 Development Board
- IR Sensors (or Ultrasonic Sensors)
- Jumper Wires (Male-to-Male, Male-to-Female, Female-to-Female)
- Breadboard
- USB Cable (for ESP32 power and programming)
- Resistors (if needed, based on sensor requirements)

### Software
- MicroPython (for ESP32)
- Flask (Python-based backend framework)
- HTML/CSS and JavaScript (for frontend display)
- Wi-Fi network for communication

## System Architecture

1. **IR Sensors**: Detect the presence of a vehicle in a parking spot.
2. **ESP32 Microcontroller**: Reads sensor data and sends it to the server via Wi-Fi.
3. **Flask Server**: Receives data from the ESP32 and updates the parking spot status.
4. **Web Interface**: Displays real-time parking spot status, accessible via any device connected to the same Wi-Fi network.

## Setup and Installation

### Step 1: Flash MicroPython onto ESP32

1. Download the latest MicroPython firmware for ESP32 from [MicroPython Downloads](https://micropython.org/download/esp32/).
2. Flash the firmware using esptool:
   ```bash
   esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash
   esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 esp32-20210902-v1.17.bin
   ```

### Step 2: Connect ESP32 to Sensors

1. **Power Rails**:
   - Connect ESP32 3.3V to the breadboard positive rail.
   - Connect ESP32 GND to the breadboard negative rail.

2. **Sensor Connections**:
   - Connect each sensor's VCC to the positive rail.
   - Connect each sensor's GND to the negative rail.
   - Connect each sensor's OUT pin to a unique GPIO pin on the ESP32 (e.g., GPIO 32, 33, 34, 35, 36).

### Step 3: MicroPython Code for ESP32

```python
import machine
import network
import urequests as requests
import time

# Wi-Fi configuration
ssid = 'your_SSID'
password = 'your_PASSWORD'

# Sensor pin configuration
sensor_pins = [32, 33, 34, 35, 36]  # Example GPIO pins for 5 sensors
sensors = [machine.Pin(pin, machine.Pin.IN) for pin in sensor_pins]

# Connect to Wi-Fi
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(ssid, password)

while not wlan.isconnected():
    time.sleep(1)

print('Connected to Wi-Fi')

# Function to send sensor data to the server
def send_data(sensor_data):
    url = 'http://192.168.1.x:5000/update'  # Replace with your server IP address
    headers = {'Content-Type': 'application/json'}
    data = {
        'spots': sensor_data
    }
    response = requests.post(url, json=data, headers=headers)
    print(response.text)

while True:
    sensor_data = [sensor.value() for sensor in sensors]
    send_data(sensor_data)
    time.sleep(5)
```

### Step 4: Setup Flask Backend

1. Install Flask:
   ```bash
   pip install flask
   ```

2. Create a `app.py` file with the following code:

```python
from flask import Flask, request, jsonify, send_from_directory

app = Flask(__name__)

parking_spots = [0] * 5  # Example for 5 parking spots

@app.route('/update', methods=['POST'])
def update_spots():
    global parking_spots
    data = request.json
    parking_spots = data['spots']
    return jsonify({'status': 'success'})

@app.route('/spots', methods=['GET'])
def get_spots():
    return jsonify(parking_spots)

@app.route('/')
def index():
    return send_from_directory('.', 'index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 5: Create HTML and JavaScript for Frontend

Create an `index.html` file with the following content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Parking Spot Monitor</title>
    <script>
        async function fetchSpots() {
            const response = await fetch('/spots');
            const data = await response.json();
            document.getElementById('spot1').innerText = data[0] ? 'Occupied' : 'Available';
            document.getElementById('spot2').innerText = data[1] ? 'Occupied' : 'Available';
            document.getElementById('spot3').innerText = data[2] ? 'Occupied' : 'Available';
            document.getElementById('spot4').innerText = data[3] ? 'Occupied' : 'Available';
            document.getElementById('spot5').innerText = data[4] ? 'Occupied' : 'Available';
        }
        setInterval(fetchSpots, 5000); // Refresh every 5 seconds
    </script>
</head>
<body>
    <h1>Parking Spot Monitor</h1>
    <p>Spot 1: <span id="spot1">Loading...</span></p>
    <p>Spot 2: <span id="spot2">Loading...</span></p>
    <p>Spot 3: <span id="spot3">Loading...</span></p>
    <p>Spot 4: <span id="spot4">Loading...</span></p>
    <p>Spot 5: <span id="spot5">Loading...</span></p>
</body>
</html>
```

### Step 6: Run the Flask Server

```bash
python app.py
```

### Step 7: Access the Web Interface

1. Find the local IP address of the machine running the Flask server.
2. Open a web browser on your phone or any other device connected to the same Wi-Fi network.
3. Enter the local IP address followed by the port number (e.g., `http://192.168.1.x:5000`).

## Usage

- Ensure the ESP32 and your display device (phone, tablet, computer) are connected to the same Wi-Fi network.
- Open the web interface on your display device to see the real-time status of parking spots.

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your changes. If you encounter any issues, feel free to open an issue on GitHub.

## License

This project is licensed under the MIT License.

---

This README file provides a comprehensive overview of this project, detailed setup instructions, and usage guidelines. We can further enhance it by adding sections such as "Troubleshooting," "Future Work," or "Acknowledgments" as needed.
