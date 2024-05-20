# ECGMonitor

O referido código mostra o desenvolvimento do código para um dispositivo caseiro de ECG (Eletrocardiograma) construido a partir de uma placa de desenvolvimento ESP32, um sensor de batimentos cardiacos AD822 e outros circuitos. Para o funcionamento, você deverá ter os mesmos componentes descrito:


Placa de desenvolvimento: Módulo WiFi Bluetooth ESP32-WROOM-32U Ipex	1 unidade
Sensor de batimentos cardíacos: AD8232	1 unidade
Cablo Micro-USB	1 unidade
Peças de fio jumper	1 unidade
Placas de ensaio	1 unidade


Além, deverá criar uma conta no Ubidots para relacionar os dados com o dashboard e, portanto, ser possivel analisar e verificar em modelos gráficos os resultados obtidos.
O código é aplicado diretamente no software:

#include <WiFi.h>
#include <PubSubClient.h>
#include <math.h>

#define WIFISSID "NomeDaRedeWifi"      
#define PASSWORD "SenhaDaRedeWifi" 
#define TOKEN "Token" 
#define MQTT_CLIENT_NAME "MonitorECG"               
 
****************************************/
#define VARIABLE_LABEL "sensor" 
#define DEVICE_LABEL "esp32"    
 
#define SENSOR A0 
 
char mqttBroker[]  = "industrial.api.ubidots.com";
char payload[100];
char topic[150];
char str_sensor[10];
 
WiFiClient ubidots;
PubSubClient client(ubidots);

void callback(char* topic, byte* payload, unsigned int length) {
  char p[length + 1];
  memcpy(p, payload, length);
  p[length] = NULL;
  Serial.write(payload, length);
  Serial.println(topic);
}

void reconnect() {
  while (!client.connected()) {
    Serial.println("Tentando conexão MQTT...");
    
  if (client.connect(MQTT_CLIENT_NAME, TOKEN, "")) {
      Serial.println("Connected");
    } else {
      Serial.print("Failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 2 seconds");

   delay(2000);
  }
  }
}

float simulateECG(int time) {
  float value = 2048 + 1024 * sin(2 * PI * time / 1000.0) + 200 * (rand() % 10 - 5); // Sinal de ECG simplificado
  return value;
}

void setup() {
  Serial.begin(115200);
  WiFi.begin(WIFISSID, PASSWORD);
  pinMode(SENSOR, INPUT);
 
  Serial.println();
  Serial.print("Esperando por WiFi ...");
  
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }
  
  Serial.println("");
  Serial.println("WiFi conectado");
  Serial.println("Endereço de IP:");
  Serial.println(WiFi.localIP());
  client.setServer(mqttBroker, 1883);
  client.setCallback(callback);  
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
 
  sprintf(topic, "%s%s", "/v1.6/devices/", DEVICE_LABEL);
  sprintf(payload, "%s", ""); 
  sprintf(payload, "{\"%s\":", VARIABLE_LABEL); 
  
  int time = millis();
  float sensor = simulateECG(time); 
  
  dtostrf(sensor, 4, 2, str_sensor);
  
  sprintf(payload, "%s {\"value\": %s}}", payload, str_sensor); 
  Serial.println
  client.publish(topic, payload);
  client.loop();
  delay(500);
}
