# global
PROGRAM 1

1A
void setup() {
  pinMode(2,OUTPUT);
}

void loop() {
  digitalWrite(2,HIGH);
  delay(1000);
  digitalWrite(2,LOW);
  delay(1000);
}

1B
void setup() {
  pinMode(2,OUTPUT);
  pinMode(35,INPUT);
}

void loop() {
  if (digitalRead(35)==HIGH)
  {
    digitalWrite(2,HIGH);
  }
  else
  {
    digitalWrite(2,LOW);
  }
}


PROGRAM 2

void setup() {
  pinMode(2,OUTPUT);
  pinMode(35,INPUT);
}

void loop() {
  if (analogRead(36)<=800)
  {
    digitalWrite(2,HIGH);
  }
  else
  {
    digitalWrite(2,LOW);
  }
}


PROGRAM 3

#include "DHT.h"

#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  Serial.println(F("DHTxx test!"));
  dht.begin();
}

void loop() {
  delay(2000);

  float h = dht.readHumidity();
  float t = dht.readTemperature();
  float f = dht.readTemperature(true);

  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
  }

  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);

  Serial.print(F("Humidity: "));
  Serial.print(h);
  Serial.print(F("% Temperature: "));
  Serial.print(t);
  Serial.print(F("\xC2\xB0 C, "));
  Serial.print(f);
  Serial.print(F("\xC2\xB0 F, Heat index: "));
  Serial.print(hic);
  Serial.print(F("\xC2\xB0 C, "));
  Serial.print(hif);
  Serial.println(F("\xC2\xB0 F."));
}


PROGRAM 4

const int IR_PIN = 33;
const int BUZZER_PIN = 14;

bool INVERT_LOGIC = true;
const unsigned long beepMs = 200;
const unsigned long holdoff = 400;
int lastState = -1;

bool isDetected(int raw) {
  return INVERT_LOGIC ? (raw == LOW) : (raw == HIGH);
}

void setup() {
  Serial.begin(115200);
  pinMode(IR_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW);
  Serial.println("IR obstacle sensor ready.");
  Serial.println("Tip: Turn the blue potentiometer to set detection distance.");
  Serial.printf("Logic mode: %s (set INVERT_LOGIC=%s if messages look inverted)\n",
  INVERT_LOGIC ? "active-LOW" : "active-HIGH",
  INVERT_LOGIC ? "false" : "true");

  int r = digitalRead(IR_PIN);
  lastState = r;
  Serial.printf("Initial raw=%d | detected=%s\n", r, isDetected(r) ? "YES" : "NO");
}

void loop() {
  int r = digitalRead(IR_PIN);

  if (r != lastState) {
    lastState = r;

    if (isDetected(r)) {
      Serial.println("Obstacle detected!");
      digitalWrite(BUZZER_PIN, HIGH);
      delay(beepMs);
      digitalWrite(BUZZER_PIN, LOW);
    }
    else {
      Serial.println("No obstacle.");
    }

    delay(holdoff);
  }
}


PROGRAM 5

#include <LiquidCrystal_I2C.h>

#define LCD_ADDR 0x27

LiquidCrystal_I2C lcd(LCD_ADDR, 16, 2);

String nameStr = "Your Name";
String semStr = "Sem: 5";
String deptStr = "Dept: CSE";
String colStr = "College: GAT";

void setup() {
  Serial.begin(115200);
  lcd.init();
  lcd.backlight();
  Serial.println("LCD ready");
}

void loop() {
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print(nameStr);
  lcd.setCursor(0,1);
  lcd.print(semStr);
  Serial.println(nameStr + " | " + semStr);
  delay(2000);

  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print(deptStr);
  lcd.setCursor(0,1);
  lcd.print(colStr);
  Serial.println(deptStr + " | " + colStr);
  delay(2000);
}


PROGRAM 6

#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int TRIG_PIN = 12;
const int ECHO_PIN = 35;

float usToCm(long us) {
  return (us * 0.0343f) / 2.0f;
}

void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0,0);
  lcd.print("Ultrasonic Ready");
  delay(800);
}

long readUS() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(4);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  return pulseIn(ECHO_PIN, HIGH, 25000UL);
}

void loop() {
  const int N = 5;
  long sum = 0;
  int valid = 0;

  for (int i=0; i<N; i++) {
    long d = readUS();

    if (d > 0) {
      sum += d;
      valid++;
    }

    delay(40);
  }

  lcd.clear();

  if (valid == 0) {
    lcd.setCursor(0,0);
    lcd.print("Out of range");
    lcd.setCursor(0,1);
    lcd.print("Try closer");
    Serial.println("No echo (timeout) — out of range");
  }
  else {
    float us = (float)sum / valid;
    float cm = usToCm(us);
    lcd.setCursor(0,0);
    lcd.print("Dist: ");
    lcd.print(cm, 1);
    lcd.print(" cm");
    Serial.printf("Duration(us)=%.1f Distance=%.2f cm\n", us, cm);
  }

  delay(300);
}


PROGRAM 7

7A
#include "WiFi.h"

void setup()
{
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(100);
  Serial.println("Setup done");
}

