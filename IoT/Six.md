ESP8266 Arduino code – Experiment6.ino

#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <ESP8266HTTPClient.h>
#include <DHT.h>

// WiFi Configuration
const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

// Firebase Database URL
const char* FIREBASE_DATABASE_URL =
"https://YOUR-PROJECT-default-rtdb.firebaseio.com";

// DHT11 Configuration
#define DHTPIN D4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

// HTTPS Client
WiFiClientSecure client;

// Upload Interval
unsigned long lastSend = 0;
const unsigned long INTERVAL = 5000;

// WiFi Connection Function
void connectWiFi()
{
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

  Serial.print("Connecting to WiFi");

  unsigned long start = millis();

  while (WiFi.status() != WL_CONNECTED)
  {
    delay(250);
    Serial.print(".");

    if (millis() - start > 20000)
    {
      Serial.println("\nWiFi connection timeout!");
      ESP.restart();
    }
  }

  Serial.println("\nWiFi connected!");

  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

// Setup Function
void setup()
{
  Serial.begin(115200);
  delay(100);

  dht.begin();

  connectWiFi();

  // For practical testing
  client.setInsecure();

  Serial.println("Ready to send data to Firebase!");
}

// Main Loop
void loop()
{
  // Check WiFi connection
  if (WiFi.status() != WL_CONNECTED)
  {
    Serial.println("WiFi disconnected. Reconnecting...");
    connectWiFi();
  }

  // Send data every 5 seconds
  if (millis() - lastSend < INTERVAL)
  {
    return;
  }

  lastSend = millis();

  // Read temperature and humidity
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // Check sensor readings
  if (isnan(temperature) || isnan(humidity))
  {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Build JSON Payload
  String payload = "{";

  payload += "\"Temperature\":";
  payload += String(temperature, 2);

  payload += ",\"Humidity\":";
  payload += String(humidity, 2);

  payload += ",\"Timestamp\":";
  payload += String(millis());

  payload += "}";

  // Firebase REST API URL
  String url = String(FIREBASE_DATABASE_URL) + "/DHT11.json";

  HTTPClient http;

  // Start HTTPS Connection
  if (http.begin(client, url))
  {
    http.addHeader("Content-Type", "application/json");

    // Upload using HTTP PUT
    int httpCode = http.PUT(payload);

    if (httpCode > 0)
    {
      Serial.print("HTTP Response Code: ");
      Serial.println(httpCode);

      Serial.print("Firebase Response: ");
      Serial.println(http.getString());

      if (httpCode == 200)
      {
        Serial.println("Data uploaded successfully!");
      }
    }
    else
    {
      Serial.print("HTTP Error: ");
      Serial.println(http.errorToString(httpCode));
    }

    http.end();
  }
  else
  {
    Serial.println("Unable to connect to Firebase!");
  }

  // Display sensor readings
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" C");

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.println("--------------------------");
}


. HTML code – index.html

<!DOCTYPE html>
<html>

<head>
    <title>DHT11 Live Dashboard</title>

    <meta name="viewport"
          content="width=device-width, initial-scale=1.0">

    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background: #f2f2f2;
            padding: 30px;
        }

        h1 {
            color: #333;
        }

        .card {
            background: white;
            padding: 25px;
            margin: 20px auto;
            max-width: 350px;
            border-radius: 12px;
            box-shadow: 0 2px 10px #ccc;
        }

        .value {
            font-size: 30px;
            color: #007bff;
            font-weight: bold;
        }
    </style>

</head>

<body>

    <h1>DHT11 Live Dashboard</h1>

    <div class="card">
        <h2>Temperature</h2>
        <p class="value" id="temp">-- °C</p>
    </div>

    <div class="card">
        <h2>Humidity</h2>
        <p class="value" id="hum">-- %</p>
    </div>

    <div class="card">
        <h3>Last Update</h3>
        <p id="time">Waiting for data...</p>
    </div>

    <script type="module">

        // Import Firebase modules
        import { initializeApp }
        from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

        import { getDatabase, ref, onValue }
        from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";


        // Firebase Configuration
        const firebaseConfig = {

            apiKey: "YOUR_API_KEY",

            authDomain: "YOUR_PROJECT.firebaseapp.com",

            databaseURL: "YOUR_DATABASE_URL",

            projectId: "YOUR_PROJECT_ID",

            appId: "YOUR_APP_ID"

        };


        // Initialize Firebase
        const app = initializeApp(firebaseConfig);

        const db = getDatabase(app);


        // Reference to DHT11 Database Node
        const dhtRef = ref(db, "DHT11");


        // Listen for Real-Time Updates
        onValue(dhtRef, (snapshot) => {

            const data = snapshot.val();

            if (data) {

                document.getElementById("temp").innerText =
                    data.Temperature + " °C";

                document.getElementById("hum").innerText =
                    data.Humidity + " %";

                document.getElementById("time").innerText =
                    new Date().toLocaleTimeString();

            }

        }, (error) => {

            console.error("Firebase error:", error);

            document.getElementById("time").innerText =
                "Database access error";

        });

    </script>

</body>

</html>



