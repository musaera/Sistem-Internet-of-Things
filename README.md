#include <WiFi.h>
#include <PubSubClient.h>

// Ganti dengan kredensial jaringan Anda
const char* ssid = "Wokwi-GUEST"; // Nama jaringan Wi-Fi
const char* password = "";         // Kata sandi Wi-Fi

// Pengaturan MQTT Broker
const char* mqtt_server = "sfl-official.cloud.shiftr.io"; // Alamat broker MQTT
const int mqtt_port = 1883;                               // Port untuk koneksi tanpa SSL
const char* mqtt_user = "sfl-official";                   // Username MQTT
const char* mqtt_password = "eDeHBopIdCqPWslY";           // Password MQTT

WiFiClient wifiClient;
PubSubClient client(wifiClient);

const int ledPin = 5; // Pin GPIO untuk mengontrol LED

// Fungsi callback, dipanggil saat ada pesan masuk
void callback(char* topic, byte* payload, unsigned int length) {
  // Mengubah payload menjadi string
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }

  Serial.print("Pesan masuk [");
  Serial.print(topic);
  Serial.print("]: ");
  Serial.println(message);

  // Mengontrol LED berdasarkan pesan yang diterima
  if (String(topic) == "sensor/led") {
    if (message == "nyala") {
      digitalWrite(ledPin, HIGH); // Menyalakan LED
      Serial.println("LED NYALA");
    } else if (message == "mati") {
      digitalWrite(ledPin, LOW);  // Mematikan LED
      Serial.println("LED MATI");
    }
  }
}

void setup() {
  Serial.begin(115200);
  delay(100);

  // Mengatur pin LED sebagai output
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW); // Memulai dengan kondisi LED mati

  // Menghubungkan ke Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Menghubungkan ke WiFi...");
  }
  Serial.println("Terhubung ke WiFi");

  // Mengatur koneksi MQTT
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);

  // Menghubungkan ke broker MQTT
  while (!client.connected()) {
    Serial.println("Menghubungkan ke MQTT...");
    if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
      Serial.println("Terhubung ke broker MQTT");
      // Berlangganan topik kontrol LED
      client.subscribe("sensor/led");
    } else {
      Serial.print("Gagal terhubung, kode error: ");
      Serial.print(client.state());
      delay(2000);
    }
  }
}

void loop() {
  client.loop(); // Memastikan koneksi tetap aktif

  // Mengirim data suhu simulasi setiap 5 detik
  if (millis() % 5000 == 0) { // Mengirim data setiap 5 detik
    float temperature = random(20, 30); // Suhu acak antara 20 dan 30
    client.publish("sensor/suhu", String(temperature).c_str());
    Serial.print("Suhu yang dikirim: ");
    Serial.println(temperature);
  }
}
