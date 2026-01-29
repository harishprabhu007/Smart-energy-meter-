

smart-energy-meter/
│
├── arduino/
│   └── energy_meter.ino
│
├── server/
│   ├── app.py
│   └── templates/
│       └── index.html
│
├── README.md
└── requirements.txt
1️⃣ Arduino code (arduino/energy_meter.ino)
Example using ACS712 current sensor:
Copy code
Cpp
const int sensorPin = A0;
float sensitivity = 0.185; // for ACS712 5A module
float offsetVoltage = 2.5; // midpoint voltage
float voltage = 230.0;     // mains voltage (assumed constant)

float energy_kwh = 0;
unsigned long lastTime = 0;

void setup() {
  Serial.begin(9600);
  lastTime = millis();
}

void loop() {
  int raw = analogRead(sensorPin);
  float sensorVoltage = (raw / 1023.0) * 5.0;
  float current = (sensorVoltage - offsetVoltage) / sensitivity;
  if(current < 0) current = -current;

  float power = voltage * current; // in watts

  unsigned long now = millis();
  float hours = (now - lastTime) / 3600000.0;
  lastTime = now;

  energy_kwh += (power * hours) / 1000.0;

  // Send: current,power,energy
  Serial.print(current);
  Serial.print(",");
  Serial.print(power);
  Serial.print(",");
  Serial.println(energy_kwh);

  delay(1000);
}
2️⃣ Python Flask server (server/app.py)
Copy code
Python
from flask import Flask, render_template
import serial

app = Flask(__name__)
ser = serial.Serial('COM3', 9600)  # change port

data_values = {"current":0, "power":0, "energy":0}

@app.route('/')
def index():
    if ser.in_waiting:
        line = ser.readline().decode().strip()
        parts = line.split(',')
        if len(parts) == 3:
            data_values["current"] = float(parts[0])
            data_values["power"]   = float(parts[1])
            data_values["energy"]  = float(parts[2])
    return render_template("index.html", data=data_values)

app.run(debug=True)
3️⃣ Web dashboard (server/templates/index.html)
Copy code
Html
<!DOCTYPE html>
<html>
<head>
    <title>Smart Energy Meter</title>
    <meta http-equiv="refresh" content="2">
    <style>
        .box{
            width:200px;height:120px;
            display:inline-block;margin:20px;
            padding:20px;font-size:20px;
            background:#222;color:#0f0;
        }
    </style>
</head>
<body>
    <h1>Smart Energy Meter</h1>
    <div class="box">Current<br>{{data.current}} A</div>
    <div class="box">Power<br>{{data.power}} W</div>
    <div class="box">Energy<br>{{data.energy}} kWh</div>
</body>
</html>
4️⃣ requirements.txt
Copy code

flask
pyserial
Install:
Copy code
Bash
pip install -r requirements.txt
5️⃣ README.md content for GitHub
Copy code
Markdown
# Smart Energy Meter ⚡

IoT based smart energy meter using Arduino and ACS712 current sensor.

## Functions
- Measures current, power and energy
- Displays real-time data on a web dashboard
- Calculates total energy in kWh

## Hardware
- Arduino Uno
- ACS712 current sensor
- Connecting wires

## How it works
1. Sensor measures load current.
2. Arduino calculates power and energy.
3. Data sent via Serial to Python server.
4. Flask web page shows live readings.

## Run
1. Upload Arduino code.
2. Update COM port in `app.py`.
3. Run:
   ```bash
   python app.py
Open browser at http://127.0.0.1:5000
Copy code

## Ideas to improve (for better GitHub project)
- Add voltage sensor (ZMPT101B) for real voltage measurement  
- Add Wi-Fi (ESP8266/ESP32) to view data on mobile  
- Add cost calculation (₹ per kWh)  
- Store data in a database and plot graphs  

This will make a good academic/mini project and a clean GitHub repository showing IoT + embedded + web integration# Smart-energy-meter-
