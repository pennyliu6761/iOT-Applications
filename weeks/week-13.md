[⬅ 回目錄](../README.md) | [⬅ 上一週：第 12 週：人機介面指令輸入](./week-12.md) | [下一週：第 14 週：突破 delay() 的限制（多任務處理觀念引入） ➡](./week-14.md)

---

# 第四階段：進階整合與期末專案（第 13～16 週）

**階段目標**：跳脫單一元件的學習模式，挑戰多任務處理與系統架構設計，展現工業管理系學生擅長的「系統整合」思維價值。

---


## 第 13 週：全彩視覺化看板
**新增元件：NeoPixel 燈環/燈條（可定址 RGB LED）**

### 📌 學習目標
- 理解「可定址 LED」與傳統 RGB LED 的差異：一條線可以控制多顆燈，且每顆燈顏色獨立。
- 學會使用 `<Adafruit_NeoPixel.h>` 函式庫。
- 練習將感測數值轉換為「視覺化長條圖」呈現方式。
- 建立產線稼動率看板的完整聲光邏輯。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **NeoPixel Strip（燈條，建議設定為 8 顆燈）** 到工作區。
2. 只需要 3 條線：**VCC 接 5V**、**GND 接 GND**、**訊號線（DIN）接 Arduino Pin 6**。
3. 沿用第 4 週可變電阻（A0）、第 6 週 TMP36（A2）。

---

### 🕐 第 1 小時：定址 LED 基礎

**情境故事**：傳統一顆一顆的 LED 太占腳位，主管希望能用「一條線控制一整排燈」，做出更精緻的視覺化效果，這正是 NeoPixel 燈條在工業看板上被廣泛採用的原因。

#### 實作 1：點亮第一顆特定顏色的燈
```cpp
// ============================================
// 實作 1：NeoPixel 基礎控制
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6        // 訊號腳位
#define NUMPIXELS 8  // 燈條上的燈珠數量

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();     // 初始化燈條
  strip.show();      // 一開始先全部熄滅（清空殘留資料）

  strip.setPixelColor(0, strip.Color(255, 0, 0));  // 第 0 顆燈設為紅色
  strip.show();       // 必須呼叫 show() 才會真正更新到硬體上
}

void loop() {
}
```

#### 實作 2：依次亮起、依次熄滅動畫
```cpp
// ============================================
// 實作 2：燈條依序點亮、依序熄滅
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.show();
}

void loop() {
  // 依序點亮
  for (int i = 0; i < NUMPIXELS; i++) {
    strip.setPixelColor(i, strip.Color(0, 255, 0));
    strip.show();
    delay(100);
  }
  // 依序熄滅
  for (int i = 0; i < NUMPIXELS; i++) {
    strip.setPixelColor(i, strip.Color(0, 0, 0));
    strip.show();
    delay(100);
  }
}
```

#### 實作 3：呼吸與閃爍特效函式
```cpp
// ============================================
// 實作 3：把「全部呼吸」與「全部閃爍」寫成自訂函式
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

// 讓整條燈條統一顯示同一種顏色
void setAll(int r, int g, int b) {
  for (int i = 0; i < NUMPIXELS; i++) {
    strip.setPixelColor(i, strip.Color(r, g, b));
  }
  strip.show();
}

void breatheEffect(int r, int g, int b) {
  for (int brightness = 0; brightness <= 255; brightness++) {
    setAll(r * brightness / 255, g * brightness / 255, b * brightness / 255);
    delay(5);
  }
  for (int brightness = 255; brightness >= 0; brightness--) {
    setAll(r * brightness / 255, g * brightness / 255, b * brightness / 255);
    delay(5);
  }
}

void setup() {
  strip.begin();
  strip.show();
}

void loop() {
  breatheEffect(0, 100, 255);   // 藍白色呼吸燈
}
```

---

### 🕑 第 2 小時：資料轉化為視覺表現

**情境故事**：把過去學過的感測資料（旋鈕、溫度）直接「畫」在燈條上，這是資訊視覺化最直觀的呈現方式，也是儀表板設計的核心能力。

