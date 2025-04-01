# load-cell-sensor 

#include <Arduino.h>
#include <LiquidCrystal_I2C.h>
#include "HX711.h"
#include <SoftwareSerial.h>

#define RX 7
#define TX 6

SoftwareSerial mySerial(RX, TX);
// HX711 circuit wiring
const int LOADCELL_DOUT_PIN = 2;
const int LOADCELL_SCK_PIN = 3;
LiquidCrystal_I2C lcd(0x27, 16,2);

HX711 scale;

float calibration_factor = 110.807; // Adjust based on calibration
float reset_threshold = 2.0; // Reset if weight falls below this (adjustable)
int avg_readings = 10; // Number of readings for averaging

void setup() {
    Serial.begin(9600);
    //Serial.println("Initializing the scale...");
    // Start the software serial communication
    mySerial.begin(9600);

    scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN);
    lcd.init(); 
    lcd.backlight();
    // Wait until HX711 is ready
    while (!scale.is_ready()) {
        delay(500);
    }

    scale.set_scale(calibration_factor); // Apply calibration factor
    scale.tare(); // Reset to zero

   // Serial.println("Scale initialized. Ready to measure.");
}

void loop() {
    if (scale.is_ready()) {  
        int weight = scale.get_units(avg_readings);  // Get stable averaged reading
        // **Ignore garbage readings**
        if (weight < -1000 || weight > 10000) { 
            Serial.println("Invalid reading detected! Ignoring...");
            return;
        }
        if(weight < 1)
        {
          weight=0;
        }
          // Check if data is available on the software serial port
    if (mySerial.available()) {
        char weight = mySerial.read();
        Serial.println(weight);
    }
     // Check if data is available on the hardware serial port
    if (Serial.available()) {
        char  weight = Serial.read();
        mySerial.write(weight);
        Serial.println(weight);
    }
        lcd.setCursor(0, 0); // set cursor to first row
        lcd.clear();
        //lcd.print("Weight : "); // print out to LCD
        lcd.setCursor(0, 0); // set cursor to second row
        lcd.print(weight); 
        //lcd.print(" gm");


    delay(500);
}}
