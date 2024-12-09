# Home-Automation-03

#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

// Wi-Fi credentials
const char* ssid = "T R I N I T Y"; // Replace with your Wi-Fi SSID
const char* password = "Shihab007!2#4%"; // Replace with your Wi-Fi password

// ESP8266WebServer instance
ESP8266WebServer server(80);

// GPIO pin definitions
const int light1 = 4; // Room 1 Light 1 (GPIO4)
const int light2 = 5; // Room 1 Light 2 (GPIO5)
const int buzzer = 0; // Buzzer (GPIO0)
const int motionSensor = 2; // Motion Sensor (GPIO2)

const int light3 = 12; // Room 2 Light 1 (GPIO12)
const int light4 = 13; // Room 2 Light 2 (GPIO13)

// Motion sensor timing
unsigned long lastMotionDetected = 0; 
const unsigned long motionTimeout = 30000; // 30 seconds timeout
bool motionDetected = false;

// HTML content for the web page
void handleRoot() {
  server.send(200, "text/html", R"rawliteral(
  <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Smart Home Control System</title>
  <style>
  body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #070313;
    color: #ffffff;
  }
  header {
    background: #1f1f1f;
    color: #ff7c11;
    text-align: center;
    padding: 1em;
  }
  section {
    padding: 20px;
    margin: 20px;
    background: #1e1e1e;
    border-radius: 5px;
  }
  button {
    padding: 10px;
    background: #09327e;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin: 5px;
  }
  button:hover {
    background: #45a049;
  }
  </style>
</head>
<body>
  <header>
    <h1>Smart Home Control System</h1>
  </header>
  <main>
    <section>
      <h2>Room 1</h2>
      <button onclick="toggleLight('light1')">Toggle Light 1</button>
      <span id="light1-status">Off</span><br>
      <button onclick="toggleLight('light2')">Toggle Light 2</button>
      <span id="light2-status">Off</span><br>
      <button onclick="toggleBuzzer()">Toggle Buzzer</button>
      <span id="buzzer-status">Off</span><br>
      <span>Motion Sensor Status: </span><span id="motion-sensor-status">Inactive</span>
    </section>
    <section>
      <h2>Room 2</h2>
      <button onclick="toggleLight('room2-light1')">Toggle Light 1</button>
      <span id="room2-light1-status">Off</span><br>
      <button onclick="toggleLight('room2-light2')">Toggle Light 2</button>
      <span id="room2-light2-status">Off</span>
    </section>
    <script>
      function toggleLight(lightId) {
        fetch(`/${lightId}`).then(() => {
          const status = document.getElementById(`${lightId}-status`);
          status.textContent = status.textContent === "Off" ? "On" : "Off";
        });
      }
      function toggleBuzzer() {
        fetch("/toggleBuzzer").then(() => {
          const status = document.getElementById("buzzer-status");
          status.textContent = status.textContent === "Off" ? "On" : "Off";
        });
      }
    </script>
  </main>
</body>
</html>
  )rawliteral");
}

// Room 1 Light 1 control
void toggleLight1() {
  digitalWrite(light1, !digitalRead(light1));
  server.send(200, "text/plain", "Light 1 Toggled");
}

// Room 1 Light 2 control
void toggleLight2() {
  digitalWrite(light2, !digitalRead(light2));
  server.send(200, "text/plain", "Light 2 Toggled");
}

// Room 2 Light 1 control
void toggleLight3() {
  digitalWrite(light3, !digitalRead(light3));
  server.send(200, "text/plain", "Room 2 Light 1 Toggled");
}

// Room 2 Light 2 control
void toggleLight4() {
  digitalWrite(light4, !digitalRead(light4));
  server.send(200, "text/plain", "Room 2 Light 2 Toggled");
}

// Buzzer control
void toggleBuzzer() {
  digitalWrite(buzzer, !digitalRead(buzzer));
  server.send(200, "text/plain", "Buzzer Toggled");
}

void setup() {
  Serial.begin(115200);

  pinMode(light1, OUTPUT);
  pinMode(light2, OUTPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(motionSensor, INPUT);
  pinMode(light3, OUTPUT);
  pinMode(light4, OUTPUT);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Wi-Fi Connected!");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.on("/light1", toggleLight1);
  server.on("/light2", toggleLight2);
  server.on("/room2-light1", toggleLight3);
  server.on("/room2-light2", toggleLight4);
  server.on("/toggleBuzzer", toggleBuzzer);

  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();

  // Motion Sensor Handling
  if (digitalRead(motionSensor) == HIGH) {
    lastMotionDetected = millis();
    motionDetected = true;
  } else if (millis() - lastMotionDetected > motionTimeout && motionDetected) {
    digitalWrite(light1, LOW);
    digitalWrite(light2, LOW);
    motionDetected = false;
  }
}
