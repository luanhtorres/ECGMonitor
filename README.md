# ECGMonitor

1) Uma breve descrição do funcionamento e uso para quem quiser reproduzir:

O referido código mostra o desenvolvimento do código para um dispositivo caseiro de ECG (Eletrocardiograma) construido a partir de uma placa de desenvolvimento ESP32, um sensor de batimentos cardiacos AD822 e outros circuitos. O eletrocardiograma é um exame rápido e simples. Os impulsos elétricos do coração são registrados em um pedaço de papel. Antes do procedimento, o paciente repousa por 5 minutos em uma maca, garantindo que fatores externos, como atividade física, cigarro ou ingestão de alimentos, não influenciem no resultado. Durante o exame, eletrodos são colocados na região frontal do peito, punhos e tornozelos, com a aplicação de gel para facilitar a condução elétrica. A pele deve estar limpa e sem oleosidade, podendo ser necessário depilar áreas com muitos pelos ou realizar limpeza com álcool em regiões oleosas. Os eletrodos são conectados ao eletrocardiógrafo, que registra a atividade elétrica do coração através de fios. Ao final, o aparelho imprime 12 visões distintas do órgão para análise.
Normalmente, os resultados do eletrocardiograma podem ser visualizados imediatamente após a conclusão do exame. Caso sejam identificados quaisquer problemas nos batimentos cardíacos, pode ser necessário repetir o ECG ou realizar outros exames complementares, como ecocardiograma, holter ou exames mais elaborados. 


2) O software desenvolvido e a documentação de código:

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

3) Descrição do hardware utilizado (plataformas de desenvolvimento, sensores, atuadores, impressão 3D de peças, medidas de peças e caixas etc):

Para construção do sistema de monitoramento, alguns materiais serão necessários:

Componentes	Quantidade
Placa de desenvolvimento: Módulo WiFi Bluetooth ESP32-WROOM-32U Ipex	1 unidade
Sensor de batimentos cardíacos: AD8232	1 unidade
Cablo Micro-USB	1 unidade
Peças de fio jumper	1 unidade
Placas de ensaio	1 unidade

Abaixo as especificações de cada componente:

Placa de desenvolvimento: Módulo WiFi Bluetooth ESP32-WROOM-32U
Sensor de batimentos cardíacos: AD8232 (folha de dados do fabricante)                          
O sensor tem nove conexões para soldar pinos e outros conectores, sendo os principais para o funcionamento: SDN, LO+, LO-, OUTPUT, 3.3V, GND.
Cabo Micro-USB (folha de dados do fabricante)
Peças de fio jumper (folha de dados do fabricante)
Placas de ensaio (folha de dados do fabricante)


4 e 5) A documentação das interfaces, protocolos e módulos de comunicação:

Para registrar os dados obtidos precisamos publicar e utilizar uma plataforma online, no caso será utilizada a Ubidots, que é capaz de fornecer um ambiente de desenvolvimento para captar os dados do sensor e transformar em informações úteis. Ela fará o intermédio do dispositivo com a internet.
Na aba de “Devices”, podemos selecioná-lo e então criar um dispositivo:
Logo após, devemos criar uma variável.
Devemos então criar o Dashboard que irá exibir as informações e atrelar a variável a um gráfico, como por exemplo Gráfico de Linha.
 
O projeto deve possuir comunicação/controle via internet (TCP/IP) e uso do Protocolo MQTT:

Para tal, será construído a interface do sistema do sensor AD8232 com a placa ESP32, utilizando, portanto, esse sistema de prototipagem eletrônica.
A interface desses sistemas possibilitará que seja enviado o sinal ECG conectando os sensores em partes especificas do corpo. Para garantir que a informação seja conectada à internet, utilizaremos o protocolo MQTT via conta na Ubidots, seguindo os seguintes passos:
1.	Criar uma conta no Ubidots
2.	Criar um dispositivo e adicionar a variável (que será a placa ESP32)
3.	Criar um dashboard para monitoramento
4.	Adicionar o widget (o gráfico que será visualizado). Nesse caso poderá ser o de linha.

Após tais configurações, será necessário criar o código que fará com que a conexão entre os dispositivos seja feita, com a rede wifi disponibilizada, a senha da rede, o Token da Ubidots e o protocolo MQTT com o nome. Serão então criadas as variáveis e parâmetros necessários. Feito isso, os dispositivos poderão entrar em funcionamento e será possível checar os dados no dashboard criado no Ubidots. 


