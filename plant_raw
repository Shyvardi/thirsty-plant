#include <SoftwareSerial.h>
#include <DFRobotDFPlayerMini.h>

SoftwareSerial mySerial(D5, D6); // RX, TX
DFRobotDFPlayerMini dfPlayer;

const int busyPin = D1; // BUSY pin from DFPlayer Mini
const int moisturePin = A0; // Analog pin connected to the capacitive sensor
const int ledPin = LED_BUILTIN; // Internal LED pin

const int dryValue = 780; // Value for completely dry
const int wetValue = 350; // Value for wet

const float moistureThreshold = 70.0; // Moisture percentage threshold for playing a file
const uint64_t sleepTime = 150000000; // Deep sleep time (10 minutes in microseconds)

void setup() {
  Serial.begin(9600);
  mySerial.begin(9600);

  pinMode(busyPin, INPUT); // Set BUSY pin as input
  pinMode(ledPin, OUTPUT); // Set LED pin as output

  if (!dfPlayer.begin(mySerial)) {
    Serial.println("DFPlayer Mini not detected.");
    while (true); // Stop execution if DFPlayer Mini is not detected
  }

  Serial.println("DFPlayer Mini ready.");
  dfPlayer.volume(30);  // Set volume level (0 to 30)

  digitalWrite(ledPin, HIGH); // Turn on the LED while active

  // Read the soil moisture sensor and check moisture level
  float moisturePercentage = getMoisturePercentage();
  Serial.print("Moisture Percentage: ");
  Serial.println(moisturePercentage);

  if (moisturePercentage < moistureThreshold) {
    // Play a random file and enter deep sleep
    playRandomFile();
  } else {
    Serial.println("Moisture level is too high. No file will be played.");
    digitalWrite(ledPin, LOW); // Turn off the LED before deep sleep
    delay(100); // Allow serial to flush
    ESP.deepSleep(sleepTime);
  }
}

void loop() {
  if (digitalRead(busyPin) == HIGH) {
    // If BUSY pin is HIGH, enter deep sleep mode
    Serial.println("File finished. Entering deep sleep mode...");
    digitalWrite(ledPin, LOW); // Ensure LED is off before entering deep sleep
    delay(100); // Allow serial to flush
    ESP.deepSleep(sleepTime);
  }
}

float getMoisturePercentage() {
  int moistureSum = 0;
  for (int i = 0; i < 5; i++) {
    moistureSum += analogRead(moisturePin);
    delay(200); // Small delay between readings (optional)
  }

  int moistureAverage = moistureSum / 5;
  return (float)(dryValue - moistureAverage) / (dryValue - wetValue) * 100;
}

void playRandomFile() {
  delay(1000);
  unsigned long t = millis();
  int randomFile = (t % 7) + 1; // Random number between 1 and 6
  dfPlayer.play(randomFile);   // Play the random file
  
  Serial.print("Playing file ");
  Serial.println(randomFile);

  while (digitalRead(busyPin) == LOW) {
    // Wait until the file is finished
    delay(100);
  }

  Serial.println("File finished.");
}
