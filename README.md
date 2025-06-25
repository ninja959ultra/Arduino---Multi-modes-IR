# Arduino---Multi-modes-IR


# Description:
This project is an Arduino-based multi-mode control system operated via an infrared (IR) remote. It features three distinct modes, each activated by a specific button on the remote. The system includes sensors and LEDs to indicate and respond to environmental inputs or manual commands.

---

Mode 1: Sound Detection Mode (Activated by Button 1)

Activates the sound sensor and its indicator LED.

When a sound exceeds a certain threshold:
The buzzer and assigned LED flash ON and OFF three times as an alert.

---

Mode 2: Light Detection Mode (Activated by Button 2)

Activates the light sensor and its indicator LED.

If the ambient light level drops below a set threshold:

The system triggers an alert:
The buzzer and assigned LED flash three times.

---

Mode 3: Manual Control Mode (Activated by Button 3)

Enables manual control using buttons 4 and 5:
Button 4: Turns ON the green LED and turns OFF the white LED.

Button 5: Turns ON the white LED and turns OFF the green LED.

---

This project demonstrates how to create a remote-controlled system that switches between sensor-driven modes and manual control, with clear LED and buzzer feedback for each interaction.



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
int brightness = A0;

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

  if (voiceValue > 535){
    alert(yellowLED);
  }

  Serial.println(voiceValue);
}


void brightnessMode(){
  byte LEDs[4] = {yellowLED, redLED, greenLED, whiteLED};
  turnOffLEDs(LEDs, 4);

  digitalWrite(blueLED, HIGH);

  brightnessValue = analogRead(brightness);

  if (brightnessValue < 250){
    alert(blueLED);
    }

  Serial.println(brightnessValue);
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
  pinMode(buzzer, OUTPUT);
  ir.enableIRIn();
}

void loop() {

  if (ir.decode()){
    signal = ir.decodedIRData.command;
    Serial.println(signal);

    ir.resume();
    delay(300);
  }

  if (signal == 69 && !(ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT)) {
    voiceStatus = true;
    brightnessStatus = false;
    manualStatus = false;
  }


  else if (signal == 70 && !(ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT)) {
    voiceStatus = false;
    brightnessStatus = true;
    manualStatus = false;
}

  else if (signal == 71 && !(ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT)) {
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
