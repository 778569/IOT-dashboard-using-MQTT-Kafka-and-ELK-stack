# IOT-dashboard-using-MQTT-Kafka-and-ELK-stack
In this Repository shows how to monitoring data using Node MCU( ESP8266), MQTT, Node Red, Kafka and ELK stack

1 Step - 

Creat Arduino code-

#include <ESP8266WiFi.h> // should download this Library 
#include <PubSubClient.h> // should download this Library 
#include <DHT.h> // should download this Library depend your sensor type

//DHT11 for reading temperature and humidity value
#define DHTPIN            0

// Uncomment whatever type you're using!
//#define DHTTYPE DHT11     // DHT 11
//#define DHTTYPE DHT22   // DHT 22, AM2302, AM2321
#define DHTTYPE DHT21   // DHT 21, AM2301

DHT dht(DHTPIN, DHTTYPE);

// Update these with values suitable for your network.

const char* ssid = "";// your wifi name 
const char* password = ""; //  your wifi password 
const char* mqtt_server = "192.168.8.171";  // Local IP address of the MQTT Broker

const char* username = "admin"; // This is set by default by HIVEMQ
const char* pass = "hivemq";

char str_hum_data[10];
char str_temp_data[10];

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username, pass) ) {
      Serial.println("connected");
      // Once connected, publish an announcement...
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  setup_wifi();
  client.setServer(mqtt_server, 1883);
}

void loop() {

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 10000) {
    float hum_data = dht.readHumidity();
    Serial.println(hum_data);
    /* 4 is mininum width, 2 is precision; float value is copied onto str_sensor*/
    dtostrf(hum_data, 4, 2, str_hum_data);

    
    float temp_data = dht.readTemperature(); // or dht.readTemperature(true) for Fahrenheit
    dtostrf(temp_data, 4, 2, str_temp_data);
    lastMsg = now;
    Serial.print("Publish message: ");
    Serial.print("Temperature - "); Serial.println(str_temp_data);
    client.publish("device2/temp", str_temp_data);
    Serial.print("Humidity - "); Serial.println(str_hum_data);
    client.publish("device2/hum", str_hum_data);
  }
}
