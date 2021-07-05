# Joystick-servo-test
test code for joystick NRF servo control (will delete the temp file and replace with proper repo when done)
WARNING!!! CURRENT CODE VERSION IS "DIRTY" AND NEEDS OPTIMISATION AND PROPER COMMENTING! MAY OR MAY NOT WORK PROPERLY!
//code begins:
//TRANSMITTER MODULE:
//include all the necessary libraries
#include <Arduino.h> //arduino library when using VSCode PlatformIO
#include <SPI.h> //SPI library for Arduino
#include <nRF24L01.h> //nRF24 library for Arduino
#include <RF24.h> //RF24 library for Arduino
//==============PINOUT==============
/*
---Joystick---
A0 - x-axis, can be any ANALOG pin to read pot value
A1 - y-axis, can be any ANALOG pin to read pot value
D6 - click, can be analog or digital, works as push button
---don't forget the VCC and Gnd for the joystick, 5V arduino output is plenty---

---NRF24---
D11(MOSI) - MOSI on NRF24 module, ---CANNOT USE D11 pin for anything else now!---
D12(MISO) - MISO on NRF24 module, ---CANNOT USE D12 pin for anything else now!---
D13(SCK) - SCK on NRF24 module, ---CANNOT USE D13 pin for anything else now!---
D7 - CE on NRF24 module, can be any analog or digital pin
D10 - CS on NRF24 module, can be any digital pin
---don't forget the the VCC and Gnd for the NRF24 module, MUST BE 3.3V!!!
Can use the arduino 3V3 output. Recommend a 10uF cap across the module VCC and GND---
*/

#define joy1xinpin A0
#define joy1yinpin A1
#define joy1butinpin 6 

int joy1xval = 0;
int joy1yval = 0;
int joy1butval = 0;

RF24 TXmodule(7, 10); // CE, CSN
 
const byte address[6] = {"42"};
 
int Joy1readvals = 0;
 
void setup() 
{
  // put your setup code here, to run once:
Serial.begin(115200); //set to whatever speed you prefer, for serial debugging

pinMode(joy1xinpin, INPUT);
pinMode(joy1yinpin, INPUT);
pinMode(joy1butinpin, INPUT_PULLUP);

TXmodule.begin();
TXmodule.openWritingPipe(address);
TXmodule.setPALevel(RF24_PA_MIN);
TXmodule.stopListening();
}

void loop() 
{
  // put your main code here, to run repeatedly:
  joy1xval = analogRead(joy1xinpin);
  joy1yval = analogRead(joy1yinpin);
  joy1butval = digitalRead(joy1butinpin);

  int Joy1readvals[3];

  Joy1readvals[0] = map(joy1xval, 0, 1023, 179, 0);
  Joy1readvals[1] = map(joy1yval, 0, 1023, 0, 179);
  Joy1readvals[2] = map(joy1butval, 0, 1, 1, 0);
  
TXmodule.write(&Joy1readvals, sizeof(Joy1readvals));
 
  delay(5);

  Serial.print("Joy1x: ");
  Serial.print(Joy1readvals[0]);
  Serial.print(" Joy1y: ");
  Serial.print(Joy1readvals[1]);
  Serial.print(" Joy1butraw: ");
  Serial.print(joy1butval);
  Serial.print(" Joy1butmap: ");
  Serial.println(Joy1readvals[2]);
}

//=============================================RECEIVER MODULE============================
//include all the necessary libraries
#include <Arduino.h> //arduino library when using VSCode PlatformIO
#include <SPI.h> //SPI library for Arduino
#include <nRF24L01.h> //nRF24 library for Arduino
#include <RF24.h> //RF24 library for Arduino
#include <Servo.h>
//==============PINOUT==============
/*
---NRFreadvals and LED---
D8 - x-axis, can be any PWM pin to output servo angle
D9 - y-axis, can be any PWM pin to output servo angle
D2 - LED for output of button press

---don't forget the VCC and Gnd for the NRFreadvals, 
should be a separate power supply, with Gnd connected to arduino---

---NRF24---
D11(MOSI) - MOSI on NRF24 module, ---CANNOT USE D11 pin for anything else now!---
D12(MISO) - MISO on NRF24 module, ---CANNOT USE D12 pin for anything else now!---
D13(SCK) - SCK on NRF24 module, ---CANNOT USE D13 pin for anything else now!---
D7 - CE on NRF24 module, can be any analog or digital pin
D10 - CS on NRF24 module, can be any digital pin
---don't forget the the VCC and Gnd for the NRF24 module, MUST BE 3.3V!!! 
Can use the arduino 3V3 output. Recommend a 10uF cap  across the module VCC and GND---
*/
void SERVO();

Servo xservo;
Servo yservo;

RF24 RXmodule(7, 10); // CE, CSN
 
const byte address[6] = {"42"};

const int xservopin = 8;
const int yservopin = 9; 
const int ledoutpin = 2; 
int ledval = 0;

int NRFreadvals = 0;
 
void setup() 
{
// put your setup code here, to run once:
Serial.begin(115200); //set to whatever speed you prefer, for serial debugging
xservo.attach(xservopin);
yservo.attach(yservopin);
pinMode(ledoutpin, OUTPUT);
 
RXmodule.begin();
RXmodule.openReadingPipe(0, address);
RXmodule.setPALevel(RF24_PA_MIN);
RXmodule.startListening();
}

void loop() 
{
  // put your main code here, to run repeatedly:
delay(5);
if ( RXmodule.available()) {
SERVO();
  }
}



void SERVO(){
      while (RXmodule.available()) {
 
 int NRFreadvals[3];
 
      RXmodule.read(&NRFreadvals, sizeof(NRFreadvals));
      xservo.write(NRFreadvals[0]);
      yservo.write(NRFreadvals[1]);
      ledval = (NRFreadvals[2]);
      digitalWrite(ledoutpin, ledval);
      Serial.print("Xserval: ");
      Serial.print(NRFreadvals[0]);
      Serial.print(" Yserval: ");
      Serial.print(NRFreadvals[1]);
      Serial.print(" butval: ");
      Serial.println(NRFreadvals[2]);      
    }
}
