*Quick links :*
[Home](/README.md) - [Part 1](../part1/README.md) - [**Part 2**](../part2/README.md) - [Part 3](../part3/README.md) - [Part 4](../part4/README.md) - [Sensors](/en/sensors/README.md)
***
**Part 2** - [Device Registration](DEVICE.md) - [Application](APP.md) - [MQTT](/MQTT.md) - [Server Certificate](CERT1.md) - [**Client Certificate**](CERT2.md)
***

# Using a Device Certificate to authenticate to the Watson IoT platform

## Lab Objectives

In this lab you will extend the application by enabling client side certificates.  You will learn how to:

- Generate client keys and certificates
- Modify the application to use the client certificates
- Configure the IoT platform connection policy to require tokens and/or certificates

### Step 1 - Generating the key and certificate for a device

The openssl tool must be used to generate the key and certificate, as in the previous lab.  You need to work in the same directory as you did in the previous lab, as the commands below need access to the rootCA_certificate.pem file.  If you altered the root CA key password, then remember to change the value in the commands shown below:

```bash
openssl genrsa -aes256 -out SecuredDev01_key.pem 2048

openssl req -new -sha256 -subj "/C=GB/ST=DOR/L=Bournemouth/O=z53u40/OU=z53u40 Corporate/CN=d:ESP32S:dev01" -key SecuredDev01_key.pem -out SecuredDev01_crt.csr

openssl x509 -days 3650 -in SecuredDev01_crt.csr -out SecuredDev01_crt.pem -req -sha256 -CA rootCA_certificate.pem -passin pass:password123 -CAkey rootCA_key.pem -set_serial 131

```

### Step 2 - Get the device cert, key and CA cert content and Fingerprint

Open rootCA_certificate.pem, SecuredDev01_key.pem and SecuredDev01_crt.pem using a text editor and get their text content to be used later in the code in Step 3

For Fingerprint, run this command:

```bash
openssl s_client -servername MQTT_HOST -connect MQTT_HOST:8883 < /dev/null 2>/dev/null | openssl x509 -fingerprint -sha256 -noout -in /dev/stdin
```
Check what MQTT_HOST in [MQTT](/MQTT.md) part

### Step 3 - Modify the application to use the client certificate and key

Now you can modify the code to load the certificates and add them to the connection:


Update the code within the setup() function to set the certificates and key.

```C++
// This is an example of how you hardcode the cert
const char* ca = \
"-----BEGIN CERTIFICATE-----\n" \
"MIIDdzCCAl+gAwIBAgIBATANBgkqhkiG9w0BAQsFADB2MQswCQYDVQQGEwJHQjEM\n" \
"MAoGA1UECAwDRE9SMRQwEgYDVQQHDAtCb3VybmVtb3V0aDEPMA0GA1UECgwGejUz\n" \
"dTQwMRkwFwYDVQQLDBBteWl5Z2ggQ29ycG9yYXRlMRcwFQYDVQQDDA5teWl5Z2gg\n" \
"Um9vdCBDQTAeFw0xOTAyMTYwNDExMzVaFw0yODExMTUwNDExMzVaMHYxCzAJBgNV\n" \
"BAYTAkdCMQwwCgYDVQQIDANET1IxFDASBgNVBAcMC0JvdXJuZW1vdXRoMQ8wDQYD\n" \
"VQQKDAZ6NTN1NDAxGTAXBgNVBAsMEG15aXlnaCBDb3Jwb3JhdGUxFzAVBgNVBAMM\n" \
"Dm15aXlnaCBSb290IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA\n" \
"szKLiVvQL9AfXopquOJ1q4CCUFUcttvQH84TvtsrtaBLUt7VZ3JZphe9ne6iA9S9\n" \
"gjFpqlNp3S+1QrIF9LhyJwJNzo/2S8n8QPZsrl7knx1WXi2P0HKxEuHovCArUawZ\n" \
"JqbseVaE/9HOf7Iz941aDXMagIG1c37GjpMSqHp8g5Fk7X6H64UoHjrOcoqqKYg0\n" \
"851UbCzUEusxpdrjx4Ld6iZ2qztVtjgfrp5f7EL7O2O7DVWdZ4U0l8O8TbRHGDMs\n" \
"aliD9/NPSZ6lVa1h9RrDZVbyxzzyLp1NmmU2s+EJcAUQVK08B+ScT7vjd64pLuY9\n" \
"hQSrsRKwcteV77Xc2nmTcwIDAQABoxAwDjAMBgNVHRMEBTADAQH/MA0GCSqGSIb3\n" \
"DQEBCwUAA4IBAQA6A9riroV72HW2HhAHM3MrcKmaxCnb5f2eBjWHzVvFJLtc5TsU\n" \
"JbtSDKXwCyrPvSHl1VwuKoAmZFFgGJKmNi8vbCq7kOgVfv640egm2TcHfvayUyT+\n" \
"APNx6YRKhato0iKfOXbsfqd1gryk7lSBiKoOHlUg0xAL432IeOvasq1D8Exzv8+h\n" \
"48uk+hW6Ms0FT+Wwd8MBsAwmMyQqjYFGLv2CCJs1i8eSUIAu+9IAQSAVMQ38M6WF\n" \
"jxf0I09fwK/814fKnCBmW5DQLeC8/AHkno161NOua+h7+DKOEsHoAA/K9K2yGO8t\n" \
"+zQ8mEcSen/LQoBxc/A5x3nUHrmRe8wcvws4\n" \
"-----END CERTIFICATE-----\n" ;

const char* key = \
"<Key content goes here similar to ca>";

const char* cert = \
"<Cert content goes here similar to ca>";

 // Start serial console
  Serial.begin(115200);
  Serial.setTimeout(2000);
  while (!Serial) { }
  Serial.println();
  Serial.println("ESP32S Sensor Application");

  // Start WiFi connection
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi Connected");

  // Start connected devices
  dht.begin();
  pixel.begin();

  wifiClient.setCACert(ca);
  wifiClient.setPrivateKey(key);
  wifiClient.setCertificate(cert);
```

