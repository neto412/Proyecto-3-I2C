# Proyecto-3-I2C
# TIVA C

//UNIVERSIDAD DEL VALLE DE GUATEMALA
//Electrónica Digital 2
//Sección 10
//Ernesto Chavez 21441


//Librerias TFT
#include <stdint.h>
#include <stdbool.h>

#include <SPI.h>
#include <SD.h>

#include <TM4C123GH6PM.h>

#include "inc/hw_ints.h"
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "driverlib/debug.h"
#include "driverlib/gpio.h"
#include "driverlib/interrupt.h"
#include "driverlib/rom_map.h"
#include "driverlib/rom.h"
#include "driverlib/sysctl.h"
#include "driverlib/timer.h"

//Librerias internas
#include "bitmaps.h"
#include "font.h"
#include "lcd_registers.h"

//Definir pines
#define LCD_RST PD_0 //Pantalla
#define LCD_DC PD_1 //Pantalla
#define LCD_CS PA_3 //Pantalla






//Prototipos de Funciones
void LCD_Init(void);
void LCD_Clear(unsigned int c);
void LCD_CMD(uint8_t cmd);
void LCD_DATA(uint8_t data);
void SetWindows(unsigned int x1, unsigned int y1, unsigned int x2, unsigned int y2);
void LCD_Bitmap(unsigned int x, unsigned int y, unsigned int width, unsigned int height, unsigned char bitmap[]);
void LCD_Print(String text, int x, int y, int fontSize, int color, int background);
void DisplayTempLevel(int temperatureC);

File myFile;


const int sw1Pin = PF_4; 
const int sw2Pin = PF_0; 
//int val = 0;
float temperatureC;

