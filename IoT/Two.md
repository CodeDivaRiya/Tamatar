
#include <DHT.h>
#include <ESP8266WiFi.h>

// DHT11 Configuration
#define DHTPIN 16
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

// WiFi Credentials
const char* ssid = "YOUR_WIFI_NAME";
const char* pass = "YOUR_WIFI_PASSWORD";

// ThingSpeak Configuration
String apiKey = "YOUR_WRITE_API_KEY";
const char* server = "api.thingspeak.com";

WiFiClient client;

void setup()
{
  Serial.begin(9600);
  delay(10);

  dht.begin();

  // Connect to WiFi
  Serial.println("Connecting to WiFi...");

  WiFi.begin(ssid, pass);

  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi connected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

void loop()
{
  // Read sensor values
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  // Check sensor readings
  if (isnan(h) || isnan(t))
  {
    Serial.println("Failed to read from DHT sensor!");
    delay(2000);
    return;
  }

  // Display readings on Serial Monitor
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.println(" Celsius");

  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.println("%");

  // Connect to ThingSpeak server
  if (client.connect(server, 80))
  {
    String postStr = "api_key=" + apiKey;

    postStr += "&field1=";
    postStr += String(t);

    postStr += "&field2=";
    postStr += String(h);

    postStr += "\r\n\r\n";

    // Send HTTP POST request
    client.print("POST /update HTTP/1.1\r\n");
    client.print("Host: api.thingspeak.com\r\n");
    client.print("Connection: close\r\n");
    client.print("Content-Type: application/x-www-form-urlencoded\r\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\r\n\r\n");

    client.print(postStr);

    Serial.println("Data sent to ThingSpeak!");

    // Read server response
    client.setTimeout(5000);
    String response = client.readString();

    Serial.println("Server Response:");
    Serial.println(response);

    client.stop();
  }
  else
  {
    Serial.println("Connection to ThingSpeak failed!");
  }

  Serial.println("Waiting...");

  // ThingSpeak update interval
  delay(30000);
}






/*
ThingSpeak requires a minimum 15-second interval delay(30000); }

Experiment 2: ThingSpeak Setup Procedure

Go to ThingSpeak.
Create an account or sign in using your MathWorks account.
Go to Channels → My Channels and click on New Channel.
Enter the channel name and basic details.
Enable Field 1 and name it Temperature.
Enable Field 2 and name it Humidity.
Click Save Channel to create the channel.
After creating the channel, open the API Keys tab.
Copy the Write API Key.
Open Arduino IDE and paste the Write API Key into the apiKey variable in your code.
Enter your Wi-Fi SSID and password in the code.
Upload the code to NodeMCU ESP8266.
Open the Serial Monitor to check the sensor readings and data transmission.
Go to your ThingSpeak channel and open the Private View tab to observe the temperature and humidity graphs.
*/
