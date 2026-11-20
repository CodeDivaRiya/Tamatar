
#include <DHT.h>

#define DHTPIN D3          // Data pin connected to D3
#define DHTTYPE DHT11      // Sensor type

DHT dht(DHTPIN, DHTTYPE);

void setup()
{
  Serial.begin(9600);
  dht.begin();
}

void loop()
{
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  // Check if sensor readings are valid
  if (isnan(h) || isnan(t))
  {
    Serial.println("Failed to read from DHT sensor!");
    delay(1000);
    return;
  }

  Serial.println("Humidity and Temperature");
  Serial.println("22/11/2026");
  Serial.println("Riya Pailwan");

  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.print("% ");

  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.println(" °C");

  Serial.println("-------------------------");

  delay(1000);
}
