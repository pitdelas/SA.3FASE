//exemplo base
//referência: https://raw.githubusercontent.com/vinicius0082/esp32-tagoIO/main/exemplos/mpu_esp32_mqtt_tagoIO


//Envio de Dados para tagoIO via MQTT

#include <ArduinoJson.h>
#include "EspMQTTClient.h"

//pinos de entrada e saída
const int trigPin = 27;
const int echoPin = 26;
const int LedPin = 32;

//variáveis para Json
char json_andares[100];

//variáveis internas
#define SOUND_SPEED 0.034
long duration;
float distanceCm;
int valor_andares;
int espaco1 = 0;
int espaco2 = 0;

int valor1 = 0;
int valor2 = 0;


//configurações da conexão MQTT
EspMQTTClient client
(
"FIESC_IOT", //nome da sua rede Wi-Fi
"C6qnM4ag81", //senha da sua rede Wi-Fi
"mqtt.tago.io", // MQTT Broker server ip padrão da tago
"Token", // username
"67f1e979-a316-4e8f-984e-89947258b1da", // Código do Token
"TestClient", // Client name that uniquely identify your device
1883 // The MQTT port, default to 1883. this line can be omitted
);

//EspMQTTClient client("FIESC_IOT", "C6qnM4ag81", "mqtt.tago.io", "Token", "a69fcc1e-31f3-4d0c-9fbe-cf23c920ec6b", "TestClient", 1883 );

//configuração dos pinos
void setup(){
Serial.begin(115200);
pinMode(trigPin, OUTPUT);
pinMode(echoPin, INPUT);
pinMode(LedPin, OUTPUT);
}

void leitura_sinais()
{
digitalWrite(trigPin, LOW);
delayMicroseconds(2);
digitalWrite(trigPin, HIGH);
delayMicroseconds(10);
digitalWrite(trigPin, LOW);

duration = pulseIn(echoPin, HIGH);

distanceCm = duration * SOUND_SPEED/2;

if(distanceCm < 19 && distanceCm > 10){
Serial.print("Distance (cm): ");
Serial.println(distanceCm);
digitalWrite(LedPin, HIGH);
andares = 1;
espaco1 = 1;
espaco2 = 1;
if(espaco1 == 0){
Serial.print("O espaço 1 está vazio! \n");
}else{
Serial.print("O espaço 1 está ocupado! \n");
Serial.print("Andares = 1 \n");
}

if(espaco2 == 0){
Serial.print("O espaço 2 está vazio! \n");
}else{
Serial.print("O espaço 2 está ocupado! \n");
Serial.print("Andares = 1 \n");
}

}else if(distanceCm <= 10 && distanceCm > 6){
Serial.print("Distance (cm): ");
Serial.println(distanceCm);
digitalWrite(LedPin, HIGH);
andares = 2;
espaco1 = 1;
espaco2 = 1;
if(espaco1 == 0){
Serial.print("O espaço 1 está vazio! \n");
}else{
Serial.print("O espaço 1 está ocupado! \n");
Serial.print("Andares = 2 \n");
}

if(espaco2 == 0){
Serial.print("O espaço 2 está vazio! \n");
}else{
Serial.print("O espaço 2 está ocupado! \n");
Serial.print("Andares = 2 \n");
}

}else if(distanceCm <= 6 && distanceCm > 0){
Serial.print("Distance (cm): ");
Serial.println(distanceCm);
digitalWrite(LedPin, HIGH);
andares = 3;
espaco1 = 1;
espaco2 = 1;
if(espaco1 == 0){
Serial.print("O espaço 1 está vazio! \n");
}else{
Serial.print("O espaço 1 está ocupado! \n");
Serial.print("Andares = 3 \n");
}

if(espaco2 == 0){
Serial.print("O espaço 2 está vazio! \n");
}else{
Serial.print("O espaço 2 está ocupado! \n");
Serial.print("Andares = 3 \n");
}

}else{
Serial.print("Distance (cm): ");
Serial.println(distanceCm);
digitalWrite(LedPin, LOW);
andares = 0;
espaco1 = 0;
espaco2 = 0;
if(espaco1 == 0){
Serial.print("O espaço 1 está vazio! \n");
}
if(espaco2 == 0){
Serial.print("O espaço 2 está vazio! \n");
}
}

valor_andares = andares;

}

void converte_json()
{
StaticJsonDocument<300> sjson_andares;

...............................................................................................................................................................................................
sjson_andares["variable"] = "andares";
sjson_andares["value"] = valor_andares;
serializeJson(sjson_andares, json_andares);
}

void envia_msg()
{
client.publish("/andares", json_andares);
}

//loop do programa
void loop()
{
leitura_sinais();
converte_json();
envia_msg();

delay(400);

client.loop();
}

void processa_msg(const String payload)
{
StaticJsonDocument<300> msg;

DeserializationError err = deserializeJson(msg, payload);
if (err) {
Serial.print(F("deserializeJson() failed with code "));
Serial.println(err.f_str());
}
Serial.print("var:");
String var = msg["variable"];
Serial.println(var);
if(var == "status")
{
Serial.print("value:");
String val = msg["value"];
Serial.println(val);
if(val == "OFF")
digitalWrite(LedPin, LOW);
else
digitalWrite(LedPin, HIGH);
}

}

// This function is called once everything is connected (Wifi and MQTT)
// WARNING : YOU MUST IMPLEMENT IT IF YOU USE EspMQTTClient
void onConnectionEstablished()
{
client.subscribe("/envia", [] (const String &payload) {
Serial.println(payload);
processa_msg(payload);
});
}
