Create an account on ThingSpeak https://thingspeak.com/
Create a new channel with one field label
Get the API Key
Review the “Update a Channel Feed” Url
ThingSpeak is insanely easy to use. Once you have your channel and key you can simply make an HTTP request to https://api.thingspeak.com/update?api_key=YOUR_KEY_HERE&field1=4 to send the value 4 into field1. Try it in your browser, then review the data in the Private View of your channel.
Use the Uno as a bridge to talk to the module directly. Bypassing the Uno boot loader connecting RESET to GND, then connect TX to TX, RX to RX, GND to GND, VCC and CH_PD to 5v.
We can connect the reset pin on the Uno to GND to bypass the Arduino bootloader. This allows you to connects to the Uno’s TX and RX from the Module. Since the Uno’s USB cable is properly hooked up to the Uno’s TR and RX, any signals put on those lines will pass right on to the module. Here’s the trick, in this case, we want the TX on the Uno to connect to the TX on the module and the RX from the Uno to the RX on the Module. Realize that all we’re doing here is extending the TX line into the module. Only in this bridge model do you have TX to TX and RX to RX.
Once you have it wired up you should be able to use the Arduino Serial Monitor in the Arduino IDE to access the Module. Remember you’ll connect to the port the Arduino is on.
 Be sure to put in your own wifi name and password
 Connect to thingspeak. Get API Key and paste it with the output. 
 AT+CIPSEND needs to be 2 more than the size of your GET line. I believe this is related to the Serial Monitor sending NL & CR.
There needs to be a space after the URL in your GET and the new line characters, that tripped me up.

# Iot-smartbulb
#include <SoftwareSerial.h>

String ssid ="WIFI_NAME";
String password="WIFI_PASSWORD";

SoftwareSerial esp(3, 2);// RX, TX

String server = "http://api.thingspeak.com"; //Your Host 
String uri = "/YOUR_API"; // Your URI

int RED_BULB=5; 
int YELLOW_BULB=6;

void setup() {

  pinMode(RED_BULB, OUTPUT);
  pinMode(YELLOW_BULB, OUTPUT);
  
  digitalWrite(RED_BULB, HIGH);
  digitalWrite(YELLOW_BULB, HIGH);
  
  esp.begin(9600);
  
  Serial.begin(9600);

  connectWifi();
 
  
  delay(1000);

}
void connectWifi() {

String cmd = "AT+CWJAP=\"" +ssid+"\",\"" + password + "\"";

esp.println(cmd);

delay(4000);

if(esp.find("OK")) {

Serial.println("Connected!");

}
else {

Serial.println("Cannot connect to wifi ! Connecting again..."); }
connectWifi();

}
void loop() {
esp.println("AT+CIPSTART=\"TCP\",\"" + server + "\",80");//start a TCP connection.

if( esp.find("OK")) {

Serial.println("TCP connection ready");

} delay(1000);

String getRequest =
"GET " + uri + " HTTP/1.0\r\n" +
"Host: " + server + "\r\n" +
"Accept: *" + "/" + "*\r\n" +
"Content-Type: application/json\r\n" +
"\r\n";

String sendCmd = "AT+CIPSEND=";

esp.print(sendCmd);

esp.println(getRequest.length() );

delay(500);

if(esp.find(">")) { 
  Serial.println("Sending.."); 
  esp.print(getRequest);
  
if( esp.find("SEND OK")) { 
  
Serial.println("Packet sent");

while (esp.available()) {

String response = esp.readString();

int RED_BULB_ON = response.indexOf("RED_BULB>TRUE")>0?1:0;
int YELLOW_BULB_ON = response.indexOf("YELLOW_BULB>TRUE")>0?1:0;

if(RED_BULB_ON==1)
{
  digitalWrite(RED_BULB, LOW);
}
else
{
  digitalWrite(RED_BULB, HIGH);
}
if(YELLOW_BULB_ON==1)
{
  digitalWrite(YELLOW_BULB, LOW);
}
else
{
  digitalWrite(YELLOW_BULB, HIGH);
}
}
esp.println("AT+CIPCLOSE");

}
}
}
