```c
////////////////////////////////////////////////////////////////////////////////
#include <Arduino.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>

#ifdef ESP8266
#include <ESP8266WiFi.h>
#else
#include <WiFi.h>
#endif

const char *ssid = "sometime";        // Tên Wifi
const char *password = "zxcvbnmm";    // Mật khẩu Wifi
const char *server_mqtt = "95c817cff7c244168ec9af03218947d5.s1.eu.hivemq.cloud";
const int port_mqtt = 8883;
const char *mqtt_id = "esp32";
const char *topic_subscribe = "esp8266";
const char *topic_publish = "esp32";
const char *mqtt_username = "thanhan2003.tn@gmail.com"; // Tên đăng nhập MQTT
const char *mqtt_password = "Lythanhan221003";          // Mật khẩu MQTT

String message_send = "";

WiFiClientSecure wifiClient;
PubSubClient mqtt_client(wifiClient);

void callback(char *topic, byte *payload, unsigned int length)
{
  String message = "";
  Serial.print("Received from: ");
  Serial.println(topic);
  Serial.print("Message: ");
  for (size_t i = 0; i < length; i++)
  {
    message += (char)payload[i];
  }
  Serial.println(message);
  Serial.println("----------------------------------------");
}

void setup()
{
  Serial.begin(9600);
  Serial.println("Connecting to Wifi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.print("Connected to Wifi ");
  Serial.println(ssid);
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  delay(1000);

  wifiClient.setInsecure(); // Tạm thời bỏ qua kiểm tra chứng chỉ SSL

  mqtt_client.setServer(server_mqtt, port_mqtt);
  mqtt_client.setCallback(callback);

  Serial.println("Connecting to MQTT...");
  while (!mqtt_client.connect(mqtt_id, mqtt_username, mqtt_password))
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected to MQTT");
  mqtt_client.subscribe(topic_subscribe);
}

void recivedMessageFromUart()
{
  if (Serial.available())
  {
    char c = (char)Serial.read();
    message_send += c;
    if (c == '\n')
    {
      Serial.print("Message to send: ");
      Serial.println(message_send);
      if (mqtt_client.connected())
      {
        bool result = mqtt_client.publish(topic_publish, message_send.c_str());
        if(result) {
          Serial.println("Message sent successfully");
        } else {
          Serial.println("Failed to send message");
        }
        message_send = "";
      } else {
        Serial.println("MQTT client not connected");
      }
    }
  }
}

void loop()
{
  mqtt_client.loop();
  recivedMessageFromUart();
}

```
