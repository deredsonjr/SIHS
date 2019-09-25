# SIHS
Códigos da disciplina sistemas integrados de hardware e software
Código de implementação de led com botão de acesso remoto 



#include <ESP8266WiFi.h>
#include <PubSubClient.h>


const char* SSID = "Apuea";
const char* PASSWORD = "desemb2019"; 

#define ID_MQTT "aahh"
const char* BROKER_MQTT = "5.196.95.208"; //URL do broker MQTT que se deseja utilizar
int BROKER_PORT = 1883; // Porta do Broker MQTT

WiFiClient espClient;
PubSubClient MQTT(espClient);

//Definicao dos pinos utilizados
#define pinLED D5
#define pinBOTAO D6

//Topicos
#define TOPICO_SUBSCRIBE "sihs/top/LED"
#define TOPICO_PUBLISH "sihs/top/BUTTON"
void reconnect(){
  while(!MQTT.connected()){
    Serial.println("Tentando reconectar ao Broker MQTT");

    if(MQTT.connect(ID_MQTT)){
      Serial.println("Conexão com sucesso");
      MQTT.subscribe(TOPICO_SUBSCRIBE);
    }
    else{
      Serial.println("Falha ao reconectar no broker");
      delay(1000);
    }
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived:");
  Serial.print(topic);
  String s = "";
  for (int i=0;i<length;i++) {
    Serial.print((char)payload[i]);
    s += ((char)payload[i]);
  }
//Verifica conteudo
  Serial.println(s);
  if(s.equals("desligar")) {
    digitalWrite(pinLED,LOW);
  }
  else if(s.equals("ligar")) {
    digitalWrite(pinLED,HIGH);
  }
}


void setup() {
  Serial.println("Setup.");

  //iniciar MQTT
  MQTT.setServer(BROKER_MQTT, BROKER_PORT);
  MQTT.setCallback(callback);

  //Inicializa rede
  WiFi.begin(SSID,  PASSWORD);
  if (WiFi.status() != WL_CONNECTED) { //Tenta por DHCP
    Serial.println("Conexão falhou...");
  }
  
  //inicializa o pino do LED como saída
  pinMode(pinLED, OUTPUT);
  pinMode(pinBOTAO, INPUT);
  //Inicializa serial a 115200 bps
  Serial.begin(115200);
  Serial.print("Node MCU incializado.");
}


void loop(){
  //Conecta MQTT
  if (!MQTT.connected()) {
    reconnect(); //Caso caia a conexao
  }
  MQTT.loop(); //Deixa MQTT no loop

  if(digitalRead(pinBOTAO) == 0)
  {
     Serial.println("Botão não pressionado");
     MQTT.publish(TOPICO_PUBLISH, "Não Pressionado");
  }
  else
  {
     Serial.println("Botão pressionado");
     MQTT.publish(TOPICO_PUBLISH,"Pressionado");
  } 

  
  
  delay(1000);
}