void loop()
{
  Serial.println("scan start");

  int n = WiFi.scanNetworks();

  Serial.println("scan done");

  if (n == 0) {
    Serial.println("no networks found");
  }
  else {
    Serial.print(n);
    Serial.println(" networks found");

    for (int i = 0; i < n; ++i) {
      Serial.print(i + 1);
      Serial.print(": ");
      Serial.print(WiFi.SSID(i));
      Serial.print(" (");
      Serial.print(WiFi.RSSI(i));
      Serial.print(")");
      Serial.println((WiFi.encryptionType(i) == WIFI_AUTH_OPEN)?" ":"*");
      delay(10);
    }
  }

  Serial.println("");
  delay(5000);
}

7B
#include <WiFi.h>

const char *ssid = "SSID_Name_your_Choice";
const char *password = " Your_Choice(min 6 character) ";

IPAddress local_IP(192,168,4,22);
IPAddress gateway(192,168,4,9);
IPAddress subnet(255,255,255,0);

void setup()
{
  Serial.begin(115200);
  Serial.println();
  Serial.print("Setting soft-AP configuration ... ");
  Serial.println(WiFi.softAPConfig(local_IP, gateway, subnet) ? "Ready" : "Failed!");

  Serial.print("Setting soft-AP ... ");
  Serial.println(WiFi.softAP(ssid,password) ? "Ready" : "Failed!");

  Serial.print("Soft-AP IP address = ");
  Serial.println(WiFi.softAPIP());
}

void loop() {
  Serial.print("[Server Connected] ");
  Serial.println(WiFi.softAPIP());
  delay(500);
}


PROGRAM 8

#include <WiFi.h>

char ssid[]="REPLACE_WITH_YOUR_SSID";
char password[]="REPLACE_WITH_YOUR_PASSWORD";

IPAddress ip;
IPAddress gateway;

void setup()
{
  Serial.begin(115200);

  Serial.print("Attempting to connect to Network named: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(300);
  }

  Serial.println("\nYou're connected to the network");

  while (WiFi.localIP() = INADDR_NONE) {
    Serial.print(".");
    delay(300);
  }

  ip=WiFi.localIP();
  Serial.println(ip);

  gateway=WiFi.gatewayIP();
  Serial.println("GATEWAY IP:");
  Serial.println(gateway);
}

void loop()
{
}


PROGRAM 9

9A
#include <WiFi.h>

const char* ssid = "Mahesh";
const char* password = "012345678";

WiFiServer server(80);

void setup()
{
  Serial.begin(115200);
  pinMode(2, OUTPUT);
  delay(10);

  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

int value = 0;

void loop(){
  WiFiClient client = server.available();

  if (client) {
    Serial.println("New Client.");
    String currentLine = "";

    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        Serial.write(c);

        if (c == '\n') {
          if (currentLine.length() == 0) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            client.print("Click <a href=\"/H\">here</a> to turn the LED on pin 2 on.<br>");
            client.print("Click <a href=\"/L\">here</a> to turn the LED on pin 2 off.<br>");

            client.println();
            break;
          }
          else {
            currentLine = "";
          }
        }
        else if (c != '\r') {
          currentLine += c;
        }

        if (currentLine.endsWith("GET /H")) {
          digitalWrite(2, HIGH);
        }

        if (currentLine.endsWith("GET /L")) {
          digitalWrite(2, LOW);
        }
      }
    }

    client.stop();
    Serial.println("Client Disconnected.");
  }
}

9B
#include <WiFi.h>
#include <WebServer.h>
#include "DHT.h"

#define DHTTYPE DHT11

const char* ssid = " REPLACE_WITH_YOUR_SSID ";
const char* password = " REPLACE_WITH_YOUR_PASSWORD";

WebServer server(80);

uint8_t DHTPin = 4;

DHT dht(DHTPin, DHTTYPE);

float Temperature;
float Humidity;

void setup() {
  Serial.begin(115200);
  delay(100);

  pinMode(DHTPin, INPUT);
  dht.begin();

  Serial.println("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected..!");
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP());

  server.on("/", handle_OnConnect);
  server.onNotFound(handle_NotFound);

  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handle_OnConnect() {
  Temperature = dht.readTemperature();
  Humidity = dht.readHumidity();

  server.send(200, "text/html", SendHTML(Temperature,Humidity));
}

void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
}

String SendHTML(float Temperaturestat,float Humiditystat){
  String ptr = "<!DOCTYPE html> <html>\n";

  ptr +="<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=no\">\n";
  ptr +="<title>ESP32 Weather Report</title>\n";
  ptr +="<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}\n";
  ptr +="body{margin-top: 50px;} h1 {color: #444444;margin: 50px auto 30px;}\n";
  ptr +="p {font-size: 24px;color: #444444;margin-bottom: 10px;}\n";
  ptr +="</style>\n";
  ptr +="</head>\n";
  ptr +="<body>\n";
  ptr +="<div id=\"webpage\">\n";
  ptr +="<h1>ESP32 Weather Report</h1>\n";

  ptr +="<p>Temperature: ";
  ptr +=(int)Temperaturestat;
  ptr +="\xC2\xB0 C</p>";

  ptr +="<p>Humidity: ";
  ptr +=(int)Humiditystat;
  ptr +="%</p>";

  ptr +="</div>\n";
  ptr +="</body>\n";
  ptr +="</html>\n";

  return ptr;
}
