# Proyecto-3-I2C
# ESP32

//UNIVERSIDAD DEL VALLE DE GUATEMALA
//Electrónica Digital 2
//Sección 10
//Ernesto Chavez - 21441

//Librerías
#include <Arduino.h>
#include <Wire.h>  // Librería para comunicación I2C
#include <Temperature_LM75_Derived.h>
#include <Adafruit_NeoPixel.h>  

//Definiciones de variables
#define LM75_ADDR 0x48
unsigned long tiempo; 
#define PIN  19
#define NUMPIXELS  16
Adafruit_NeoPixel pixels(NUMPIXELS,PIN, NEO_GRB);
Generic_LM75 temperature;  // Objeto de la clase Generic_LM75
unsigned long lastPrintTime = 0;
bool buttonPressed = false;  // Variable para rastrear el estado del botón

//Funciones de colores del neopixel
void showBlue() {
    for(int i = 0; i < NUMPIXELS; i++) {
        pixels.setPixelColor(i, pixels.Color(0, 0, 255)); // Color azul
        pixels.show();
        delay(50);
        pixels.clear();
        pixels.show();
        delay(50);
    }
}

void showGreen() {
    for(int i = 0; i < NUMPIXELS; i++) {
        pixels.setPixelColor(i, pixels.Color(0, 255, 0)); // Color verde
        pixels.show();
        delay(500);
        pixels.clear();
        pixels.show();
        delay(500);}
}

void showRed() {
    for(int i = 0; i < NUMPIXELS; i++) {
        pixels.setPixelColor(i, pixels.Color(255, 0, 0)); // Color rojo
        pixels.show();
        delay(150);
        pixels.clear();
        pixels.show();
        delay(200);}
    }


// Setup
void setup() {
  Serial.begin(115200);
  Serial2.begin(115200);
  Wire.begin();  // Inicia comunicación I2C con los pines definidos

  pixels.begin(); 
  pixels.clear();
  pixels.setBrightness(255);

  
}

// Loop
void loop() {
  unsigned long currentTime = millis();
  tiempo = millis();
  // Detectar si la TIVA C envió la señal
  if (Serial2.available()) {
    char tivaSignal = Serial2.read();
    if (tivaSignal == 'A') {
      buttonPressed = true;
    }
  }

  // Procede con la lectura del sensor si el botón ha sido presionado
  if (buttonPressed) {
    float temperatureC = temperature.readTemperatureC();
    if (currentTime - lastPrintTime >= 1000) {
      lastPrintTime = currentTime;
      //float temperatureC = temperature.readTemperatureC();  // Lectura de temperatura

      // Asumiendo que -999 no es un valor válido de temperatura, lo usamos para detectar errores
      if (temperatureC != -999) {  
        Serial2.println(String(temperatureC));  // Enviar temperatura a la TIVA C

        // Interpretación de resultados de temperatura
        if (temperatureC < 23) {
          Serial.println("Hipotermia");
          Serial.println(temperatureC);
          showBlue();


        } else if (temperatureC >= 23 && temperatureC < 26) {
          Serial.println("Temperatura normal");
          Serial.println(temperatureC);
          showGreen();

        } else {
          Serial.println("Fiebre");
          Serial.println(temperatureC);
          showRed();
        }
      } 
  
  }
  buttonPressed = false; 
}
}