#### 實作 4：進度條實作
```cpp
// ============================================
// 實作 4：依旋鈕數值，點亮對應數量的燈（類似手機電量條）
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
int pinPot = A0;

void setup() {
  strip.begin();
  strip.show();
}

void loop() {
  int rawValue = analogRead(pinPot);
  int litCount = map(rawValue, 0, 1023, 0, NUMPIXELS);  // 轉換為 0~8 顆燈的數量

  for (int i = 0; i < NUMPIXELS; i++) {
    if (i < litCount) {
      strip.setPixelColor(i, strip.Color(0, 255, 0));  // 點亮
    } else {
      strip.setPixelColor(i, strip.Color(0, 0, 0));    // 熄滅
    }
  }
  strip.show();
}
```

#### 實作 5：溫度計視覺化（冷到暖漸變）
```cpp
// ============================================
// 實作 5：溫度計燈條 - 藍(冷) 到紅(熱) 的漸變顯示
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
int pinTemp = A2;

void setup() {
  strip.begin();
  strip.show();
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  // 假設溫度顯示範圍為 15°C ~ 40°C
  int litCount = map(celsius, 15, 40, 0, NUMPIXELS);
  litCount = constrain(litCount, 0, NUMPIXELS);

  for (int i = 0; i < NUMPIXELS; i++) {
    if (i < litCount) {
      // 顏色隨位置由藍轉紅：越後面（溫度越高）越紅
      int redAmount = map(i, 0, NUMPIXELS - 1, 0, 255);
      strip.setPixelColor(i, strip.Color(redAmount, 0, 255 - redAmount));
    } else {
      strip.setPixelColor(i, strip.Color(0, 0, 0));
    }
  }
  strip.show();
}
```

#### 實作 6：AGV 電量指示燈模擬
```cpp
// ============================================
// 實作 6：AGV 電量指示 - 電量低於門檻時整條閃紅燈警示
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
int pinPot = A0;   // 用旋鈕模擬電量感測

void setup() {
  strip.begin();
  strip.show();
}

void loop() {
  int rawValue = analogRead(pinPot);
  int batteryPercent = map(rawValue, 0, 1023, 0, 100);
  int litCount = map(batteryPercent, 0, 100, 0, NUMPIXELS);

  if (batteryPercent < 20) {
    // 電量過低：整條閃紅燈
    for (int i = 0; i < NUMPIXELS; i++) {
      strip.setPixelColor(i, strip.Color(255, 0, 0));
    }
    strip.show();
    delay(200);
    for (int i = 0; i < NUMPIXELS; i++) {
      strip.setPixelColor(i, strip.Color(0, 0, 0));
    }
    strip.show();
    delay(200);
  } else {
    for (int i = 0; i < NUMPIXELS; i++) {
      if (i < litCount) {
        strip.setPixelColor(i, strip.Color(0, 255, 0));
      } else {
        strip.setPixelColor(i, strip.Color(0, 0, 0));
      }
    }
    strip.show();
  }
}
```

---

### 🕒 第 3 小時：工廠稼動率環狀指標

**情境故事**：期末專題正式起跑前的最後一個主題週，我們把 NeoPixel 燈條包裝成完整的「機台稼動率狀態指示器」，這將是許多學生期末專題會採用的核心視覺元件。

#### 實作 7：依機台狀態切換整條燈環模式
```cpp
// ============================================
// 實作 7：待機（呼吸白燈）/ 運轉（旋轉綠燈）雙模式
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
int pinButton = 7;
bool machineRunning = false;
int lastButtonState = LOW;
int rotateIndex = 0;

void setup() {
  strip.begin();
  strip.show();
  pinMode(pinButton, INPUT);
}

void loop() {
  int currentState = digitalRead(pinButton);
  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);
    machineRunning = !machineRunning;
  }
  lastButtonState = currentState;

  if (machineRunning) {
    // 運轉狀態：單顆綠燈繞圈旋轉
    for (int i = 0; i < NUMPIXELS; i++) {
      strip.setPixelColor(i, i == rotateIndex ? strip.Color(0, 255, 0) : strip.Color(0, 0, 0));
    }
    strip.show();
    rotateIndex = (rotateIndex + 1) % NUMPIXELS;
    delay(100);
  } else {
    // 待機狀態：全體白色呼吸燈
    for (int b = 0; b <= 150; b += 5) {
      for (int i = 0; i < NUMPIXELS; i++) {
        strip.setPixelColor(i, strip.Color(b, b, b));
      }
      strip.show();
      delay(20);
    }
  }
}
```

