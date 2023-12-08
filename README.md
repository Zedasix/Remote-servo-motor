# Remote-servo-motor

``` /*
 * WebSocketServer.ino
 *
 *  Created on: 22.05.2015
 *
 */

#include <Arduino.h>

#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>
#include <WebSocketsServer.h>
#include <Hash.h>

ESP8266WiFiMulti WiFiMulti;

WebSocketsServer webSocket = WebSocketsServer(81);

const char* ssid = "test-WiFi";
const char* password = "test1234";
IPAddress ip(192, 168, 1, 44);
IPAddress gateway(192, 168, 1, 254);
IPAddress subnet(255, 255, 255, 0);
const int analogInPin = A0;  // ESP8266 Analog Pin ADC0 = A0
int sensorValue = 0;  // value read from the pot
String potVal;

#define USE_SERIAL Serial
void webSocketEvent(uint8_t num, WStype_t type, uint8_t * payload, size_t length) {

    switch(type) {
        case WStype_DISCONNECTED:
            USE_SERIAL.printf("[%u] Disconnected!\n", num);
            break;
        case WStype_CONNECTED:
            {
                IPAddress ip = webSocket.remoteIP(num);
                USE_SERIAL.printf("[%u] Connected from %d.%d.%d.%d url: %s\n", num, ip[0], ip[1], ip[2], ip[3], payload);
				
				// send message to client
				webSocket.sendTXT(num, "Connected");
            }
            break;
        case WStype_TEXT:
            USE_SERIAL.printf("[%u] get Text: %s\n", num, payload);

            // send message to client
             webSocket.sendTXT(num, "potVal");

            // send data to all connected clients
            // webSocket.broadcastTXT("message here");
            break;
        case WStype_BIN:
            USE_SERIAL.printf("[%u] get binary length: %u\n", num, length);
            hexdump(payload, length);

            // send message to client
            // webSocket.sendBIN(num, payload, length);
            break;
    }

}

void setup() {
    // USE_SERIAL.begin(921600);
    USE_SERIAL.begin(115200);

    //Serial.setDebugOutput(true);
    USE_SERIAL.setDebugOutput(true);

    USE_SERIAL.println();
    USE_SERIAL.println();
    USE_SERIAL.println();

 	Serial.println(F("Initialize System"));
 	//Init ESP8266 Wifi
 	WiFi.config(ip, gateway, subnet); 						// forces to use the fix IP
 	WiFi.begin(ssid, password);
 	while (WiFi.status() != WL_CONNECTED) {
 			delay(500);
 			Serial.print(F("."));
 	} 	Serial.print(F("connected to Wifi! IP address : http://")); Serial.println(WiFi.localIP()); // Print the IP address

    for(uint8_t t = 4; t > 0; t--) {
        USE_SERIAL.printf("[SETUP] BOOT WAIT %d...\n", t);
        USE_SERIAL.flush();
        delay(1000);
    }

    /*WiFiMulti.addAP("test-WiFi", "test1234");

    while(WiFiMulti.run() != WL_CONNECTED) {
        delay(100);
    }
*/
    webSocket.begin();
    webSocket.onEvent(webSocketEvent);
}

void loop() {
    webSocket.loop();
           sensorValue = analogRead(A0);
      
       Serial.println(sensorValue);
       String valeur=String(sensorValue);
webSocket.broadcastTXT(valeur);



  //  webSocket.sendTXT(num, potVal);


} ```

#Code pour le servo moteur

``` #include <Arduino.h>

#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>

#include <WebSocketsClient.h>

#include <Hash.h>

ESP8266WiFiMulti WiFiMulti;
WebSocketsClient webSocket;

const char* ssid = "test-WiFi";
const char* password = "test1234";
IPAddress ip(192, 168, 1, 44);
IPAddress gateway(192, 168, 1, 254);
IPAddress subnet(255, 255, 255, 0);

#define USE_SERIAL Serial

void webSocketEvent(WStype_t type, uint8_t * payload, size_t length) {

	switch(type) {
		case WStype_DISCONNECTED:
			USE_SERIAL.printf("[WSc] Disconnected!\n");
			break;
		case WStype_CONNECTED: {
			USE_SERIAL.printf("[WSc] Connected to url: %s\n", payload);

			// send message to server when Connected
			webSocket.sendTXT("Connected");
		}
			break;
		case WStype_TEXT:
			USE_SERIAL.printf("[WSc] get text: %s\n", payload);

			// send message to server
			// webSocket.sendTXT("message here");
			break;
		case WStype_BIN:
			USE_SERIAL.printf("[WSc] get binary length: %u\n", length);
			hexdump(payload, length);

			// send data to server
			// webSocket.sendBIN(payload, length);
			break;
        case WStype_PING:
            // pong will be send automatically
            USE_SERIAL.printf("[WSc] get ping\n");
            break;
        case WStype_PONG:
            // answer to a ping we send
            USE_SERIAL.printf("[WSc] get pong\n");
            break;
    }

}

void setup() {
	// USE_SERIAL.begin(921600);
	USE_SERIAL.begin(115200);

	//Serial.setDebugOutput(true);
	USE_SERIAL.setDebugOutput(true);

	USE_SERIAL.println();
	USE_SERIAL.println();
	USE_SERIAL.println();

 	Serial.println(F("Initialize System"));
 	//Init ESP8266 Wifi
 	WiFi.config(ip, gateway, subnet); 						// forces to use the fix IP
 	WiFi.begin(ssid, password);
 	while (WiFi.status() != WL_CONNECTED) {
 			delay(500);
 			Serial.print(F("."));
 	}
 	Serial.print(F("connected to Wifi! IP address : http://")); Serial.println(WiFi.localIP()); // Print the IP address


	for(uint8_t t = 4; t > 0; t--) {
		USE_SERIAL.printf("[SETUP] BOOT WAIT %d...\n", t);
		USE_SERIAL.flush();
		delay(1000);
	}

/*	WiFiMulti.addAP("test-WiFi", "test1234");

	//WiFi.disconnect();
	while(WiFiMulti.run() != WL_CONNECTED) {
		delay(100);
	}
*/
	// server address, port and URL
	webSocket.begin("192.168.1.44", 81, "/");

	// event handler
	webSocket.onEvent(webSocketEvent);

	// use HTTP Basic Authorization this is optional remove if not needed
//	webSocket.setAuthorization("user", "Password");

	// try ever 5000 again if connection has failed
	webSocket.setReconnectInterval(5000);
  
  // start heartbeat (optional)
  // ping server every 15000 ms
  // expect pong from server within 3000 ms
  // consider connection disconnected if pong is not received 2 times
  webSocket.enableHeartbeat(15000, 3000, 2);

}

```
void loop() {
	webSocket.loop();
}

```
