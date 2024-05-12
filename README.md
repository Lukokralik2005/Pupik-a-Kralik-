# Pupik-a-Kralik-
#include <Wire.h>
#include <BH1750.h>
#include <Adafruit_NeoPixel.h>
#include "DHT.h"

#define pinDHT 5
#define pinSoilMoisture A0
#define pinWaterPump 6
#define pinLightSensor A1
#define pinLED 2

#define numLEDs 8

#define soilMoistureThreshold 500
#define wateringInterval 8 * 3600000 // 8 hodin v milisekundách
#define waterAmountLow 100 // Nízké množství vody pro zavlažování
#define waterAmountHigh 300 // Vysoké množství vody pro zavlažování

#define DHTTYPE DHT11

DHT dht(pinDHT, DHTTYPE);
BH1750 lightSensor;

Adafruit_NeoPixel strip = Adafruit_NeoPixel(numLEDs, pinLED, NEO_GRB + NEO_KHZ800);

unsigned long lastWateringTime = 0;

void setup() {
  Serial.begin(9600);
  
  dht.begin();
  lightSensor.begin();
  
  pinMode(pinSoilMoisture, INPUT);
  pinMode(pinWaterPump, OUTPUT);
  pinMode(pinLightSensor, INPUT);
  
  strip.begin();
  strip.show(); // Initialize all pixels to 'off'
}

void loop() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  float soilMoisture = analogRead(pinSoilMoisture);
  float lightIntensity = lightSensor.readLightLevel();
  
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" °C");
  
  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");
  
  Serial.print("Soil Moisture: ");
  Serial.println(soilMoisture);
  
  Serial.print("Light Intensity: ");
  Serial.println(lightIntensity);
  
  // Předpověď počasí
  bool rainExpected = false; // Předpoklad, že bude pršet
  
  // Pokud je očekáváno, že bude pršet, přeskočte zavlažování
  if (rainExpected) {
    Serial.println("Rain expected, skipping watering.");
  } else {
    unsigned long currentTime = millis();
    if (currentTime - lastWateringTime >= wateringInterval) {
      int waterAmount = map(soilMoisture, 0, 1023, waterAmountHigh, waterAmountLow); // Mapování množství vody na základě vlhkosti půdy
      digitalWrite(pinWaterPump, HIGH); // Zapnout čerpadlo
      delay(waterAmount); // Zavlažování na základě mapovaného množství vody
      digitalWrite(pinWaterPump, LOW); // Vypnout čerpadlo
      lastWateringTime = currentTime; // Aktualizace času posledního zavlažování
    }
  }
  
  int brightness = map(lightIntensity, 0, 10000, 0, 255); // Mapování intenzity světla na jas
  for(int i = 0; i < numLEDs; i++) {
    strip.setPixelColor(i, strip.Color(brightness, brightness, brightness)); // Nastavení jasu pro všechny LED
  }
  strip.show();
  
  delay(5000); // Čekání 5 sekund před dalším čtením
}