#### 實作 8：異常發生時燈條轉紅快速閃爍
```cpp
// ============================================
// 實作 8：加入異常狀態 - 紅色快速閃爍，優先權最高
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
int pinTemp = A2;
int pinButton = 7;
bool machineRunning = false;
int lastButtonState = LOW;
int rotateIndex = 0;
float alarmThreshold = 35.0;

void setAll(int r, int g, int b) {
  for (int i = 0; i < NUMPIXELS; i++) {
    strip.setPixelColor(i, strip.Color(r, g, b));
  }
  strip.show();
}

void setup() {
  strip.begin();
  strip.show();
  pinMode(pinButton, INPUT);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > alarmThreshold) {
    // 異常：優先權最高，直接快速閃紅燈，跳過其他邏輯
    setAll(255, 0, 0);
    delay(150);
    setAll(0, 0, 0);
    delay(150);
    return;
  }

  int currentState = digitalRead(pinButton);
  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);
    machineRunning = !machineRunning;
  }
  lastButtonState = currentState;

  if (machineRunning) {
    for (int i = 0; i < NUMPIXELS; i++) {
      strip.setPixelColor(i, i == rotateIndex ? strip.Color(0, 255, 0) : strip.Color(0, 0, 0));
    }
    strip.show();
    rotateIndex = (rotateIndex + 1) % NUMPIXELS;
    delay(100);
  } else {
    setAll(50, 50, 50);
  }
}
```

#### 實作 9：自訂聲光組合函式庫（供期末專題呼叫）
```cpp
// ============================================
// 實作 9：把常用的聲光效果整理成一套函式庫
// 這些函式可以直接複製到期末專題中重複使用
// ============================================

#include <Adafruit_NeoPixel.h>

#define PIN 6
#define NUMPIXELS 8

Adafruit_NeoPixel strip(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
int pinBuzzer = 8;

// --- 聲光函式庫 ---
void setAll(int r, int g, int b) {
  for (int i = 0; i < NUMPIXELS; i++) {
    strip.setPixelColor(i, strip.Color(r, g, b));
  }
  strip.show();
}

void alarmFlash() {
  setAll(255, 0, 0);
  tone(pinBuzzer, 2000);
  delay(150);
  setAll(0, 0, 0);
  noTone(pinBuzzer);
  delay(150);
}

void normalIdle() {
  setAll(0, 50, 0);
  noTone(pinBuzzer);
}

void warningPulse() {
  setAll(255, 150, 0);
  tone(pinBuzzer, 800);
  delay(300);
  noTone(pinBuzzer);
  delay(300);
}

void setup() {
  strip.begin();
  strip.show();
}

void loop() {
  normalIdle();
  delay(2000);
  warningPulse();
  warningPulse();
  alarmFlash();
  alarmFlash();
}
```

### 🌶️ 第 13 週進階挑戰題
1. **彩虹旋轉效果**：讓 8 顆燈同時呈現彩虹漸層，並整體緩慢旋轉。
2. **雙資料源視覺化**：同一條燈條，左半部顯示溫度、右半部顯示距離，兩種資料同時呈現在一條燈條上。
3. **期末專題函式庫整理**：把本學期學過的所有「自訂函式」（setColor、readDistance、setAll 等）整理成一份個人的「函式庫筆記」，作為期末專題的工具箱。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 12 週：人機介面指令輸入](./week-12.md) | [下一週：第 14 週：突破 delay() 的限制（多任務處理觀念引入） ➡](./week-14.md)
