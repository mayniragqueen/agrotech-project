# Agrotech Project 2022
Measuring PAR outside and inside the greenhouse


# Components

## Building the circuit

In order to build one measuring station we used the following parts:

1. 1X ESP32
2. 2X RY-GH PHOTOSYNTHETICALLY EFFECTIVE RADIATION SENSORS
4. 1X BreadBord
5. 1X ADS1115 - ADC Analog to Digital Converter 
6. 2X Two Wires Conectors
7. 8X small Wires (Male)
8. 1X Plastic Enclosure Electronic Box 

the finnished board looked like this:

![image](https://user-images.githubusercontent.com/106690258/178973247-748636ac-e6a3-4f68-8b36-018683267e40.png)


## Instructions
At first we coneccted the two RY-GH PHOTOSYNTHETICALLY EFFECTIVE RADIATION SENSORS to the breadbord by using the Wires Conectors. then we connected both Sensors to the ADS1115 (A1\A0 connections) with their red Wires and wrote on top of the sensors Which one is A1 and which sensor is A2. The Sensors white wires were connected to the ADS1115 GND. Then we connected the ADS1115 to our ESP32 using the following connections:

![image](https://user-images.githubusercontent.com/106690258/178981331-0b54b38c-d5aa-462e-a72b-d49b5c27ed49.png)

## Our Circuit Diagram
the connections were made on the the board according to the schematics down below:

![agrotech project](https://user-images.githubusercontent.com/106690258/179032479-2e69e00c-9ea2-48c6-b30d-cca44b3dfdd4.png)


# Code & Thingspeak
```C++
#include <Adafruit_ADS1X15.h>
#include <WiFi.h>
#include "ThingSpeak.h"

unsigned long myChannelNumber = 1769653;
const char * myWriteAPIKey = "KEAKCMOH5FRZUJAZ";
const char* ssid = "agrotech-lab-1"; // your wifi SSID name
const char* password = "1Afuna2Gezer" ;// wifi pasword
const char* server = "api.thingspeak.com";
WiFiClient client;

Adafruit_ADS1115 ads;  /* Use this for the 16-bit version */

void setup(void)
{
  Serial.begin(9600);
  Serial.println("Hello!");

  Serial.println("Getting single-ended readings from AIN0..3");
  Serial.println("ADC Range: +/- 6.144V (1 bit = 3mV/ADS1015, 0.1875mV/ADS1115)");

  // The ADC input range (or gain) can be changed via the following
  // functions, but be careful never to exceed VDD +0.3V max, or to
  // exceed the upper and lower limits if you adjust the input range!
  // Setting these values incorrectly may destroy your ADC!
  //                                                                ADS1015  ADS1115
  //                                                                -------  -------
  // ads.setGain(GAIN_TWOTHIRDS);  // 2/3x gain +/- 6.144V  1 bit = 3mV      0.1875mV (default)
  // ads.setGain(GAIN_ONE);        // 1x gain   +/- 4.096V  1 bit = 2mV      0.125mV
  // ads.setGain(GAIN_TWO);        // 2x gain   +/- 2.048V  1 bit = 1mV      0.0625mV
  // ads.setGain(GAIN_FOUR);       // 4x gain   +/- 1.024V  1 bit = 0.5mV    0.03125mV
  // ads.setGain(GAIN_EIGHT);      // 8x gain   +/- 0.512V  1 bit = 0.25mV   0.015625mV
   ads.setGain(GAIN_SIXTEEN);    // 16x gain  +/- 0.256V  1 bit = 0.125mV  0.0078125mV

  if (!ads.begin()) {
    Serial.println("Failed to initialize ADS.");
    while (1);
  }
WiFi.disconnect();
  delay(10);
  WiFi.begin(ssid, password);

  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

    ThingSpeak.begin(client);
 
  WiFi.begin(ssid, password);
  
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("NodeMcu connected to wifi...");
  Serial.println(ssid);
  Serial.println();
}


void loop(void)
{
  int16_t adc0, adc1;
  float volts0, volts1;

  adc0 = ads.readADC_SingleEnded(0);
  adc1 = ads.readADC_SingleEnded(1);
 // adc2 = ads.readADC_SingleEnded(2);
 // adc3 = ads.readADC_SingleEnded(3);

  volts0 = ads.computeVolts(adc0);
  volts1 = ads.computeVolts(adc1);
  
  int sum0=0;
  float average0 = 0;
  float par0;
  int sum1=0;
  float average1 = 0;
  float par1;
    for (int i = 0; i <= 6000; i++) {
    adc0 = ads.readADC_SingleEnded(0);
    sum0 += adc0;
    adc1 = ads.readADC_SingleEnded(1);
    sum1 += adc1;
    Serial.print(adc0);
    Serial.print(" ");
    Serial.print(adc1);
    Serial.print(" ");
    Serial.println(i);
    delay(10);
    }
  average0=sum0/6000;   //1 min 
  par0=average0*0.897;
  Serial.print("par0:");Serial.println(par0);
  average1=sum1/6000;   //1 min 
  par1=average1*1.017;
  Serial.print("par1:");Serial.println(par1);
  delay(10);
ThingSpeak.setField(2,par1);
ThingSpeak.setField(1,par0);
ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);     
 
  client.stop();
 
  // thingspeak needs minimum 15 sec delay between updates
  //delay(60000);
}
```

  
## Preperation
in order to proceed with the code make sure the following libraries are installed:
1. Adafruit_ADS1X15.h
2. WiFi.h
3. ThingSpeak.h
In addition in order to get the PAR from the sensor we needed to calaulate the factor we multipliered with using the next equation:



