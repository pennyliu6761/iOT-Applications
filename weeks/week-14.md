[⬅ 回目錄](../README.md) | [⬅ 上一週：第 13 週：全彩視覺化看板](./week-13.md) | [下一週：第 15 週：期末專案實作與 QA 驗證（Troubleshooting） ➡](./week-15.md)

---

## 第 14 週：突破 delay() 的限制（多任務處理觀念引入）
> 這是工管系學生寫程式最容易卡關的地方，但對自動化系統極為重要，請務必放慢步調講解。

### 📌 學習目標
- 理解 `delay()` 會讓整個程式「暫停」，導致無法同時處理其他任務的根本限制。
- 學會使用 `millis()` 實現非阻塞式（Non-blocking）計時。
- 學會使用 `switch-case` 建構簡易狀態機（State Machine）。
- 為期末專題建立多工並行的系統架構能力。

### 🔌 Tinkercad 電路搭建指引
沿用第 13 週電路，並加回第 1 週的 LED（Pin 13）、第 2 週按鈕（Pin 7）、第 9 週超音波（Trig=2, Echo=3，若腳位衝突可挪用其他空腳位）。本週重點在程式邏輯，硬體不再新增元件。

---

### 🕐 第 1 小時：認識 millis() 計時器

**情境故事**：假設你在用 `delay()` 讓 LED 閃爍時，同時有一顆按鈕需要偵測——你會發現，只要程式卡在 `delay()` 裡，按鈕不管怎麼按都沒有反應！這正是自動化系統中最常見卻最致命的問題。

#### 實作 1：不用 delay，改用 millis() 讓 LED 閃爍
```cpp
// ============================================
// 實作 1：millis() 版本的 Blink
// 核心觀念：不斷比較「現在時間」和「上次動作時間」的差距
// ============================================

int pinLed = 13;
unsigned long previousTime = 0;   // 上一次切換燈光狀態的時間點
long interval = 500;              // 間隔時間（毫秒）
bool ledState = false;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  unsigned long currentTime = millis();   // 取得開機至今經過的毫秒數

  if (currentTime - previousTime >= interval) {
    previousTime = currentTime;    // 更新「上一次動作時間」
    ledState = !ledState;
    digitalWrite(pinLed, ledState);
  }
  // 這裡沒有任何 delay()，loop() 每次都會快速執行完畢，馬上進入下一輪
}
```
**教學類比**：把 `millis()` 想像成「牆上的時鐘」，`delay()` 則像是「叫你閉眼睛數 500 下才能張開眼睛做事」。用時鐘的好處是——即使你在等待，眼睛仍然睜著，可以同時觀察其他事情。

#### 實作 2：比較法——millis 閃爍 + 即時讀取按鈕
```cpp
// ============================================
// 實作 2：驗證 millis() 不會卡住按鈕讀取
// ============================================

int pinLed = 13;
int pinButton = 7;
unsigned long previousTime = 0;
long interval = 500;
bool ledState = false;

void setup() {
  pinMode(pinLed, OUTPUT);
  pinMode(pinButton, INPUT);
  Serial.begin(9600);
}

void loop() {
  unsigned long currentTime = millis();

  if (currentTime - previousTime >= interval) {
    previousTime = currentTime;
    ledState = !ledState;
    digitalWrite(pinLed, ledState);
  }

  // 因為沒有用 delay，這裡可以「同時」即時偵測按鈕，完全不受 LED 閃爍影響
  if (digitalRead(pinButton) == HIGH) {
    Serial.println("按鈕被按下！（即使 LED 正在閃爍中，仍能立刻偵測到）");
  }
}
```

#### 實作 3：多顆 LED 各自獨立頻率閃爍
```cpp
// ============================================
// 實作 3：三顆 LED 用完全不同的頻率同時獨立閃爍
// 用一個 delay() 絕對做不到這件事！
// ============================================

int ledPins[] = {11, 12, 13};
long intervals[] = {200, 500, 900};   // 三顆燈各自的閃爍間隔
unsigned long previousTimes[] = {0, 0, 0};
bool ledStates[] = {false, false, false};

void setup() {
  for (int i = 0; i < 3; i++) {
    pinMode(ledPins[i], OUTPUT);
  }
}

void loop() {
  unsigned long currentTime = millis();

  for (int i = 0; i < 3; i++) {
    if (currentTime - previousTimes[i] >= intervals[i]) {
      previousTimes[i] = currentTime;
      ledStates[i] = !ledStates[i];
      digitalWrite(ledPins[i], ledStates[i]);
    }
  }
}
```

---

### 🕑 第 2 小時：多任務產線系統

**情境故事**：真實產線同時運作著感測、顯示、動力輸出等多套子系統，彼此互不干擾。這一小時我們用 `millis()` 打造出一套真正的多任務框架，並學會用 `switch-case` 管理系統狀態，這將是期末專題的核心骨架。

#### 實作 4：三任務同時執行
```cpp
// ============================================
// 實作 4：任務 A（超音波掃描）、任務 B（LED 提示）、任務 C（Serial 輸出）同時執行
// ============================================

int pinTrig = 2;
int pinEcho = 3;
int pinLed = 13;

unsigned long lastTaskA = 0, lastTaskB = 0, lastTaskC = 0;
long intervalA = 200, intervalB = 500, intervalC = 1000;

float readDistance() {
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);
  long duration = pulseIn(pinEcho, HIGH);
  return duration / 58.0;
}

void setup() {
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  pinMode(pinLed, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  unsigned long now = millis();

  // 任務 A：每 200 毫秒掃描一次距離
  if (now - lastTaskA >= intervalA) {
    lastTaskA = now;
    float d = readDistance();
    // (這裡可以把距離存到全域變數，供其他任務使用)
  }

  // 任務 B：每 500 毫秒切換一次 LED
  if (now - lastTaskB >= intervalB) {
    lastTaskB = now;
    digitalWrite(pinLed, !digitalRead(pinLed));
  }

  // 任務 C：每 1000 毫秒印出一次心跳訊息
  if (now - lastTaskC >= intervalC) {
    lastTaskC = now;
    Serial.println("系統運作中...");
  }
}
```

#### 實作 5：狀態機框架（switch-case）
```cpp
// ============================================
// 實作 5：用 switch-case 管理系統狀態
// ============================================

enum SystemState { IDLE, RUNNING, ALARM };   // 定義三種系統狀態的名稱
SystemState currentState = IDLE;             // 目前狀態，預設為待機

int pinLed = 13;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  switch (currentState) {
    case IDLE:
      digitalWrite(pinLed, LOW);
      // 待機狀態的邏輯
      break;

    case RUNNING:
      digitalWrite(pinLed, HIGH);
      // 運轉狀態的邏輯
      break;

    case ALARM:
      digitalWrite(pinLed, !digitalRead(pinLed));  // 閃爍
      // 警報狀態的邏輯
      break;
  }
}
```

#### 實作 6：狀態切換的無縫演練
```cpp
// ============================================
// 實作 6：待機 → 運轉 → 警報 的完整狀態切換
// ============================================

enum SystemState { IDLE, RUNNING, ALARM };
SystemState currentState = IDLE;

int pinLed = 13;
int pinButton = 7;
int pinTemp = A2;
int lastButtonState = LOW;
unsigned long lastBlink = 0;

void setup() {
  pinMode(pinLed, OUTPUT);
  pinMode(pinButton, INPUT);
}

void loop() {
  // --- 狀態轉換判斷（不論目前在哪個狀態，都要持續檢查是否需要切換） ---
  int rawTemp = analogRead(pinTemp);
  float voltage = rawTemp * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > 35.0) {
    currentState = ALARM;   // 高溫，最高優先權切換到警報狀態
  } else {
    int currentButtonState = digitalRead(pinButton);
    if (currentButtonState == HIGH && lastButtonState == LOW) {
      delay(50);
      // 按鈕在 IDLE 與 RUNNING 之間切換
      currentState = (currentState == IDLE) ? RUNNING : IDLE;
    }
    lastButtonState = currentButtonState;
  }

  // --- 依目前狀態執行對應動作 ---
  switch (currentState) {
    case IDLE:
      digitalWrite(pinLed, LOW);
      break;

    case RUNNING:
      digitalWrite(pinLed, HIGH);
      break;

    case ALARM:
      unsigned long now = millis();
      if (now - lastBlink >= 150) {
        lastBlink = now;
        digitalWrite(pinLed, !digitalRead(pinLed));
      }
      break;
  }
}
```

---

### 🕒 第 3 小時：期末專題企劃與架構

**情境故事**：從這一刻起，你不再是跟著老師寫程式的學生，而是要獨當一面設計一套小型自動化系統的「系統工程師」。

#### 實作 7：小組發想期末題目
> 討論型任務。每組發想一個期末專題題目，**必須符合以下最低規格**：
> - 至少 1 個輸入元件（按鈕 / 旋鈕 / 光敏電阻 / TMP36 / 超音波 / Keypad）
> - 至少 2 個輸出元件（LED / RGB LED / 蜂鳴器 / 伺服馬達 / 直流馬達 / NeoPixel）
> - 必須包含 LCD 顯示狀態資訊
> - 建議主題方向：智慧倉儲管理、產線異常監控站、AGV 避障系統、電子密碼門禁、智慧交通閘門、產線 Andon 看板等（鼓勵學生結合自己熟悉的產業情境自由發想）

#### 實作 8：在 Tinkercad 完成硬體接線佈局
> 操作型任務：各組將所有選定元件佈置到同一塊麵包板，並完成接線。建議先手繪接線草圖並經教師確認後，再實際操作 Tinkercad，避免走太多冤枉路。

#### 實作 9：建立程式主架構
```cpp
// ============================================
// 實作 9：期末專題程式主架構範本
// 各組可直接複製此架構開始填入自己的邏輯
// ============================================

// ---------- 函式庫引入區 ----------
#include <Servo.h>
// #include <LiquidCrystal.h>
// #include <Keypad.h>
// #include <Adafruit_NeoPixel.h>

// ---------- 腳位與物件宣告區 ----------
// int pinXXX = ...;

// ---------- 系統參數區 ----------
enum SystemState { IDLE, RUNNING, ALARM };
SystemState currentState = IDLE;

// ---------- 多工計時變數區 ----------
unsigned long lastTaskA = 0;
long intervalA = 200;

void setup() {
  Serial.begin(9600);
  // 各元件初始化寫在這裡
}

void loop() {
  unsigned long now = millis();

  // --- 任務 A：感測器讀取（每 intervalA 執行一次）---
  if (now - lastTaskA >= intervalA) {
    lastTaskA = now;
    // 讀取感測器、更新狀態判斷
  }

  // --- 狀態機：依 currentState 執行對應輸出邏輯 ---
  switch (currentState) {
    case IDLE:
      break;
    case RUNNING:
      break;
    case ALARM:
      break;
  }
}
```

### 🌶️ 第 14 週進階挑戰題
1. **四任務並行**：在實作 4 的基礎上，再加入第四個獨立任務（如 NeoPixel 動畫更新），確認四個任務互不干擾。
2. **狀態機加入子狀態**：在 RUNNING 狀態下，再細分「低速運轉」與「高速運轉」兩個子狀態，思考如何設計資料結構。
3. **millis() 溢位思考題（進階選修）**：`millis()` 大約 49 天後會溢位歸零，請思考 `currentTime - previousTime` 這種寫法為什麼即使溢位也不會出錯？（提示：與無號整數的運算特性有關，可留給有興趣的學生課後研究）

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 13 週：全彩視覺化看板](./week-13.md) | [下一週：第 15 週：期末專案實作與 QA 驗證（Troubleshooting） ➡](./week-15.md)
