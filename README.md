# Wookwi simulation
https://wokwi.com/projects/463733281356208129
# Smart-IoT-Based-Automatic-Air-Conditioner-System
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>

#define DHTPIN 2        // Digital pin connected to DHT22 sensor
#define PIRPIN 3        // PIR motion sensor pin
#define RELAYPIN 8      // Relay control pin

// Uncomment sensor type in use
//#define DHTTYPE    DHT11
#define DHTTYPE    DHT22
//#define DHTTYPE    DHT21

// Initialize DHT Unified Sensor
DHT_Unified dht(DHTPIN, DHTTYPE);

uint32_t delayMS;

void setup() {
  Serial.begin(9600);

  // Initialize sensors
  dht.begin();

  pinMode(PIRPIN, INPUT);
  pinMode(RELAYPIN, OUTPUT);

  digitalWrite(RELAYPIN, LOW); // Ensure AC is OFF at startup

  Serial.println(F("Smart AC Control System Started"));

  // Get sensor details (Temperature)
  sensor_t sensor;
  dht.temperature().getSensor(&sensor);

  Serial.println(F("------------------------------------"));
  Serial.println(F("Temperature Sensor Info"));
  Serial.print(F("Sensor Type: ")); Serial.println(sensor.name);
  Serial.print(F("Max Value: ")); Serial.print(sensor.max_value); Serial.println(F("°C"));
  Serial.print(F("Min Value: ")); Serial.print(sensor.min_value); Serial.println(F("°C"));
  Serial.print(F("Resolution: ")); Serial.print(sensor.resolution); Serial.println(F("°C"));
  Serial.println(F("------------------------------------"));

  // Set delay based on sensor capability
  delayMS = sensor.min_delay / 1000;
}

void loop() {

  // Wait between measurements
  delay(delayMS);

  // =====================
  // Read Temperature
  // =====================
  sensors_event_t event;
  dht.temperature().getEvent(&event);

  float temperature;

  if (isnan(event.temperature)) {
    Serial.println(F("Error reading temperature!"));
    return;
  } else {
    temperature = event.temperature;

    Serial.print(F("Temperature: "));
    Serial.print(temperature);
    Serial.println(F(" °C"));
  }

  // =====================
  // Read Motion Sensor
  // =====================
  int motion = digitalRead(PIRPIN);

  if (motion == LOW) {

    Serial.println(F("No motion detected"));
    Serial.println(F("AC OFF"));

    digitalWrite(RELAYPIN, LOW);
  }
  else {

    Serial.println(F("Motion detected"));

    // =====================
    // Smart Temperature Control
    // =====================
    if (temperature > 28) {

      Serial.println(F("Room is hot"));
      Serial.println(F("AC ON"));

      digitalWrite(RELAYPIN, HIGH);
    }
    else {

      Serial.println(F("Temperature normal"));
      Serial.println(F("AC OFF"));

      digitalWrite(RELAYPIN, LOW);
    }
  }

  Serial.println(F("------------------------------"));
}
