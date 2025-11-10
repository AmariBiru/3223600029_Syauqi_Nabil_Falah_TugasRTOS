# 3223600029_Syauqi-Nabil-Falah_ESP32-S3-Peripherals_n_Cores
Sebelum mulai memprogram, rangkailah ESP32 berikut ini dengan konfigurasi pin yang sesuai.

<img width="633" height="708" alt="image" src="https://github.com/user-attachments/assets/c4d0d1d4-1219-46ac-af00-3a31464146ec" />

Masukkan program berikut pada Wokwi:
# Program memilih Task pada setup
```cpp
// =================== SETUP ====================
void setup() {
  Serial.begin(115200);
  delay(300);

  // ================= USER SELECTION =================
  TaskChoice selectedTask = TASK_LED;  // 🟢 Choose which task to run
  uint8_t selectedCore = 0;               // 🟢 Choose core (0 or 1)
  // ==================================================

  const char* taskName = "";
  void (*taskFunc)(void*) = nullptr;
  uint32_t stackSize = 4000;

  switch (selectedTask) {
    case TASK_LED:     taskFunc = taskLED;     taskName = "LED";     break;
    case TASK_BUZZER:  taskFunc = taskBuzzer;  taskName = "BUZZER";  break;
    case TASK_BUTTON:  taskFunc = taskButton;  taskName = "BUTTON";  break;
    case TASK_POT:     taskFunc = taskPot;     taskName = "POT";     break;
    case TASK_OLED:    taskFunc = taskOLED;    taskName = "OLED";    stackSize = 6000; break;
    case TASK_ENCODER: taskFunc = taskEncoder; taskName = "ENCODER"; break;
    case TASK_SERVO:   taskFunc = taskServo;   taskName = "SERVO";   stackSize = 5000; break;
    case TASK_STEPPER: taskFunc = taskStepper; taskName = "STEPPER"; stackSize = 7000; break;
  }

  // Create the selected task
  if (taskFunc != nullptr) {
    xTaskCreatePinnedToCore(
      taskFunc,
      taskName,
      stackSize,
      NULL,
      1,
      &currentTask,
      selectedCore
    );
    Serial.printf("[SETUP] Running %s on Core %d\n", taskName, selectedCore);
  } else {
    Serial.println("[SETUP] Invalid task selected");
  }
}
```
untuk mengganti task maka selectedTask dapat diganti menjadi salah satu task yang terdapat pada enum berikut ini, setelah pemilihan task dilakukan, pemilihan core juga dapat di inisialisasi.

**TASK_LED**
Task ini berfungsi untuk menyalakan dan mematikan LED secara bergantian dengan interval waktu tertentu. Delay selama 500 ms digunakan untuk mengatur kecepatan kedipan agar terlihat jelas di mata.

**TASK_BUZZER**
Bagian ini mengatur buzzer agar berbunyi dengan pola tertentu, yaitu hidup selama 200 ms dan mati selama 200 ms secara berulang.

**TASK_BUTTON**
Task ini bertugas membaca dua tombol input yang terhubung ke pin BUTTON_A dan BUTTON_B. Keduanya diatur dengan mode INPUT_PULLUP, sehingga logika yang dibaca akan aktif rendah (LOW saat ditekan). Nilai dari masing-masing tombol akan ditampilkan di Serial Monitor.

**TASK_POT**
Fungsi dari task ini adalah membaca nilai dari potensiometer melalui pin analog. Nilai tersebut kemudian dikirim ke Serial Monitor agar kita bisa melihat perubahan tegangan saat potensiometer diputar. Nilai ini biasanya digunakan untuk mengatur parameter seperti kecepatan, intensitas, atau posisi motor secara manual. Pembacaan dilakukan setiap 200 ms agar respon tetap halus tanpa terlalu membebani prosesor.

**TASK_OLED**
Task ini menangani tampilan pada layar OLED dengan menggunakan komunikasi I2C. Setiap setengah detik, tampilan diperbarui untuk menjaga agar layar tetap aktif. Task ini berguna untuk menampilkan status, data sensor, atau pesan singkat dari sistem secara real-time.

