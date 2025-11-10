# 3223600029_Syauqi-Nabil-Falah_ESP32-S3-Peripherals_n_Cores

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