/*
# **EXPERIMENT 6**

**Aim:** To connect ESP8266 NodeMCU with a DHT11 sensor and push live temperature and humidity data to Firebase Realtime Database, then display it on a web dashboard.

## **Procedure**

### **Step 1: Hardware Connections**

1. Connect the DHT11 sensor to the ESP8266 NodeMCU.
2. Make the following connections:

- DHT11 VCC → NodeMCU 3.3V
- DHT11 GND → NodeMCU GND
- DHT11 DATA → NodeMCU D4 (GPIO2)

### **Step 2: Setup Firebase**

1. Open [https://console.firebase.google.com/](https://console.firebase.google.com/?utm_source=chatgpt.com).
2. Create a new Firebase project.
3. Open the project and click on the Web App (`</>`) icon.
4. Enter an app nickname and register the Web App.
5. Go to **Build → Realtime Database**.
6. Click on **Create Database**.
7. Select the database location.
8. Select **Start in Test Mode** and create the database.
9. Copy the Firebase Realtime Database URL.
10. Go to **Project Settings → General → Your Apps** and copy the Firebase SDK configuration details:
    - API Key
    - Auth Domain
    - Database URL
    - Project ID
    - App ID

### **Step 3: Configure Arduino IDE**

1. Open Arduino IDE.
2. Install the ESP8266 board package if it is not already installed.
3. Install the following libraries using Library Manager:
   - ESP8266WiFi
   - DHT Sensor Library by Adafruit
   - Adafruit Unified Sensor
4. Select **NodeMCU 1.0 (ESP-12E Module)** from Tools → Board.
5. Select the appropriate COM port.
6. Set the Serial Monitor baud rate to 115200.

### **Step 4: Write and Upload ESP8266 Code**

1. Create a new sketch in Arduino IDE.
2. Include the required header files:

- `ESP8266WiFi.h`
- `WiFiClientSecure.h`
- `ESP8266HTTPClient.h`
- `DHT.h`

3. Enter the Wi-Fi SSID and password in the program.
4. Enter the Firebase Realtime Database URL.
5. Define the DHT11 data pin as D4 and sensor type as DHT11.
6. Initialize the DHT11 sensor using `dht.begin()`.
7. Connect ESP8266 to Wi-Fi using `WiFi.begin()`.
8. Read temperature using `dht.readTemperature()` and humidity using `dht.readHumidity()`.
9. Create a JSON payload containing temperature, humidity and timestamp.
10. Construct the Firebase REST API URL using `/DHT11.json`.
11. Use the HTTP PUT method to upload the sensor readings to Firebase.
12. Compile and upload the program to ESP8266.
13. Open the Serial Monitor and verify the Wi-Fi connection and successful Firebase upload.

### **Step 5: Create the Web Dashboard**

1. Open the project folder in Visual Studio Code.
2. Create a new file named `index.html`.
3. Write the HTML code to create a DHT11 Live Dashboard.
4. Add the Firebase SDK configuration details to the HTML file.
5. Import the Firebase modules:
   - `initializeApp`
   - `getDatabase`
   - `ref`
   - `onValue`
6. Initialize Firebase using `initializeApp(firebaseConfig)`.
7. Connect to Firebase Realtime Database using `getDatabase(app)`.
8. Access the DHT11 database node using `ref(db, "DHT11")`.
9. Use the `onValue()` function to retrieve real-time temperature and humidity values.
10. Display temperature and humidity on the webpage using HTML elements.

### **Step 6: Run and Test the Dashboard**

1. Install the Live Server extension in Visual Studio Code.
2. Right-click on `index.html`.
3. Select **Open with Live Server**.
4. The dashboard will open in the web browser.
5. Check whether temperature and humidity values are displayed.
6. Verify that the values update automatically when new readings are uploaded to Firebase.
7. Confirm the complete data flow:

**DHT11 Sensor → ESP8266 → Firebase Realtime Database → Local HTML Dashboard.**

## **Required code files**

| File              | Purpose                                         |
| ----------------- | ----------------------------------------------- |
| `Experiment6.ino` | Reads DHT11 data and uploads it to Firebase     |
| `index.html`      | Displays live temperature and humidity readings |

## **Expected Output**

1. ESP8266 connects to Wi-Fi successfully.
2. Temperature and humidity readings are displayed on the Serial Monitor.
3. Sensor data is uploaded to Firebase Realtime Database every 5 seconds.
4. Firebase stores the readings under the `DHT11` node.
5. The web dashboard displays temperature and humidity values in real time.


5. Final execution order
1. Create Firebase project.
2. Register Firebase Web App.
3. Create Realtime Database in Test Mode.
4. Copy Database URL and Firebase configuration.
5. Install required Arduino libraries.
6. Connect DHT11 to ESP8266 using D4.
7. Open Arduino IDE and paste Experiment6.ino.
8. Enter Wi-Fi credentials and Firebase Database URL.
9. Select NodeMCU 1.0 board and correct COM port.
10. Upload the code.
11. Open Serial Monitor at 115200 baud.
12. Verify that sensor readings are uploaded successfully.
13. Open VS Code and create index.html.
14. Paste the HTML code and enter Firebase configuration.
15. Install Live Server extension.
16. Right-click index.html and select Open with Live Server.
17. Verify that temperature and humidity appear on the dashboard.
18. Confirm that readings update automatically every 5 seconds.



## **Result**

Successfully interfaced the DHT11 temperature and humidity sensor with ESP8266 NodeMCU, uploaded sensor readings to Firebase Realtime Database using REST API, and displayed live temperature and humidity data on a web dashboard.

**Important:** Keep Firebase Test Mode enabled during the lab demonstration and ensure its rules permit reading and writing. Do not leave unrestricted test rules enabled for a public deployment.
*/
