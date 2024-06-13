#include "HX711.h"

const int LOADCELL_DOUT_PIN = 2; // HX711 DOUT pin
const int LOADCELL_SCK_PIN = 3;  // HX711 SCK pin
const int BUZZER_PIN = 4;        // Buzzer control pin
const int TRIG_PIN = 5;          // Ultrasonic sensor Trig pin
const int ECHO_PIN = 6;          // Ultrasonic sensor Echo pin

HX711 scale;

unsigned long previousMillis = 0; // To store the last time the buzzer was started
unsigned long cycleInterval = 11000; // 1 minute + 1 second (in milliseconds)
unsigned long buzzerDuration = 1000; // 10 seconds (in milliseconds)

void setup() {
  Serial.begin(9600);
  pinMode(BUZZER_PIN, OUTPUT); // Set the buzzer pin as output
  pinMode(TRIG_PIN, OUTPUT);   // Set the ultrasonic sensor Trig pin as output
  pinMode(ECHO_PIN, INPUT);    // Set the ultrasonic sensor Echo pin as input
  scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN);
  scale.set_scale(1.0); // Set this to the calibration factor for your load cell
  scale.tare(); // Reset the scale to 0
}

void loop() {
  // Read the raw value from the load cell
  long rawValue = scale.read();

  // Convert the raw value to weight in grams
  // You may need to calibrate this conversion based on your load cell and setup
  float grams = rawValue / 1.0; // Change the calibration factor accordingly

  // Read the distance from the ultrasonic sensor
  float distance = getUltrasonicDistance();

  // Display the weight and distance on the serial monitor
  Serial.print("Weight: ");
  Serial.print(grams);
  Serial.println(" grams");
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Check if the weight is within the specified range (250000 to 270000 grams)
  // and the distance is less than 5.4 cm
  if (grams >= 250000 && grams <= 270000 && distance < 5.4) {
    // Buzzer should be OFF when the conditions are met
    noTone(BUZZER_PIN);
  } else {
    // Buzzer should be ON otherwise
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= cycleInterval) {
      previousMillis = currentMillis;
      // Start the buzzer for the specified duration
      tone(BUZZER_PIN, 1000); // You can change the frequency (1000 Hz) for a different tune
      delay(buzzerDuration);
      noTone(BUZZER_PIN);
    }
  }
}

// Function to measure the distance from the ultrasonic sensor
float getUltrasonicDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  long duration = pulseIn(ECHO_PIN, HIGH);
  // Calculate the distance in centimeters
  float distance_cm = duration * 0.034 / 2;
  return distance_cm;
}
