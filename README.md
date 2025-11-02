# Live-Weather-Station-NodeMCU
#include <ESP8266WiFi.h>
#include <DHT.h>

// DHT sensor configuration
#define DHTPIN D4       // DHT11 data pin connected to NodeMCU D4 (GPIO2)
#define DHTTYPE DHT11   // Change to DHT22 if you are using that

DHT dht(DHTPIN, DHTTYPE);

// Wi-Fi credentials
const char* ssid = "Your_WiFi_Name";
const char* password = "Your_WiFi_Password";

// Create a web server on port 80
WiFiServer server(80);

void setup() {
  Serial.begin(115200);
  delay(10);

  dht.begin();

  // Connect to Wi-Fi
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("NodeMCU IP address: ");
  Serial.println(WiFi.localIP());

  // Start the web server
  server.begin();
  Serial.println("Server started");
}

void loop() {
  WiFiClient client = server.available();
  if (!client) {
    return;
  }

  Serial.println("New client connected");
  String request = client.readStringUntil('\r');
  client.flush();

  float h = dht.readHumidity();
  float t = dht.readTemperature();

  // HTML page
  String html = "<!DOCTYPE html><html>";
  html += "<head><meta http-equiv='refresh' content='5'/><title>Weather Station</title></head>";
  html += "<body style='text-align:center; font-family:Arial'>";
  html += "<h1>🌦 Live Weather Station</h1>";
  html += "<p><b>Temperature:</b> " + String(t) + " °C</p>";
  html += "<p><b>Humidity:</b> " + String(h) + " %</p>";
  html += "<p>Updated every 5 seconds</p>";
  html += "</body></html>";

  client.print(html);
  delay(100);
  Serial.println("Client disconnected");
}
