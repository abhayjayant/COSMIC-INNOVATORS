# COSMIC-INNOVATORS
SPACE HABITAT ENVIRONMENTAL MONITORING SYSTEM
#include <DHT.h>
#include <LiquidCrystal.h>

// Pin definitions
#define DHTPIN 2           // Pin for DHT11 sensor
#define DHTTYPE DHT11      // DHT 11
#define PIRPIN 3           // Pin for PIR sensor
#define TRIGPIN 4          // Pin for Ultrasonic sensor (Trig)
#define ECHOPIN 5          // Pin for Ultrasonic sensor (Echo)
#define BUZZERPIN 6        // Pin for Buzzer
#define BUTTONPIN 7        // Pin for Self-locking button
#define REEDPIN 8          // Pin for Magnetic Reed switch
#define REDPIN 9           // Pin for RGB LED (Red)
#define GREENPIN 10        // Pin for RGB LED (Green)
#define BLUEPIN 11         // Pin for RGB LED (Blue)
#define RS 12              // Pin for LCD RS
#define E 13               // Pin for LCD E
#define LIGHTPIN A0        // Pin for Photoresistor

// Objects
DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal lcd(RS, E, 11, 10, 9, 8);

void setup() {
  // Initialize components
  pinMode(PIRPIN, INPUT);
  pinMode(TRIGPIN, OUTPUT);
  pinMode(ECHOPIN, INPUT);
  pinMode(BUZZERPIN, OUTPUT);
  pinMode(BUTTONPIN, INPUT_PULLUP);
  pinMode(REEDPIN, INPUT_PULLUP);
  pinMode(REDPIN, OUTPUT);
  pinMode(GREENPIN, OUTPUT);
  pinMode(BLUEPIN, OUTPUT);
  pinMode(LIGHTPIN, INPUT);
  
  lcd.begin(16, 2); // Initialize LCD with 16 columns and 2 rows
  dht.begin();
  
  lcd.print("Initializing...");
  delay(2000);
  lcd.clear();
}

void loop() {
  // Read sensors
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  int lightLevel = analogRead(LIGHTPIN);
  int pirState = digitalRead(PIRPIN);
  int reedState = digitalRead(REEDPIN);
  float distance = getDistance();

  // Display on LCD
  lcd.setCursor(0, 0);
  lcd.print("T:");
  lcd.print(t);
  lcd.print("C H:");
  lcd.print(h);
  lcd.print("%");

  lcd.setCursor(0, 1);
  lcd.print("L:");
  lcd.print(lightLevel);
  lcd.print(" D:");
  lcd.print(distance);

  // RGB LED for status indication
  if (t > 30 || h > 70 || lightLevel < 200 || distance < 10 || reedState == LOW) {
    digitalWrite(REDPIN, HIGH);
    digitalWrite(GREENPIN, LOW);
    digitalWrite(BLUEPIN, LOW);
    tone(BUZZERPIN, 1000); // Buzzer alarm
  } else {
    digitalWrite(REDPIN, LOW);
    digitalWrite(GREENPIN, HIGH);
    digitalWrite(BLUEPIN, LOW);
    noTone(BUZZERPIN); // Turn off buzzer
  }

  // Check for button press
  if (digitalRead(BUTTONPIN) == LOW) {
    digitalWrite(REDPIN, LOW);
    digitalWrite(GREENPIN, LOW);
    digitalWrite(BLUEPIN, HIGH);
    noTone(BUZZERPIN); // Turn off buzzer
  }

  delay(2000); // Update every 2 seconds
}

float getDistance() {
  // Ultrasonic sensor distance calculation
  digitalWrite(TRIGPIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIGPIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIGPIN, LOW);
  
  long duration = pulseIn(ECHOPIN, HIGH);
  float distance = (duration * 0.034) / 2;
  return distance;
}
