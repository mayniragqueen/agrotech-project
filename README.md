# Agrotech final Project 2022
Measuring PAR outside and inside the greenhouse. PAR = Photosynthetically Active Radiation, the wavelengths of light within the visible range of 400 to 700 nanometers that are critical for photosynthesis. Knowing the PAR values outside and inside the greenhouse can help us make important descisions regarding the way we grow our plants.

![image](https://user-images.githubusercontent.com/106690258/179044073-31b28b74-fd6d-4be6-820d-0743e409548e.png)


![image](https://user-images.githubusercontent.com/106690258/179367660-58478d76-ff7c-495f-a287-7b5efae438d7.png)

# Components
We used two different PAR sensors:
Both of them are measuring PAR but they have different sensitivities

![image](https://user-images.githubusercontent.com/106690258/179042812-0467437d-4fe9-498e-946a-37aa3475da90.png)
![image](https://user-images.githubusercontent.com/106690258/179042928-c1f48bd9-3414-4488-9897-b3e738eae899.png)

## Our circuit

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
the connections were made on the the board according to the schematics down below (the lamps represent the RY-GH PHOTOSYNTHETICALLY EFFECTIVE RADIATION SENSORS):

![Circuit Diagram](https://user-images.githubusercontent.com/106690258/179476648-ef51a8bd-d6bd-4170-aa72-505e59307643.png)


# Our Arduino code
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
 
}
```

  
## Preperation
Our code included the following libraries:
1. Adafruit_ADS1X15.h
2. WiFi.h
3. ThingSpeak.h

In addition in order to get the PAR value calibrated we needed to calaulate the factor we multiply with after using the GAIN_SIXSTEEN analog to digital method(refers to the ADC sensor)  using the next equations:

![image](https://user-images.githubusercontent.com/106690258/179539884-622d268e-ebca-414b-a621-056adaa483c9.png)

The number we calculated for the sensor with the 8.71 PAR sensetivity was multiplied with the average measurements per minute (we used average in order to reduce the noise we observed in the graphs).

# Data Analysis

<img width="401" alt="image" src="https://user-images.githubusercontent.com/106690258/179734729-2497261e-a5b7-4be0-89cb-ee0c3c5f6cf6.png">

## Our MATLAB code
```MATLAB
% Read PAR data past day from a ThingSpeak channel and 
% visualize hourly PAR using the AREA function. 
   

% Field 1 records Inside the greenhouse and Field 2 is outside. 
   
% Channel ID to read data from 
readChannelID = 1769653; 
   
% Channel Read API Key   
% If your channel is private, then enter the read API 
% Key between the '' below:   
readAPIKey = 'ROZLL6RP93CIW861'; 
   
% Read PAR data for the last 33 hours in a timetable, including 
% timestamps for each measurement 
PARData = thingSpeakRead(readChannelID, 'Fields', [1 2], 'NumMinutes', 8000,...
                         'ReadKey', readAPIKey, 'Outputformat', 'Timetable');
   
% Compute the hourly average of the data 
avePAR = retime(PARData, 'hourly', 'mean'); 
inPAR = avePAR.A0; 
outPAR = avePAR.A1; 
  
% Plot the averaged data as an area plot. 
area(avePAR.Timestamps,[inPAR, outPAR]);
xlabel('Time');
ylabel('Average PAR per Hour');
legend({'Inside','Outside'});
```


