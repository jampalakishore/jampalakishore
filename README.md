- 👋 Hi, I’m @jampalakishore
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
jampalakishore/jampalakishore is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
PROGRAM:
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

// WiFi settings
const char* ssid = "BalaMurugan"; // Replace with your WiFi network name
const char* password = "8220638855"; // Replace with your WiFi password

// MQTT Broker settings
const char* mqtt_broker = "test.mosquitto.org"; // Using Mosquitto's test broker
const char* topic = "temperature"; // Topic to publish temperature data
const int mqtt_port = 1883; // Default MQTT port

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
 
  // Connect to MQTT Broker
  client.setServer(mqtt_broker, mqtt_port);
  while (!client.connected()) {
    Serial.println("Connecting to MQTT Broker...");
    if (client.connect("ESP8266Client")) { // Client ID for MQTT
      Serial.println("Connected to MQTT Broker!");
    } else {
      Serial.println("Failed to connect to MQTT Broker");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    // Reconnect if connection is lost
    while (!client.connected()) {
      Serial.println("Reconnecting to MQTT Broker...");
      if (client.connect("ESP8266Client")) {
        Serial.println("Reconnected to MQTT Broker!");
      } else {
        Serial.println("Failed to reconnect to MQTT Broker");
        delay(5000);
      }
    }
  }
  client.loop();
 
  // Read temperature from LM35
  int analogValue = analogRead(A0);
  float voltage = (analogValue / 1023.0) * 3300; // Convert to voltage (in mV)
  float temperature = voltage / 10; // Convert voltage to temperature
 
  // Convert temperature to string and publish to MQTT
  char temperatureString[8];
  dtostrf(temperature, 1, 2, temperatureString);
  client.publish(topic, temperatureString);
 
  delay(10000); // Delay between readings
}

