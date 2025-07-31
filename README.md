/*
23143
Paulina Martínez
Electrónica digital 2
Sección 10
Laboratorio #2
*/
#include <Arduino.h>
#include<stdint.h>

//Definición de variables
#define LED_V 27
#define LED_R 26
#define LED_A 12
#define LED_B 14
#define botonmas  5//pull up interno
#define botonmenos 21 //pull down interno
#define cambiomodo 22 //pull down externo

//Variables globales
int contador = 0;
const int maximo = 3;
const int maximobinario = 15; 
const int minimo = 0;
bool modobinario = false;

//Antirrebote en boleanos 
unsigned long lastDebounce = 0;
const long tiempo = 100;
bool estadobotonmas = HIGH;
bool estadobotonmenos = LOW;
bool estadomodo = LOW;
bool lecturabotonmas;
bool lecturabotonmenos;
bool lecturamodo;

//Prototipo de funciones para aumentar y decrementar el contador y el encendido y apagado de leds
void leds();
void incrementar();
void decrementar();

void setup() {
  //Iniciamos comunicación serial

  Serial.begin(115200);
  //Definimos pines de leds como salidas
  pinMode(LED_V, OUTPUT);
  pinMode(LED_R, OUTPUT);
  pinMode(LED_A, OUTPUT);
  pinMode(LED_B, OUTPUT);

  //Definimos pin de botones como entrada
  pinMode(botonmas, INPUT_PULLUP);
  pinMode(botonmenos, INPUT_PULLDOWN);
  pinMode(cambiomodo, INPUT);

  //Inicializar estados de leds
  digitalWrite(LED_V, LOW);
  digitalWrite(LED_R, LOW);
  digitalWrite(LED_A, LOW);
  digitalWrite(LED_B, LOW);
}

void loop() {
  //Lectura de los botones
  lecturabotonmas = digitalRead(botonmas);
  lecturabotonmenos = digitalRead(botonmenos);
  lecturamodo = digitalRead(cambiomodo);
  
  //Condición antirrebote con millis para los botones
  unsigned long currentMillis = millis();
  if (currentMillis - lastDebounce >= tiempo){
    //Este flanco tiene lógica inversa por ser pull_up
    bool flancomas = (lecturabotonmas==LOW && estadobotonmas == HIGH);

    //Estos flancos usan lógica normal, al ser pull_down
    bool flancomenos = (lecturabotonmenos == HIGH && estadobotonmenos == LOW);
    bool flancomodo = (lecturamodo==LOW && estadomodo==HIGH);

    //Igualamos los estados para la toma de datos
    estadobotonmas = lecturabotonmas;
    estadobotonmenos = lecturabotonmenos;
    estadomodo = lecturamodo;
    lastDebounce = currentMillis;
    
    //Condición de incremento del contador (con antirrebote) si se apacha el botonmas
    if(flancomas){ 
      incrementar();
    }

    //Condición de decremento del contador (con antirrebote) si se apacha el botonmenos
    if (flancomenos){
       decrementar();
    }

    //Condición de cambio de modo al presionar cambiomodo
    if (flancomodo){
      modobinario = !modobinario;
      contador=0;
      leds();
    }
  }

}

//Definición de funciones
//Función de incremento del contador
void incrementar(){
  contador++;
  //Condición circular para el modo de décadas
  if (!modobinario && contador > maximo) contador = minimo;
  //Actualización de leds
  leds(); 
  //Condición circular para el modo binario
  if (modobinario && contador > maximobinario) contador = minimo;
  //Actualización de leds
  leds();
}

//Función de decremento del contados
void decrementar(){
  contador--;
  //Condición circular para el modo de décadas
  if (!modobinario && contador < minimo) contador = maximo;
  //Actualización de leds
  leds();
  //Condición circular para el modo binario
  if (modobinario && contador < minimo) contador = maximobinario;
  //Actualización de leds
  leds();
}

//Función de encendido y apagado de leds
void leds(){
  //Condición para el modo binario
  if (modobinario){
    digitalWrite(LED_A, bitRead(contador, 0)); //bitRead sirve para leer el valor de un bit específico
    digitalWrite(LED_V, bitRead(contador, 1)); //primero va la variable del que se lee y luego el número de bit
    digitalWrite(LED_B, bitRead(contador, 2)); //enciende o apaga los leds de acuerdo al valor binario del contador
    digitalWrite(LED_R, bitRead(contador, 3)); //va de mayor a menor bit
  }
  //Condición para el contador de décadas
  else{
    if (contador==0){
      digitalWrite (LED_A, HIGH);
      digitalWrite (LED_V, LOW);
      digitalWrite (LED_B, LOW);
      digitalWrite (LED_R, LOW);
    }
      if (contador==1){
      digitalWrite (LED_A, LOW);
      digitalWrite (LED_V, LOW);
      digitalWrite (LED_B, HIGH);
      digitalWrite (LED_R, LOW);
    }
      if (contador==2){
      digitalWrite (LED_A, LOW);
      digitalWrite (LED_V, HIGH);
      digitalWrite (LED_B, LOW);
      digitalWrite (LED_R, LOW);
    }
      if (contador==3){
      digitalWrite (LED_A, LOW);
      digitalWrite (LED_V, LOW);
      digitalWrite (LED_B, LOW);
      digitalWrite (LED_R, HIGH);
    }
  }
}
