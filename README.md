
#include <WiFi.h>
#include <WebSocketClient.h>

const char* ssid     = "Your SSID here";
const char* password = "Your Password here";


char path[] = "/echo";
char host[] = "demos.kaazing.com";
int pingCount = 0;

const int relayPin = 2;
String currState = "";
String pin_data = "";
String state = "";

WebSocketClient webSocketClient;

// Use WiFiClient class to create TCP connections
WiFiClient client;

void setup() {
  Serial.begin(115200);
  delay(10);

  // We start by connecting to a WiFi network

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
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  delay(5000);

  // Connect to the websocket server
  if (client.connect("demos.kaazing.com/echo", 80)) {
    Serial.println("Connected");
  } else {
    Serial.println("Connection failed.");
    while (1) {
      // Hang on failure
    }
  }

  // Handshake with the server
  webSocketClient.path = path;
  webSocketClient.host = host;
  if (webSocketClient.handshake(client)) {
    Serial.println("Handshake successful");
  } else {
    Serial.println("Handshake failed.");
    while (1) {
      // Hang on failure
    }
  }

}


void loop() {
  String data;

  if (client.connected()) {

    webSocketClient.getData(data);
    if (data.length() > 0) {
      Serial.print("Received data: ");
      Serial.println(data);
      if (data == "?" ) {
        Serial.println("Sending Status");
        if (pin_data == "1") {
          webSocketClient.sendData("\"qstate\":\"ON\"");
        } else {
          webSocketClient.sendData("\"qstate\":\"OFF\"");
        }
        //webSocketClient.sendData("ESP Status");
      } else if (data == "CMD:on") { //if command then execute
        Serial.println("Sending CMD");
        pinMode(2, OUTPUT);
        digitalWrite(2, HIGH);
        webSocketClient.sendData("\"cstate\":\"ON\"");

      } else if (data == "CMD:off") { //if command then execute
        Serial.println("Sending CMD");
        pinMode(2, OUTPUT);
        digitalWrite(2, LOW);
        webSocketClient.sendData("\"cstate\":\"OFF\"");

      } else if (data == "aack") { //if command then execute
        Serial.println("Received aack!");
      } else if (data == "cack") { //if command then execute
        Serial.println("Received cack!");
      } else if (data == "qack") { //if command then execute
        Serial.println("Received qack!");
      }

      else {
        Serial.println("ESP Command not recognised!");
      }

    }


    if (pingCount > 10) {
      pingCount = 0;

      //capture the value of digital pin 2, send it along
      pinMode(2, OUTPUT);
      pin_data = String(digitalRead(2));
      Serial.println("PinD - " + pin_data);

      if (pin_data == "1") {
        webSocketClient.sendData("\"astate\":\"ON\"");
      } else {
        webSocketClient.sendData("\"astate\":\"OFF\"");
      }

    }


  } else {
    Serial.println("Client disconnected.");
    while (1) {
      // Hang on disconnect.
    }
  }

  // wait to fully let the client disconnect
  pingCount += 1;
  Serial.println(pingCount);
  delay(1000);

}
