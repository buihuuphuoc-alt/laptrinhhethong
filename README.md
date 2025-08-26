# 🚀 ESP32 + MQTT + Flask + Web Dashboard

Dự án này sử dụng **ESP32** để đọc dữ liệu từ các cảm biến (ánh sáng, nhiệt độ, độ ẩm, chuyển động, âm thanh), sau đó gửi dữ liệu qua **MQTT (HiveMQ Cloud)**.  
Một ứng dụng **Flask (Python)** sẽ nhận dữ liệu và hiển thị real-time trên web.

---

## 📂 Cấu trúc thư mục

```

WEB/
│── app.py                # Flask backend + MQTT client
│── README.md             # Hướng dẫn chi tiết
│
├── templates/
│   └── index.html        # Giao diện web
│
├── static/
│   ├── script.js         # Xử lý WebSocket + cập nhật dữ liệu
│   └── style.css         # CSS giao diện
│
└── arduino/
└── sketch_aug20a.ino    # Code Arduino (ESP32)

```
---
## 🛠️ Thư viện cần cài

### 1. Arduino IDE (cho ESP32)
Vào **Library Manager** (`Sketch → Include Library → Manage Libraries`) và cài:  
- ✅ **WiFi** (đi kèm ESP32 core)  
- ✅ **PubSubClient** (MQTT client)  
- ✅ **DHT sensor library** (Adafruit)  
- ✅ **Adafruit Unified Sensor**  

👉 Nếu chưa có **ESP32 core**, cần thêm trong Board Manager:  
- Mở `File → Preferences → Additional Board URLs` và thêm:  
```
[https://dl.espressif.com/dl/package\_esp32\_index.json](https://dl.espressif.com/dl/package_esp32_index.json)

````
---

### 2. Python (Flask + MQTT client)
Cài thư viện:  
```bash
pip install flask paho-mqtt
````

Tuỳ chọn (nếu muốn realtime nhanh hơn):

```bash
pip install flask-socketio eventlet
```
---
## ⚙️ Cấu hình cần chỉnh

### ESP32 (`arduino/esp32_mqtt.ino`)

* Cập nhật WiFi SSID & Password
* Cập nhật MQTT Broker (HiveMQ Cloud):

```cpp
const char* mqtt_server = "xxxxxxx.s1.eu.hivemq.cloud";
const int mqtt_port = 8883;
const char* mqtt_user = "your-username";
const char* mqtt_pass = "your-password";
```

* Chân kết nối phần cứng:

| Thiết bị          | GPIO trên ESP32  |
| ----------------- | ---------------- |
| DHT11 (Data)      | 4                |
| Light sensor (AO) | 34               |
| PIR (Motion)      | 27               |
| Sound sensor (AO) | 35               |
| LED               | 2                |
| Quạt (qua relay)  | 5                |
| LED RGB module    | R=16, G=17, B=18 |

---

### Flask (`app.py`)

* MQTT Broker, user, pass phải **trùng với ESP32**
* Topics (ví dụ):

  * ESP32 publish: `esp32/sensors`
  * Flask subscribe: `esp32/sensors`

---

## 🔌 Sơ đồ kết nối phần cứng

### ESP32 + Cảm biến + Relay + LED RGB

```
+------------------+          +------------------+
| ESP32            |          | Thiết bị         |
|------------------|----------|------------------|
| GPIO 4           | ---> DHT11 (DATA)          |
| GPIO 34 (ADC)    | ---> Light sensor (AO)     |
| GPIO 27          | ---> PIR sensor (DO)       |
| GPIO 35 (ADC)    | ---> Sound sensor (AO)     |
| GPIO 2           | ---> LED thường            |
| GPIO 5           | ---> Relay (IN) -> Quạt    |
| GPIO 16          | ---> LED RGB (R)           |
| GPIO 17          | ---> LED RGB (G)           |
| GPIO 18          | ---> LED RGB (B)           |
| 3.3V / 5V, GND   | ---> Cấp nguồn cho module  |
+------------------+----------------------------+
```

---

## ▶️ Cách chạy

### 1. Nạp code Arduino

* Mở `arduino/esp32_mqtt.ino` bằng Arduino IDE
* Chọn board: **ESP32 Dev Module**
* Nạp code → ESP32 sẽ tự động publish dữ liệu cảm biến lên HiveMQ Cloud

### 2. Chạy Flask Web

```bash
cd WEB
python app.py
```

Mở trình duyệt tại:
👉 [http://127.0.0.1:5000](http://127.0.0.1:5000)

---

## 📊 Kết quả

* ESP32 gửi dữ liệu (ánh sáng, nhiệt độ, độ ẩm, chuyển động, âm thanh)
* Flask nhận qua MQTT → phát tới frontend
* Web hiển thị dữ liệu real-time (update dưới 0.5s)

---

## 💡 Lưu ý

* Light sensor module có **2 đèn LED**:

  * PWR LED (nguồn) luôn sáng.
  * DO LED sáng khi tín hiệu digital = LOW (tối) → có thể cần chỉnh biến trở màu xanh để đúng ngưỡng.
* Nếu muốn đo **giá trị ánh sáng chính xác**, hãy dùng chân **AO (Analog)** thay vì DO.

```
---