# Snap-n-Store-using-Xiao-ESP32S3
A hobby project i made as a prequel to my other project
Here's a **detailed and beginner-friendly README** for your project. It includes step-by-step instructions, explanations, and troubleshooting tips. 🚀  

---

### **📸 ESP32-CAM Image Uploader with Flask Server**  
This project lets an **ESP32-CAM** capture images and send them to a **Flask server** over WiFi. The Flask server saves these images in a folder with timestamps.  

---

## **📜 Table of Contents**
1. [📌 Features](#features)  
2. [🛠️ Hardware & Software Requirements](#hardware--software-requirements)  
3. [📡 How It Works](#how-it-works)  
4. [🚀 Getting Started](#getting-started)  
5. [📝 ESP32-CAM Code](#esp32-cam-code)  
6. [💻 Flask Server Code](#flask-server-code)  
7. [⚡ Running the Project](#running-the-project)  
8. [🛠️ Troubleshooting](#troubleshooting)  
9. [📚 References](#references)  

---

## **📌 Features**
✅ Captures images using **ESP32-CAM**  
✅ Sends images to a **Flask server** via WiFi  
✅ Saves images in an **uploads folder**  
✅ Names images using **timestamps** to prevent overwriting  
✅ Works on **local network (LAN)**  

---

## **🛠️ Hardware & Software Requirements**  
### **🔩 Hardware**  
1. 🟢 ESP32-CAM (XIAO ESP32S3 or any ESP32-CAM module)  
2. 🔵 Micro USB cable  
3. 🔴 WiFi network  

### **💻 Software**  
1. **Arduino IDE** (for ESP32-CAM programming)  
2. **Python 3** (for Flask server)  
3. **Flask** (Python web framework)  

---

## **📡 How It Works**  
📷 The ESP32-CAM **captures** an image →  
📡 Sends it **via WiFi** to the Flask server →  
💾 The server **saves** the image in a folder  

🔹 **Example Flow:**  
1️⃣ ESP32-CAM captures an image every **10 seconds**  
2️⃣ Sends the image to the server at **`http://192.168.X.X:5000/uploads`**  
3️⃣ Flask server receives and saves the image in an **`uploads`** folder  

---

## **🚀 Getting Started**
### **1️⃣ Install Required Software**
📥 Download and install:  
- **Arduino IDE** 👉 [Download Here](https://www.arduino.cc/en/software)  
- **Python 3** 👉 [Download Here](https://www.python.org/downloads/)  

### **2️⃣ Install ESP32 Board in Arduino IDE**
1. Open **Arduino IDE**  
2. Go to **File → Preferences**  
3. In **Additional Board Manager URLs**, enter:  
   ```
   https://dl.espressif.com/dl/package_esp32_index.json
   ```
4. Go to **Tools → Board → Board Manager**  
5. Search for **ESP32** and install **"esp32 by Espressif Systems"**  

### **3️⃣ Install Required Python Libraries**
Open **Command Prompt (Windows)** or **Terminal (Mac/Linux)** and type:  
```sh
pip install flask
```

---

## **📝 ESP32-CAM Code**
This code runs on the ESP32-CAM. It connects to **WiFi**, captures an image, and sends it to the Flask server.  

### **🔹 Steps to Upload Code:**
1. Open **Arduino IDE**  
2. Install **ESP32 Board** (if not installed)  
3. Copy and paste this code into Arduino IDE  
4. **Change WiFi credentials** (`ssid` and `password`)  
5. Select **ESP32-CAM Board**  
6. Upload the code 🚀  

```cpp
#include "esp_camera.h"
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "YOUR_WIFI_NAME";  
const char* password = "YOUR_WIFI_PASSWORD";  
const char* serverUrl = "http://192.168.X.X:5000/uploads";  

#define CAMERA_MODEL_XIAO_ESP32S3  
#include "camera_pins.h"  

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);
    
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    
    Serial.println("WiFi Connected!");

    camera_config_t config;
    config.ledc_channel = LEDC_CHANNEL_0;
    config.ledc_timer = LEDC_TIMER_0;
    config.pin_d0 = Y2_GPIO_NUM;
    config.pin_d1 = Y3_GPIO_NUM;
    config.pin_d2 = Y4_GPIO_NUM;
    config.pin_d3 = Y5_GPIO_NUM;
    config.pin_d4 = Y6_GPIO_NUM;
    config.pin_d5 = Y7_GPIO_NUM;
    config.pin_d6 = Y8_GPIO_NUM;
    config.pin_d7 = Y9_GPIO_NUM;
    config.pin_xclk = XCLK_GPIO_NUM;
    config.pin_pclk = PCLK_GPIO_NUM;
    config.pin_vsync = VSYNC_GPIO_NUM;
    config.pin_href = HREF_GPIO_NUM;
    config.pin_sccb_sda = SIOD_GPIO_NUM;
    config.pin_sccb_scl = SIOC_GPIO_NUM;
    config.pin_pwdn = PWDN_GPIO_NUM;
    config.pin_reset = RESET_GPIO_NUM;
    config.xclk_freq_hz = 20000000;
    config.pixel_format = PIXFORMAT_JPEG;
    config.frame_size = FRAMESIZE_QVGA;
    config.jpeg_quality = 10;
    config.fb_count = 2;

    if (esp_camera_init(&config) != ESP_OK) {
        Serial.println("Camera Init Failed!");
        return;
    }
}

void loop() {
    camera_fb_t* fb = esp_camera_fb_get();
    if (!fb) return;

    HTTPClient http;
    http.begin(serverUrl);
    http.addHeader("Content-Type", "image/jpeg");
    int httpResponseCode = http.POST(fb->buf, fb->len);
    Serial.println(httpResponseCode);
    http.end();

    esp_camera_fb_return(fb);
    delay(10000);
}
```

---

## **💻 Flask Server Code**
This Python script runs on your computer and saves images sent by ESP32-CAM.  

### **🔹 Steps to Run Server:**
1. Open **Command Prompt / Terminal**  
2. Navigate to the project folder  
3. Run this command:  
   ```sh
   python server.py
   ```
4. The server will start at **`http://0.0.0.0:5000/uploads`**  

```python
import os
import time
from flask import Flask, request

app = Flask(__name__)

UPLOAD_FOLDER = "uploads"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/uploads', methods=['POST'])
def upload_file():
    try:
        if not request.data:
            return "No image data received", 400

        timestamp = time.strftime("%Y%m%d-%H%M%S")
        filename = f"{UPLOAD_FOLDER}/image_{timestamp}.jpg"

        with open(filename, "wb") as f:
            f.write(request.data)
        
        print(f"Image saved: {filename}")
        return "Image received", 200

    except Exception as e:
        print("Error:", str(e))
        return "Server Error", 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

---

## **⚡ Running the Project**
1️⃣ **Run Flask Server** 🖥️  
```sh
python server.py
```
2️⃣ **Upload Code to ESP32-CAM** 📡  
3️⃣ **Check `uploads` folder for saved images** 📷  

---

## **🛠️ Troubleshooting**
❌ **ESP32 Not Connecting to WiFi?**  
➡️ Check `ssid` and `password` in ESP32 code  

❌ **Server Not Receiving Images?**  
➡️ Ensure both devices are on the same WiFi network  

---

## **📚 References**
🔹 [ESP32-CAM Documentation](https://www.espressif.com/en/products/socs/esp32)  
🔹 [Flask Documentation](https://flask.palletsprojects.com/)  

---

This README should make it **super easy** for anyone to understand and set up your project! 🚀
