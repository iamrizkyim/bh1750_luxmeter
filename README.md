//#bh1750_luxmeter
//about arduino for bh1750 automatic lamp control with thinger.io (IoT)
#include <Wire.h>
#include <BH1750.h>
#include <ESP8266WiFi.h>
#include <ThingerESP8266.h>

#define LIGHT_SENSOR_SDA_PIN D2
#define LIGHT_SENSOR_SCL_PIN D1
#define RELAY_LIGHT_PIN D8

#define USERNAME "iamrizkyim"
#define DEVICE_ID "AL"
#define DEVICE_CREDENTIAL "fffx5R-HR!GQXwzp"

ThingerESP8266 thing(USERNAME, DEVICE_ID, DEVICE_CREDENTIAL);
BH1750 lightSensor;

bool lightStatus = false;
bool lightAuto = false;
bool lightManual = false;

void setup() {
  Serial.begin(115200);
  Wire.begin(LIGHT_SENSOR_SDA_PIN, LIGHT_SENSOR_SCL_PIN);
  lightSensor.begin();
  pinMode(RELAY_LIGHT_PIN, OUTPUT);
  digitalWrite(RELAY_LIGHT_PIN, LOW);
  lightStatus = false;

  thing.add_wifi("Xiaomi 13T", "bayardong");

  thing["lightIntensity"] >> [](pson& out) {
    out = lightSensor.readLightLevel();
  };
  thing["light"] << [](pson& in) {
    if (in.is_empty()) {
      if (!lightAuto && !lightManual) {
        in = lightStatus;
      }
    } else {
      lightStatus = in;
      digitalWrite(RELAY_LIGHT_PIN, (lightAuto || lightManual) ? (lightStatus ? HIGH : LOW) : LOW);
    }
  };
  thing["lightAuto"] << [](pson& in) {
    if (in.is_empty()) {
      in = lightAuto;
      digitalWrite(RELAY_LIGHT_PIN, (lightAuto || lightManual) ? (lightStatus ? HIGH : LOW) : LOW);
      if (!lightAuto) {
        lightStatus = false;
        digitalWrite(RELAY_LIGHT_PIN, HIGH);
      }
    } else {
      lightAuto = in;
      digitalWrite(RELAY_LIGHT_PIN, (lightAuto || lightManual) ? (lightStatus ? HIGH : LOW) : LOW);
      if (!lightAuto) {
        lightStatus = false;
        digitalWrite(RELAY_LIGHT_PIN, HIGH);
      }
    }
  };

  // Tombol manual on/off
  thing["lightManual"] << [](pson& in) {
    if (in.is_empty()) {
      in = lightManual;
    } else {
      lightManual = in;
      digitalWrite(RELAY_LIGHT_PIN, (lightAuto || lightManual) ? (lightStatus ? HIGH : LOW) : LOW);
    }
  };

  Serial.println("Program dimulai.");
}

void loop() {
  thing.handle();

  if (lightAuto) {
    float lightIntensity = lightSensor.readLightLevel();
    Serial.print("Intensitas cahaya saat ini: ");
    Serial.println(lightIntensity);

    if (lightIntensity > 50) {
      lightStatus = true;
    } else {
      lightStatus = false;
    }

    digitalWrite(RELAY_LIGHT_PIN, (lightAuto || lightManual) ? (lightStatus ? HIGH : LOW) : LOW);
  }

  delay(1000);
}
