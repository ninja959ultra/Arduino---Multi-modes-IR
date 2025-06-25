# Arduino---Multi-modes-IR



# Code below:
```cpp
#include <IRremote.hpp>

IRrecv ir(2); // IR signal on pin 2

// LEDs and buzzer

byte yellowLED = 6;
byte blueLED = 7;
byte redLED = 8;
byte greenLED = 11;
byte whiteLED = 12;
byte buzzer = 10;

// analog values
int voice = A5;
int bightness = A0;

// global variables
int voiceValue;
int brightnessValue;
int signal;
byte count;

// modes status
bool brightnessStatus = false;
bool voiceStatus = false;
bool manualStatus = false;


void turnOffLEDs(byte LEDs[], byte size){

  for (byte i=0; i<size; i++){

  digitalWrite(LEDs[i], LOW);
  }

}

void alert(byte LED){

  byte count = 0;

  do{
    digitalWrite(LED, LOW);
    digitalWrite(buzzer, HIGH);    
    delay(300);
    digitalWrite(LED, HIGH);
    digitalWrite(buzzer, LOW);
    delay(300);
    count++;
  }

  while(count < 3);

}

void voiceMode(){
  byte LEDs[4] = {blueLED, redLED, greenLED, whiteLED};
  turnOffLEDs(LEDs, 4);

  digitalWrite(yellowLED, HIGH);

  voiceValue = analogRead(voice);

  if (voiceValue > 500){

    alert(yellowLED);

    }

}


void brightnessMode(){
  byte LEDs[4] = {yellowLED, redLED, greenLED, whiteLED};
  turnOffLEDs(LEDs, 4);

  digitalWrite(blueLED, HIGH);

  brightnessValue = analogRead(brightness);

  if (brightnessValue < 200){

    alert(blueLED);
    }

}

void manualMode(){
  byte LEDs[2] = {yellowLED, blueLED};
  turnOffLEDs(LEDs, 2);

  digitalWrite(redLED, HIGH);

  if (signal == 68) {  // button 4
    digitalWrite(greenLED, HIGH);
    digitalWrite(whiteLED, LOW);
    }


else if (signal == 64) {  // button 5
    digitalWrite(whiteLED, HIGH);
    digitalWrite(greenLED, LOW);
    }

  }

void (*modes[])() = {voiceMode, brightnessMode, manualMode};


void setup() {
  Serial.begin(9600);
  pinMode(yellowLED, OUTPUT);
  pinMode(blueLED, OUTPUT);
  pinMode(redLED, OUTPUT);
  pinMode(greenLED, OUTPUT);
  pinMode(whiteLED, OUTPUT);
  ir.enableIRIn();
}

void loop() {

  if (ir.decode()){
    signal = ir.decodedIRData.command;
    Serial.println(signal);

    ir.resume();
    delay(300);
  }

  if (signal == 69) {
    voiceStatus = true;
    brightnessStatus = false;
    manualStatus = false;
  }


  else if (signal == 70) {
    voiceStatus = false;
    brightnessStatus = true;
    manualStatus = false;
}

  else if (signal == 71) {
    voiceStatus = false;
    brightnessStatus = false;
    manualStatus = true;

}

  if (voiceStatus) {
    modes[0]();
  }


  if (brightnessStatus) {
    modes[1]();
  }

  if (manualStatus) {
    modes[2]();
  }

} 
```