//Setup
void setup() {
  Serial.begin(115200);
  Serial2.begin(115200);
  pinMode(sw1Pin, INPUT_PULLUP);
  pinMode(sw2Pin, INPUT_PULLUP);


  SysCtlClockSet(SYSCTL_SYSDIV_2_5 | SYSCTL_USE_PLL | SYSCTL_OSC_MAIN | SYSCTL_XTAL_16MHZ);
  SPI.setModule(0);
 
  LCD_Init();
  LCD_Clear(0x00);


  //SD
  SysCtlClockSet(SYSCTL_SYSDIV_2_5 | SYSCTL_USE_PLL | SYSCTL_OSC_MAIN | SYSCTL_XTAL_16MHZ);
  SPI.setModule(0);
  Serial.print("Initializing SD card...");
  pinMode(29, OUTPUT); //Pin SD

  if (!SD.begin(29)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done.");
  delay(250);


}

//LOOP principal
void loop() {
  if (Serial2.available()) {
    String readString = Serial2.readStringUntil('\n');
    temperatureC = readString.toFloat();
    DisplayTempLevel(temperatureC);
    
    
    if (temperatureC < 23) {
      Serial.println("Valor de temperatura: ");
      Serial.println(temperatureC);
      Serial.println("hipotermia");
      
    } else if (temperatureC >= 23 && temperatureC < 26) {
      Serial.println("Valor de temperatura: " + String(temperatureC));
      Serial.println(temperatureC);
      Serial.println("Temp normal");
      
    } else if (temperatureC > 26) {
      Serial.println("Valor del temperatura: " + String(temperatureC));
      Serial.println(temperatureC);
      Serial.println("Fiebre");
      
    }  
  }
  if (digitalRead(sw1Pin) == LOW) {
    Serial2.write('A');
    delay(100);
}
if (digitalRead(sw2Pin) == LOW) {
    myFile = SD.open("test.txt", FILE_WRITE);

    // if the file opened okay, write to it:
    if (myFile) {
      //String cadena = String(tempC); // Convierte el float a una String
      Serial.println("Valor de temp");
      myFile.println("Valor de temp guardado: ");
      myFile.println(temperatureC);
      

      // close the file:
      myFile.close();
      Serial.println("Hecho.");
    } else {
      // if the file didn't open, print an error:
      Serial.println("error opening test.txt");
    }
    delay(250);}
}




//Funcion para inicializar la pantalla
void LCD_Init(void) {
  pinMode(LCD_RST, OUTPUT);
  pinMode(LCD_CS, OUTPUT);
  pinMode(LCD_DC, OUTPUT);
  //****************************************
  // Secuencia de Inicialización
  //****************************************
  digitalWrite(LCD_CS, HIGH);
  digitalWrite(LCD_DC, HIGH);
  digitalWrite(LCD_RST, HIGH);
  delay(5);
  digitalWrite(LCD_RST, LOW);
  delay(20);
  digitalWrite(LCD_RST, HIGH);
  delay(150);
  digitalWrite(LCD_CS, LOW);
  //****************************************
  LCD_CMD(0xE9);  // SETPANELRELATED
  LCD_DATA(0x20);
  //****************************************
  LCD_CMD(0x11); // Exit Sleep SLEEP OUT (SLPOUT)
  delay(100);
  //****************************************
  LCD_CMD(0xD1);    // (SETVCOM)
  LCD_DATA(0x00);
  LCD_DATA(0x71);
  LCD_DATA(0x19);
  //****************************************
  LCD_CMD(0xD0);   // (SETPOWER)
  LCD_DATA(0x07);
  LCD_DATA(0x01);
  LCD_DATA(0x08);
  //****************************************
  LCD_CMD(0x36);  // (MEMORYACCESS)
  LCD_DATA(0x40 | 0x80 | 0x20 | 0x08); // LCD_DATA(0x19);
  //****************************************
  LCD_CMD(0x3A); // Set_pixel_format (PIXELFORMAT)
  LCD_DATA(0x05); // color setings, 05h - 16bit pixel, 11h - 3bit pixel
  //****************************************
  LCD_CMD(0xC1);    // (POWERCONTROL2)
  LCD_DATA(0x10);
  LCD_DATA(0x10);
  LCD_DATA(0x02);
  LCD_DATA(0x02);
  //****************************************
  LCD_CMD(0xC0); // Set Default Gamma (POWERCONTROL1)
  LCD_DATA(0x00);
  LCD_DATA(0x35);
  LCD_DATA(0x00);
  LCD_DATA(0x00);
  LCD_DATA(0x01);
  LCD_DATA(0x02);
  //****************************************
  LCD_CMD(0xC5); // Set Frame Rate (VCOMCONTROL1)
  LCD_DATA(0x04); // 72Hz
  //****************************************
  LCD_CMD(0xD2); // Power Settings  (SETPWRNORMAL)
  LCD_DATA(0x01);
  LCD_DATA(0x44);
  //****************************************
  LCD_CMD(0xC8); //Set Gamma  (GAMMASET)
  LCD_DATA(0x04);
  LCD_DATA(0x67);
  LCD_DATA(0x35);
  LCD_DATA(0x04);
  LCD_DATA(0x08);
  LCD_DATA(0x06);
  LCD_DATA(0x24);
  LCD_DATA(0x01);
  LCD_DATA(0x37);
  LCD_DATA(0x40);
  LCD_DATA(0x03);
  LCD_DATA(0x10);
  LCD_DATA(0x08);
  LCD_DATA(0x80);
  LCD_DATA(0x00);
  //****************************************
  LCD_CMD(0x2A); // Set_column_address 320px (CASET)
  LCD_DATA(0x00);
  LCD_DATA(0x00);
  LCD_DATA(0x01);
  LCD_DATA(0x3F);
  //****************************************
  LCD_CMD(0x2B); // Set_page_address 480px (PASET)
  LCD_DATA(0x00);
  LCD_DATA(0x00);
  LCD_DATA(0x01);
  LCD_DATA(0xE0);
  LCD_CMD(0x29); //display on
  LCD_CMD(0x2C); //display on

  LCD_CMD(ILI9341_INVOFF); //Invert Off
  delay(120);
  LCD_CMD(ILI9341_SLPOUT);    //Exit Sleep
  delay(120);
  LCD_CMD(ILI9341_DISPON);    //Display on
  digitalWrite(LCD_CS, HIGH);
}

// Función para borrar la pantalla - parámetros (color)
//***************************************************************************************************************************************
void LCD_Clear(unsigned int c) {
  unsigned int x, y;
  LCD_CMD(0x02c); // write_memory_start
  digitalWrite(LCD_DC, HIGH);
  digitalWrite(LCD_CS, LOW);
  SetWindows(0, 0, 319, 239); // 479, 319);
  for (x = 0; x < 320; x++)
    for (y = 0; y < 240; y++) {
      LCD_DATA(c >> 8);
      LCD_DATA(c);
    }
  digitalWrite(LCD_CS, HIGH);
}

// Función para enviar comandos a la LCD - parámetro (comando)
//***************************************************************************************************************************************
void LCD_CMD(uint8_t cmd) {
  digitalWrite(LCD_DC, LOW);
  SPI.transfer(cmd);
}

void LCD_DATA(uint8_t data) {
  digitalWrite(LCD_DC, HIGH);
  SPI.transfer(data);
}

// Función para definir rango de direcciones de memoria con las cuales se trabajara (se define una ventana)
//***************************************************************************************************************************************
void SetWindows(unsigned int x1, unsigned int y1, unsigned int x2, unsigned int y2) {
  LCD_CMD(0x2a); // Set_column_address 4 parameters
  LCD_DATA(x1 >> 8);
  LCD_DATA(x1);
  LCD_DATA(x2 >> 8);
  LCD_DATA(x2);
  LCD_CMD(0x2b); // Set_page_address 4 parameters
  LCD_DATA(y1 >> 8);
  LCD_DATA(y1);
  LCD_DATA(y2 >> 8);
  LCD_DATA(y2);
  LCD_CMD(0x2c); // Write_memory_start
}

// Función para dibujar texto - parámetros ( texto, coordenada x, cordenada y, color, background)
//***************************************************************************************************************************************
void LCD_Print(String text, int x, int y, int fontSize, int color, int background) {
  int fontXSize ;
  int fontYSize ;

  if (fontSize == 1) {
    fontXSize = fontXSizeSmal ;
    fontYSize = fontYSizeSmal ;
  }
  if (fontSize == 2) {
    fontXSize = fontXSizeBig ;
    fontYSize = fontYSizeBig ;
  }

  char charInput ;
  int cLength = text.length();
  //Serial.println(cLength, DEC);
  int charDec ;
  int c ;
  int charHex ;
  char char_array[cLength + 1];
  text.toCharArray(char_array, cLength + 1) ;
  for (int i = 0; i < cLength ; i++) {
    charInput = char_array[i];
    //Serial.println(char_array[i]);
    charDec = int(charInput);
    digitalWrite(LCD_CS, LOW);
    SetWindows(x + (i * fontXSize), y, x + (i * fontXSize) + fontXSize - 1, y + fontYSize );
    long charHex1 ;
    for ( int n = 0 ; n < fontYSize ; n++ ) {
      if (fontSize == 1) {
        charHex1 = pgm_read_word_near(smallFont + ((charDec - 32) * fontYSize) + n);
      }
      if (fontSize == 2) {
        charHex1 = pgm_read_word_near(bigFont + ((charDec - 32) * fontYSize) + n);
      }
      for (int t = 1; t < fontXSize + 1 ; t++) {
        if (( charHex1 & (1 << (fontXSize - t))) > 0 ) {
          c = color ;
        } else {
          c = background ;
        }
        LCD_DATA(c >> 8);
        LCD_DATA(c);
      }
    }
    digitalWrite(LCD_CS, HIGH);
  }
}

//****************************************************
void LCD_Bitmap(unsigned int x, unsigned int y, unsigned int width, unsigned int height, unsigned char bitmap[]){
  LCD_CMD(0x002c);
  digitalWrite(LCD_DC, HIGH);
  digitalWrite(LCD_CS, LOW);

  unsigned int x2, y2;
  x2 = x+width;
  y2 = y+height;
  SetWindows(x, y, x2-1, y2-1);
  unsigned int k = 0;
  unsigned int i, j;

  for (int i = 0; i < width; i++) {
    for (int j = 0; j < height; j++) {
      LCD_DATA(bitmap[k]);
      LCD_DATA(bitmap[k+1]);
      k = k + 2;
  }
 }
 digitalWrite(LCD_CS, HIGH);
}
//***************************************************


// Función para mostrar el valor del alcoholímetro en la pantalla TFT
void DisplayTempLevel(float temperatureC) {
  String textToShow;
  String textToShow1;// Texto a mostrar en pantalla
  int textColor = 0xFFFF; // Color del texto en formato RGB565 (blanco)
  int bgColor = 0x0000; // Color de fondo en formato RGB565 (negro)

  // Evaluamos el valor del alcoholímetro y establecemos el mensaje a mostrar
  if (temperatureC < 23) {
    textToShow = "Hipotermia:";
    textToShow1 = int(temperatureC);
    LCD_Bitmap(150, 100, 44, 50, frio);
    delay(2000);
   
    
  } else if (temperatureC >= 23 && temperatureC < 26) {
    textToShow = "Temp normal:  ";
    textToShow1 = int(temperatureC);
    LCD_Bitmap(150, 100, 70, 69, like);
    
  } else if (temperatureC > 26) {
    textToShow = "Fiebre:";
    textToShow1 = int(temperatureC);
    LCD_Bitmap(150, 100, 70, 9, fuego);
    //textToShow1 = String(val);
  } 

  // Limpia una sección de la pantalla (opcional)
  // Puedes ajustar las coordenadas según necesites
  LCD_Clear(0x0000); // Color de fondo negro

  // Utilizamos la función LCD_Print para mostrar el texto en la pantalla
  // Puedes ajustar las coordenadas y el tamaño de fuente según necesites
  LCD_Print(textToShow, 0, 50, 2, textColor, bgColor);
  LCD_Print(textToShow1, 0, 80, 2, textColor, bgColor);



  }

