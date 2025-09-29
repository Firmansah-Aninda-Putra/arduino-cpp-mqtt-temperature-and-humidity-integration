## 📡 ESP32 Sending Sensor Readings Using MQTT Protocol

### I. PRACTICUM OBJECTIVE

To send sensor reading results using the **MQTT protocol**.

-----

### II. TOOLS AND MATERIALS

1.  Laptop/PC with internet connection
2.  Smartphone (Android)

-----

### III. THEORETICAL BASIS

#### MQTT Protocol

**MQTT (Message Queuing Telemetry Transport)** is a standard-based messaging protocol, or set of rules, used for **machine-to-machine communication**. Smart sensors, wearables, and other Internet of Things (**IoT**) devices typically need to send and receive data over networks with limited resources and bandwidth. These IoT devices use MQTT for data transmission because it's **easy to implement** and can communicate IoT data efficiently. MQTT supports message delivery between devices-to-cloud and cloud-to-devices.

#### MQTT Components

MQTT implements a **publish/subscribe model** by defining clients and a broker:

1.  **MQTT Client:** Any device, from a server to a microcontroller, running an MQTT library. If a client sends a message, it acts as a **publisher**; if it receives a message, it acts as a **subscriber**. Essentially, any device communicating using MQTT over a network is an MQTT client device.
2.  **MQTT Broker:** The backend system that **coordinates messages** between different clients. The broker's responsibilities include:
      * Receiving and filtering messages.
      * Identifying which clients are subscribed to each message.
      * Forwarding messages to interested subscribers.
      * Authorizing and authenticating MQTT clients.
      * Sending messages to other systems for further analysis.
      * Handling undelivered messages and client sessions.

#### How MQTT Works

1.  An **MQTT client** establishes a connection with the **MQTT broker**.
2.  Once connected, the client can **publish** messages, **subscribe** to specific messages, or do both.
3.  When the MQTT broker receives a message, it **forwards that message to interested subscribers**.

-----

### IV. PRACTICUM STEPS

This practicum uses the free public MQTT server provided by **`www.emqx.io`** (`broker.emqx.io`).

#### A. ESP32 Coding and Wiring

1.  **Wire the ESP32 and DHT11 sensor** as shown in the diagram.
    **Wiring Details:**
    a. DHT11 Data Pin connects to ESP32 pin **D4**.
    b. DHT11 VCC Pin connects to **VCC**.
    c. DHT11 GND Pin connects to **GND**.
2.  Add the necessary libraries: `WiFi.h`, `PubSubClient.h`, `DHT.h`.
3.  Select the **DOIT ESP32 DEVKIT V1** board.
4.  **Write the source code** and make adjustments for:
    a. `WIFI_SSID`: your Wi-Fi network name.
    b. `WIFI_PASSWORD`: your Wi-Fi network password.
    c. `TOPIK_ANDA`: a unique name for your topic.

**(Source Code Provided)**

-----

#### B. Preparing the MQTT Client (MQTT X Application)

