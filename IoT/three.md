
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

// WiFi Credentials
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

// Create Web Server on Port 80
ESP8266WebServer server(80);

// Function to handle webpage request
void handleRoot()
{
  int gasValue = analogRead(A0);

  String html = "<!DOCTYPE html>";
  html += "<html>";
  html += "<head>";
  html += "<meta name='viewport' content='width=device-width, initial-scale=1.0'>";
  html += "<title>Gas Sensor Monitor</title>";
  html += "</head>";

  html += "<body>";
  html += "<h2>Gas Sensor Monitor</h2>";
  html += "<p>Raw Gas Level: ";
  html += String(gasValue);
  html += "</p>";
  html += "</body>";
  html += "</html>";

  server.send(200, "text/html", html);
}

void setup()
{
  Serial.begin(115200);

  // Connect to WiFi
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi Connected!");

  // Display IP Address
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  // Configure Web Server
  server.on("/", handleRoot);

  // Start Server
  server.begin();

  Serial.println("HTTP server started");
}

void loop()
{
  server.handleClient();
}






/*
Steps:
1. Upload the code.
2. Open Serial Monitor at 115200 baud.
3. Note the IP address displayed.
4. Connect your mobile to the same Wi-Fi network.
5. Open Chrome.
6. Enter the IP address (example: 192.168.1.8).
7. You will see the Gas Sensor Monitor webpage with the raw gas level.
Refresh the webpage to obtain another reading.
*/
