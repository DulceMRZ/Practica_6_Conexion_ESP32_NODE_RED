# Practica 6_ ESP32 con NODE-RED

**Sensor de Temperatura y Humedad+ Node -RED**

En esta práctica podemos programar una ESP32 con el sensor DHT22.

## Contenido 

- Introducción 
- Objetivo
- Material Requerido
- Procedimiento 
- Instrucciones de operación 
- Resultados 



## 1. Introducción

La ```Esp32``` la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (```DTH22```).
### 1.1 Descripción
 El ```DHT22``` utiliza un solo cable digital para la comunicación de datos, lo que hace que su conexión sea sencilla y fácil de implementar en proyectos de electrónica. Este sensor proporciona mediciones de temperatura y humedad de alta precisión, con una precisión de +/-0.5 °C para la temperatura y +/-2-5% para la humedad para adquirir temperatura y humedad del entorno. 
 
 ### 1.2 Especificación 
 Es importante considerar que esta practica se estara realiando en un simulador llamado [WOKWI](https://https://wokwi.com/).

## 2. Objetivo 

Poder diseñar un diagrama con sensor DHT22 de temperatura y humedad que se utilice para medir la temperatura y la humedad relativa del ambiente en el que se encuentra, mediante la utilización de una ESP32.


## 3. Material Requerido

Tomar en cuenta el material necesario para realizar esta práctica:

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Sensor DHT22



## 4. Procedimiento a realizar 

 - Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


## Paso 1 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt= "dulcemrz";
String password_mqtt= "1241";

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
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
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
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("dulce/tokai", output.c_str());
  }
}
```

## Paso 2 

2. Instalar las librerias
   **DHT sensor library for ESPx** 
   **ArduinoJson**
   **PubSubClient**

Como se muestra en la siguente imagen.

![](https://github.com/DulceMRZ/Practica_6_Conexion_ESP32_NODE_RED/blob/main/Captura%201.PNG?raw=true)

## Paso 3

3. Hacer la conexion de **DHT22** con la **ESP32** como se muestra en la siguentes imagenes:

3.1 Es importante considerar que para **ESP32** se maneja un ```Voltaje de trabajo 3.3 VDC```. 

a) Observar conexión de pin del Sensor al **pin 1** de **ESP32**. 

![](https://github.com/DulceMRZ/PRACTICA_1_DHT22/blob/main/Practica_1_DTH22%20diagrama%20(1).PNG?raw=true)

b) Conexión de pin 2 (GND) 


![](https://github.com/DulceMRZ/PRACTICA_1_DHT22/blob/main/Practica_1_DTH22%20diagrama%20(2)..PNG?raw=true)

c) Conexión pin 3 (pin 15) 


![](https://github.com/DulceMRZ/PRACTICA_1_DHT22/blob/main/Practica_1_DTH22%20diagrama..png?raw=true)







d) Corrobora que el simulador compile bien el programa 

![](https://github.com/DulceMRZ/Practica_6_Conexion_ESP32_NODE_RED/blob/main/Datos%20Repositorios%20-%20Hojas%20de%20c%C3%A1lculo%20de%20Google%20-%20Google%20Chrome%2016_06_2023%2008_54_15%20p.%20m..png?raw=true)


### 5. Conexión de Node - Red











### 6. Instrucciónes de operación

1. Iniciar simulador.
2. Visualizar los datos en el monitor serial.
3. Colocar la temperatura y humedad dando *doble click* al sensor **DHT11** 

## 7. Resultados

Cuando haya funcionado, verás los valores dentro del monitor serial como se muestra en la siguente imagen.

![]()


# Créditos

Desarrollado por Dulce M Rodriguez Zarate 

- [GitHub](https://github.com/DulceMRZ)