**TASK_ENCODER**
Bagian ini membaca sinyal dari rotary encoder yang terdiri dari dua pin, yaitu CLK dan DT. Setiap kali posisi encoder berubah, task akan mendeteksi arah putaran dengan membandingkan kedua sinyal tersebut. Nilai hasil perhitungan disimpan di variabel encValue dan dicetak ke Serial Monitor. Dengan cara ini, kita bisa mengetahui apakah encoder diputar ke kanan atau kiri.

**TASK_SERVO**
Task ini menggerakkan motor servo dengan pola maju-mundur dari sudut 20 derajat hingga 160 derajat, lalu kembali lagi ke posisi awal. Pergerakan dilakukan bertahap setiap beberapa derajat dengan jeda 15 ms agar gerakannya halus.

**TASK_STEPPER**
Task terakhir ini berfungsi untuk mengontrol motor stepper menggunakan library AccelStepper. Motor digerakkan ke posisi 300 langkah, lalu kembali ke -150 langkah secara berulang. Setiap pergerakan dilakukan dengan akselerasi dan kecepatan tertentu agar gerakannya tidak tiba-tiba.

# enum untuk memilih Task
```cpp
// =================== ENUM FOR SELECTION ====================
enum TaskChoice {
  TASK_LED,
  TASK_BUZZER,
  TASK_BUTTON,
  TASK_POT,
  TASK_OLED,
  TASK_ENCODER,
  TASK_SERVO,
  TASK_STEPPER
};
```
# Task untuk LED
```cpp
#include <Arduino.h>

#define LED_PIN 2

void taskLED(void *pv) {
  pinMode(LED_PIN, OUTPUT);
  while (1) {
    digitalWrite(LED_PIN, !digitalRead(LED_PIN));
    Serial.printf("[LED] Core %d\n", xPortGetCoreID());
    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
```
LED Core 0:

https://github.com/user-attachments/assets/0321390b-c566-46a1-b55f-7278a747336b

LED Core 1:

https://github.com/user-attachments/assets/440f6657-2f46-42db-890a-94dc96640974



# Task untuk BUZZER
```cpp
#include <Arduino.h>

#define BUZZER_PIN 46

void taskBuzzer(void *pv) {
  pinMode(BUZZER_PIN, OUTPUT);
  while (1) {
    digitalWrite(BUZZER_PIN, HIGH);
    vTaskDelay(200 / portTICK_PERIOD_MS);
    digitalWrite(BUZZER_PIN, LOW);
    vTaskDelay(200 / portTICK_PERIOD_MS);
    Serial.printf("[BUZZER] Core %d\n", xPortGetCoreID());
  }
}
```
BUZZER Core 0:

https://github.com/user-attachments/assets/0f676dee-6f51-4714-a2c5-6e06cab43293

BUZZER Core 1:

https://github.com/user-attachments/assets/9db83a41-3d54-48e9-b8e3-153175b6b4f2



# Task untuk BUTTON
```cpp
#include <Arduino.h>

#define BUTTON_A 35
#define BUTTON_B 19

void taskButton(void *pv) {
  pinMode(BUTTON_A, INPUT_PULLUP);
  pinMode(BUTTON_B, INPUT_PULLUP);
  while (1) {
    Serial.printf("[BUTTON] A=%d B=%d Core %d\n",
                  digitalRead(BUTTON_A),
                  digitalRead(BUTTON_B),
                  xPortGetCoreID());
    vTaskDelay(200 / portTICK_PERIOD_MS);
  }
}
```
BUTTON Core 0:

https://github.com/user-attachments/assets/2207cf32-366b-4707-b53e-5649cfd6c430

BUTTON Core 1:

https://github.com/user-attachments/assets/23133d3f-925c-4868-8318-4bb19525ebd1



# Task untuk POTENTIOMETER
```cpp
#include <Arduino.h>

#define POT_INPUT 5

void taskPot(void *pv) {
  while (1) {
    int val = analogRead(POT_INPUT);
    Serial.printf("[POT] %d Core %d\n", val, xPortGetCoreID());
    vTaskDelay(200 / portTICK_PERIOD_MS);
  }
}
```
POTENTIOMETER Core 0:

https://github.com/user-attachments/assets/c37425e8-345d-4a95-9e33-d17f6fc1c41f

POTENTIOMETER Core 1:

https://github.com/user-attachments/assets/38d41251-28b1-4ba3-8aed-46a979f11d18



# Task untuk OLED
```cpp
#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define OLED_SDA 38
#define OLED_SCL 39

Adafruit_SSD1306 screen(128, 64, &Wire, -1);

void taskOLED(void *pv) {
  Wire.begin(OLED_SDA, OLED_SCL);

  if (!screen.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("[OLED] Failed");
    vTaskDelete(NULL);
  }

  while (1) {
    screen.clearDisplay();
    screen.setTextSize(1);
    screen.setTextColor(SSD1306_WHITE);
    screen.setCursor(0, 0);
    screen.printf("OLED Running\nCore %d", xPortGetCoreID());
    screen.display();
    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
```
OLED Core 0:

https://github.com/user-attachments/assets/7346863a-d3f7-4935-93f2-c5ebe73b2e46

OLED Core 1:

https://github.com/user-attachments/assets/7bd608a2-3112-493f-b586-6fb87591ae92



# Task untuk ENCODER
```cpp
#include <Arduino.h>

#define ENC_CLK 41
#define ENC_DT  40

volatile int encValue = 0;
int prevClk = 0;

void taskEncoder(void *pv) {
  pinMode(ENC_CLK, INPUT);
  pinMode(ENC_DT, INPUT);
  prevClk = digitalRead(ENC_CLK);

  while (1) {
    int clk = digitalRead(ENC_CLK);
    if (clk != prevClk) {
      if (digitalRead(ENC_DT) == clk) encValue++;
      else encValue--;
      Serial.printf("[ENC] %d Core %d\n", encValue, xPortGetCoreID());
    }
    prevClk = clk;
    vTaskDelay(5 / portTICK_PERIOD_MS);
  }
}
```
ENCODER Core 0:

https://github.com/user-attachments/assets/94e925d8-955f-4b97-8309-17c8bfec6ca7

ENCODER Core 1:

https://github.com/user-attachments/assets/bf785606-52f6-408e-91b6-11c0ad0c77ab



# Task untuk SERVO
```cpp
#include <Arduino.h>
#include <ESP32Servo.h>

#define SERVO_PIN 18

Servo myservo;

void taskServo(void *pv) {
  myservo.attach(SERVO_PIN);
  while (1) {
    for (int p = 20; p <= 160; p += 4) {
      myservo.write(p);
      vTaskDelay(15 / portTICK_PERIOD_MS);
    }
    for (int p = 160; p >= 20; p -= 4) {
      myservo.write(p);
      vTaskDelay(15 / portTICK_PERIOD_MS);
    }
    Serial.printf("[SERVO] Core %d\n", xPortGetCoreID());
  }
}
```
SERVO Core 0:

https://github.com/user-attachments/assets/42651da9-b34d-4c5b-bfcc-bdbab68c23f0

SERVO Core 1:

https://github.com/user-attachments/assets/01f76c42-5d4f-445b-b2eb-f0174f73558e



# Task untuk STEPPER
```cpp
#include <Arduino.h>
#include <AccelStepper.h>

#define STP1 4
#define STP2 7
#define STP3 8
#define STP4 9

AccelStepper motor(AccelStepper::FULL4WIRE, STP1, STP3, STP2, STP4);

void taskStepper(void *pv) {
  motor.setMaxSpeed(500);
  motor.setAcceleration(200);

  while (1) {
    motor.moveTo(300);
    while (motor.distanceToGo()) motor.run();

    motor.moveTo(-150);
    while (motor.distanceToGo()) motor.run();

    Serial.printf("[STEPPER] Core %d\n", xPortGetCoreID());
  }
}
```
STEPPER Core 0:

https://github.com/user-attachments/assets/8ddda1a9-4750-48b6-bc43-8b0cc5c7eef4

STEPPER Core 1:

https://github.com/user-attachments/assets/769fb321-cb34-4856-8a8a-715f7ed4af34