This practicum uses the **MQTT X** application ([https://mqttx.app](https://mqttx.app)) as the MQTT client.

1.  Install MQTT X on your computer.
2.  Create a **new connection** by clicking the "New Connection" button or the **`+`** symbol.
3.  Fill in the connection information according to the ESP32 script and click **Connect**:
      * **Host Address:** `broker.emqx.io`
      * **Username:** `emqx`
      * **Password:** `public`
      * **Port:** `1883`
      * *(Enter a name as needed)*
4.  Next, create a **new subscription** by clicking "New Subscription."
5.  Enter the topic that matches the one in the ESP32 code (e.g., `TOPIK_ANDA`).

-----

#### C. Testing

1.  Start the application on the ESP32 and observe the results in MQTT X.
2.  If successful, the ESP32 will send data containing temperature and humidity information **every 2 seconds** to the MQTT server, and the results will be displayed in MQTT X.

-----

### V. Assignment

1.  Show that the initial MQTT test was successful\!
2.  Run the application, then observe by changing the temperature and humidity. How does the application work?
3.  Modify the source code so that the ESP32 sends humidity and temperature to **separate topics** (changing to two topics)\!
4.  Attempt to send the information **your\_name** from the MQTT server (using MQTT X) to the ESP32 with the topic **`tugas_praktik`** and prove it with a screenshot\!

#### 2\. Application Workflow Explanation

The monitoring system works through integrated serial and network protocols. The ESP32 reads analog data from the **DHT11 sensor** via the one-wire protocol on GPIO pin 4. The sensor uses **PWM (Pulse Width Modulation)** to transmit 40-bit data (16-bit humidity, 16-bit temperature, 8-bit checksum), which the DHT library converts into digital values.

This data is then sent via the **MQTT protocol** using the ESP32's WiFi stack to the public cloud broker **`broker.emqx.io`** via a TCP/IP connection on port 1883. For monitoring, the laptop is connected to the ESP32 via a **USB-to-Serial converter** (CH340/CP210x) for **UART communication** at a baud rate of 115200 to view the serial log. Simultaneously, the laptop uses the **MQTT X application** connected to the internet to **subscribe** to the specified topic, receiving the **real-time data** published by the ESP32. Monitoring can thus be done through both the serial monitor and the MQTT X dashboard.

-----

#### 3\. Modified Source Code for Two Topics

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"

#define DHTPIN 4  
#define DHTTYPE DHT11 
 
const char* ssid = "3-JAGOANKU"; // << Your WiFi SSID
const char* password = "janganlupabismillah"; // << Your WiFi Password
 
const char *mqtt_broker = "broker.emqx.io";
const char *mqtt_topic_temperature = "weather-monitor/temperature";  // Topic for temperature
const char *mqtt_topic_humidity = "weather-monitor/humidity";        // Topic for humidity
const char *mqtt_username = "emqx";
const char *mqtt_password = "public";
const int mqtt_port = 1883;
 
WiFiClient esp_client;
PubSubClient mqtt_client(esp_client);
unsigned long lastMsg = 0;

DHT dht(DHTPIN, DHTTYPE);
 
void connectToWiFi() { 
    WiFi.begin(ssid, password); 
    Serial.print("Connecting to WiFi"); 
    while (WiFi.status() != WL_CONNECTED) { 
        delay(500); 
        Serial.print("."); 
    } 
    Serial.println("\nConnected to WiFi"); 
} 
 
void connectToMQTT() { 
    while (!mqtt_client.connected()) { 
        String client_id = "esp32-client" + String(WiFi.macAddress()); 
        Serial.printf("Connecting to MQTT Broker as %s...\n", client_id.c_str()); 
        if (mqtt_client.connect(client_id.c_str(), mqtt_username, mqtt_password)) { 
            Serial.println("Connected to MQTT broker"); 
        } else { 
            Serial.print("Failed to connect to MQTT broker, rc="); 
            Serial.print(mqtt_client.state()); 
            Serial.println(" Retrying in 5 seconds."); 
            delay(5000); 
        } 
    } 
} 
 
void mqttCallback(char *topic, byte *payload, unsigned int length) { 
    Serial.print("Message received on topic: "); 
    Serial.println(topic); 
    Serial.print("Message: "); 
    for (unsigned int i = 0; i < length; i++) { 
        Serial.print((char) payload[i]); 
    } 
    Serial.println("\n-----------------------"); 
} 
 
void setup() { 
    Serial.begin(115200); 
    dht.begin(); 
    connectToWiFi(); 
    mqtt_client.setServer(mqtt_broker, mqtt_port); 
    mqtt_client.setKeepAlive(60); 
    mqtt_client.setCallback(mqttCallback); 
    connectToMQTT(); 
    // Subscribe to both topics (optional, for monitoring) 
    mqtt_client.subscribe(mqtt_topic_temperature); 
    mqtt_client.subscribe(mqtt_topic_humidity);
    // Subscribe to the new topic for assignment 4
    mqtt_client.subscribe("tugas_praktik");
} 
 
void loop() { 
    float h = dht.readHumidity(); 
    float t = dht.readTemperature(); 
     
    if (isnan(h) || isnan(t)) { 
        Serial.println(F("Failed to read from DHT sensor!")); 
        return; 
    } 
     
    String suhu = String(t, 2) + " Celcius"; 
    String kelembapan = String(h, 1) + " %"; 
     
    mqtt_client.loop(); 
    unsigned long now = millis(); 
     
    if (now - lastMsg > 2000) { 
        lastMsg = now; 
         
        // Publish temperature to the temperature topic 
        mqtt_client.publish(mqtt_topic_temperature, suhu.c_str()); 
        Serial.println("Published temperature: " + suhu + " to topic: " + String(mqtt_topic_temperature)); 
         
        // Publish humidity to the humidity topic   
        mqtt_client.publish(mqtt_topic_humidity, kelembapan.c_str()); 
        Serial.println("Published humidity: " + kelembapan + " to topic: " + String(mqtt_topic_humidity)); 
    } 
}
```

-----

#### 4\. Sending Information from MQTT Server to ESP32

The process of sending information from MQTTX to the ESP32 is shown by subscribing the ESP32 to a specific topic (`tugas_praktik`) and then publishing a message to that topic from the MQTTX client.

**A. MQTTX PUBLISH OUTPUT SCREENSHOT**

*(This screenshot would show the MQTTX application successfully publishing the message "FIRMANSAH ANINDA PUTRA" to the topic `tugas_praktik`.)*

**B. ARDUINO IDE SERIAL MONITOR OUTPUT SCREENSHOT**

*(This screenshot would show the ESP32 Serial Monitor displaying the output of the `mqttCallback` function, confirming the reception of the message.)*

`Message received on topic: tugas_praktik`
`Message: FIRMANSAH ANINDA PUTRA`
`-----------------------`
