#include <ESP8266WiFi.h>
#include <ThingSpeak.h>
#include <DHT.h>
#include <PulseSensorPlayground.h>

#define DHTPIN D2          
#define DHTTYPE DHT11     
#define PULSE_PIN A0      
#define THRESHOLD 550     

const char* ssid = "SOUR";         
const char* password = "ommuruga"; 
unsigned long myChannelNumber =3135282; 
const char* myWriteAPIKey = "D60AX8TWJPW8JDNX";    

WiFiClient client;
DHT dht(DHTPIN, DHTTYPE);
PulseSensorPlayground pulseSensor;

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("Connecting to WiFi...");

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ WiFi connected!");
  Serial.println(WiFi.localIP());

  ThingSpeak.begin(client);
  dht.begin();
  
  pulseSensor.analogInput(PULSE_PIN);
  pulseSensor.setThreshold(THRESHOLD);
  pulseSensor.begin();
}

void loop() {
  float temp = dht.readTemperature();
  float hum = dht.readHumidity();
  int bpm = pulseSensor.getBeatsPerMinute();

  if (pulseSensor.sawStartOfBeat()) {
    Serial.print("💓 BPM: ");
    Serial.println(bpm);
  }

  Serial.print("🌡️ Temp: ");
  Serial.print(temp);
  Serial.print("°C  💧 Humidity: ");
  Serial.print(hum);
  Serial.println("%");


  ThingSpeak.setField(1, temp);
  ThingSpeak.setField(2, hum);
  ThingSpeak.setField(3, bpm);

  int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);

  if (x == 200) {
    Serial.println("✅ Data sent to ThingSpeak!");
  } else {
    Serial.print("❌ Error sending data. HTTP code: ");
    Serial.println(x);
  }

  Serial.println("-----------------------------");
  delay(20000); // 
}