### Step 4 - Run the application

Save, compile and upload the sketch to the device and verify the device connects.

### Step 5 - Configure the security policy on the IoT platform

You now have client certificates working with the device, so can now choose how you want devices to be verified.  If you open the IoT Platform console and got to the settings section then the Connection Security section and Open Connection Security Policy you see you have a number of options:

- TLS Optional
- TLS with Token Authentication
- TLS with Client Certificate Authentication (This option is used for the example code)
- TLS with Client Certificate AND Token Authentication
- TLS with Client Certificate OR Token Authentication

You can now decide what policy you want.  If you don't want to use Token Authentication then change the **connect()** function call and omit the user and token information:

- with token authentication : `if (mqtt.connect(MQTT_DEVICEID, MQTT_USER, MQTT_TOKEN)) {`
- without token authentication : `if (mqtt.connect(MQTT_DEVICEID)) {`

You will also see that you can create Custom Rules in addition to the Default Rule.  This allows different device types to have a different policy . If a device type doesn't match a custom rule then the default rule is used.

### Solution Code

The finished application should look like this:

```C++
#include <FS.h>
#include <SPIFFS.h>
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <time.h>
#include <Adafruit_NeoPixel.h>
#include <DHT.h>
#include <ArduinoJson.h>
#include <PubSubClient.h>

// --------------------------------------------------------------------------------------------
//        UPDATE CONFIGURATION TO MATCH YOUR ENVIRONMENT
// --------------------------------------------------------------------------------------------
// Watson IoT connection details
#define MQTT_HOST "z53u40.messaging.internetofthings.ibmcloud.com"
#define MQTT_PORT 8883
#define MQTT_DEVICEID "d:z53u40:ESP32S:dev01"
#define MQTT_USER "use-token-auth"
#define MQTT_TOKEN "password"
#define MQTT_TOPIC "iot-2/evt/status/fmt/json"
#define MQTT_TOPIC_DISPLAY "iot-2/cmd/display/fmt/json"
// Add GPIO pins used to connect devices
#define RGB_PIN 5 // GPIO pin the data line of RGB LED is connected to
#define DHT_PIN 4 // GPIO pin the data line of the DHT sensor is connected to

// Specify DHT11 (Blue) or DHT22 (White) sensor
#define DHTTYPE DHT11
#define NEOPIXEL_TYPE NEO_RGB + NEO_KHZ800

// Temperatures to set LED by (assume temp in C)
#define ALARM_COLD 0.0
#define ALARM_HOT 30.0
#define WARN_COLD 10.0
#define WARN_HOT 25.0

//Timezone info
#define TZ_OFFSET -5  //Hours timezone offset to GMT (without daylight saving time)
#define TZ_DST    60  //Minutes timezone offset for Daylight saving

// Add WiFi connection information
char ssid[] = "duyhard";     //  your network SSID (name)
char pass[] = "YouPassWord";  // your network password


// --------------------------------------------------------------------------------------------
//        SHOULD NOT NEED TO CHANGE ANYTHING BELOW THIS LINE
// --------------------------------------------------------------------------------------------
Adafruit_NeoPixel pixel = Adafruit_NeoPixel(1, RGB_PIN, NEOPIXEL_TYPE);
DHT dht(DHT_PIN, DHTTYPE);


// MQTT objects
void callback(char* topic, byte* payload, unsigned int length);
WiFiClientSecure wifiClient;
PubSubClient mqtt(MQTT_HOST, MQTT_PORT, callback, wifiClient);

// variables to hold data
StaticJsonBuffer<100> jsonBuffer;
JsonObject& payload = jsonBuffer.createObject();
JsonObject& status = payload.createNestedObject("d");
static char msg[50];

float h = 0.0;
float t = 0.0;

unsigned char r = 0;
unsigned char g = 0;
unsigned char b = 0;

void callback(char* topic, byte* payload, unsigned int length) {
  // handle message arrived
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] : ");
  
  payload[length] = 0; // ensure valid content is zero terminated so can treat as c-string
  Serial.println((char *)payload);
}

void setup() {

// This is an example of how you hardcode the cert
const char* ca = \
"-----BEGIN CERTIFICATE-----\n" \
"MIIDdzCCAl+gAwIBAgIBATANBgkqhkiG9w0BAQsFADB2MQswCQYDVQQGEwJHQjEM\n" \
"MAoGA1UECAwDRE9SMRQwEgYDVQQHDAtCb3VybmVtb3V0aDEPMA0GA1UECgwGejUz\n" \
"dTQwMRkwFwYDVQQLDBBteWl5Z2ggQ29ycG9yYXRlMRcwFQYDVQQDDA5teWl5Z2gg\n" \
"Um9vdCBDQTAeFw0xOTAyMTYwNDExMzVaFw0yODExMTUwNDExMzVaMHYxCzAJBgNV\n" \
"BAYTAkdCMQwwCgYDVQQIDANET1IxFDASBgNVBAcMC0JvdXJuZW1vdXRoMQ8wDQYD\n" \
"VQQKDAZ6NTN1NDAxGTAXBgNVBAsMEG15aXlnaCBDb3Jwb3JhdGUxFzAVBgNVBAMM\n" \
"Dm15aXlnaCBSb290IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA\n" \
"szKLiVvQL9AfXopquOJ1q4CCUFUcttvQH84TvtsrtaBLUt7VZ3JZphe9ne6iA9S9\n" \
"gjFpqlNp3S+1QrIF9LhyJwJNzo/2S8n8QPZsrl7knx1WXi2P0HKxEuHovCArUawZ\n" \
"JqbseVaE/9HOf7Iz941aDXMagIG1c37GjpMSqHp8g5Fk7X6H64UoHjrOcoqqKYg0\n" \
"851UbCzUEusxpdrjx4Ld6iZ2qztVtjgfrp5f7EL7O2O7DVWdZ4U0l8O8TbRHGDMs\n" \
"aliD9/NPSZ6lVa1h9RrDZVbyxzzyLp1NmmU2s+EJcAUQVK08B+ScT7vjd64pLuY9\n" \
"hQSrsRKwcteV77Xc2nmTcwIDAQABoxAwDjAMBgNVHRMEBTADAQH/MA0GCSqGSIb3\n" \
"DQEBCwUAA4IBAQA6A9riroV72HW2HhAHM3MrcKmaxCnb5f2eBjWHzVvFJLtc5TsU\n" \
"JbtSDKXwCyrPvSHl1VwuKoAmZFFgGJKmNi8vbCq7kOgVfv640egm2TcHfvayUyT+\n" \
"APNx6YRKhato0iKfOXbsfqd1gryk7lSBiKoOHlUg0xAL432IeOvasq1D8Exzv8+h\n" \
"48uk+hW6Ms0FT+Wwd8MBsAwmMyQqjYFGLv2CCJs1i8eSUIAu+9IAQSAVMQ38M6WF\n" \
"jxf0I09fwK/814fKnCBmW5DQLeC8/AHkno161NOua+h7+DKOEsHoAA/K9K2yGO8t\n" \
"+zQ8mEcSen/LQoBxc/A5x3nUHrmRe8wcvws4\n" \
"-----END CERTIFICATE-----\n" ;

const char* key = \
"<Key content goes here similar to ca>";

const char* cert = \
"<Cert content goes here similar to ca>";

 // Start serial console
  Serial.begin(115200);
  Serial.setTimeout(2000);
  while (!Serial) { }
  Serial.println();
  Serial.println("ESP32S Sensor Application");

  // Start WiFi connection
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi Connected");

  // Start connected devices
  dht.begin();
  pixel.begin();
 
  //Set certificates and key
  wifiClient.setCACert(ca);
  wifiClient.setPrivateKey(key);
  wifiClient.setCertificate(cert);
  

  // Set time from NTP servers
  configTime(TZ_OFFSET * 3600, TZ_DST * 60, "pool.ntp.org", "0.pool.ntp.org");
  Serial.println("\nWaiting for time");
  unsigned timeout = 5000;
  unsigned start = millis();
  while (millis() - start < timeout) {
      time_t now = time(nullptr);
      if (now > (2018 - 1970) * 365 * 24 * 3600) {
          break;
      }
      delay(100);
  }
  delay(1000); // Wait for time to fully sync
  Serial.println("Time sync'd");
  time_t now = time(nullptr);
  Serial.println(ctime(&now));

  // Connect to MQTT - IBM Watson IoT Platform
   while(! mqtt.connected()){
//    if (mqtt.connect(MQTT_DEVICEID, MQTT_USER, MQTT_TOKEN)) { // Token Authentication
   if (mqtt.connect(MQTT_DEVICEID)) { // No Token Authentication
    const char* fingerprint = "Fingerprint value from Step 2";
      if (wifiClient.verify(fingerprint,MQTT_HOST)) {
        Serial.println("certificate matches");
      } else {
        // ignore for now - but usually don't want to proceed if a valid cert not presented!
        Serial.println("certificate doesn't match");
      }
      Serial.println("MQTT Connected");
      mqtt.subscribe(MQTT_TOPIC_DISPLAY);
    } else {
      Serial.println("MQTT Failed to connect! ... retrying");
      delay(500);
    }
  }
}

void loop() {
  mqtt.loop();
  while (!mqtt.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (mqtt.connect(MQTT_DEVICEID, MQTT_USER, MQTT_TOKEN)) { // Token Authentication
//    if (mqtt.connect(MQTT_DEVICEID, MQTT_USER, MQTT_TOKEN)) { // No Token Authentication
      Serial.println("MQTT Connected");
// Should verify the certificates here - like in the startup function
      mqtt.subscribe(MQTT_TOPIC_DISPLAY);
      mqtt.loop();
    } else {
      Serial.println("MQTT Failed to connect!");
      delay(5000);
    }
  }
  h = dht.readHumidity();
  t = dht.readTemperature(); // uncomment this line for centigrade
  // t = dht.readTemperature(true); // uncomment this line for Fahrenheit

  // Check if any reads failed and exit early (to try again).
  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
  } else {
    // Set RGB LED Colour based on temp
    b = (t < ALARM_COLD) ? 255 : ((t < WARN_COLD) ? 150 : 0);
    r = (t >= ALARM_HOT) ? 255 : ((t > WARN_HOT) ? 150 : 0);
    g = (t > ALARM_COLD) ? ((t <= WARN_HOT) ? 255 : ((t < ALARM_HOT) ? 150 : 0)) : 0;
    pixel.setPixelColor(0, r, g, b);
    pixel.show();

    // Send data to Watson IoT Platform
    status["temp"] = t;
    status["humidity"] = h;
    payload.printTo(msg, 50);
    Serial.println(msg);
    if (!mqtt.publish(MQTT_TOPIC, msg)) {
      Serial.println("MQTT Publish failed");
    }
  }
  // Pause - but keep polling MQTT for incoming messages
  for (int i = 0; i < 10; i++) {
    mqtt.loop();
    delay(1000);
  }
}
```

***
**Part 2** - [Device Registration](DEVICE.md) - [Application](APP.md) - [MQTT](MQTT.md) - [Server Certificate](CERT1.md) - [**Client Certificate**](CERT2.md)
***
*Quick links :*
[Home](/README.md) - [Part 1](../part1/README.md) - [**Part 2**](../part2/README.md) - [Part 3](../part3/README.md) - [Part 4](../part4/README.md) - [Sensors](/en/sensors/README.md)