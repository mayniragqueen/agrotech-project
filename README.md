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
At first we coneccted the two RY-GH PHOTOSYNTHETICALLY EFFECTIVE RADIATION SENSORS to the breadbord by using the Wires Conectors. then we connected both Sensors to the ADS1115 (A1\A0 connections) with their red Wires and wrote on top of the sensors Which one is A1 and which sensor is A2. The Sensors white wire was connected to the ADS1115 GND. Then we connected the ADS1115 to our ESP32 using the following connections:

![image](https://user-images.githubusercontent.com/106690258/178981331-0b54b38c-d5aa-462e-a72b-d49b5c27ed49.png)

## Our Circuit Diagram
the connections were made on the back of the board according to the schematics down below:
--ask Nadav/Guy to help us drow it

# Code & Thingspeak
## Preperation
in order to proceed with the code make sure the following libraries are installed:
write here the libraries

upload the next code to the ESP32 in order to see the DS18B20 code:
