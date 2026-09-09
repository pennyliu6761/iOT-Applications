[⬅ 回目錄](../README.md) | [⬅ 上一週：第 7 週：精準角度控制](./week-07.md) | [下一週：第 9 週：空間距離感知 ➡](./week-09.md)

---

## 第 8 週：期中專題 — 智慧倉儲環境監控站
**整合元件：LED、按鈕、RGB LED、可變電阻、蜂鳴器、光敏電阻、TMP36、伺服馬達**

### 📌 學習目標
- 綜合運用前 7 週所學的所有元件與程式邏輯，完成一個具備實際情境意義的小型系統。
- 練習系統化的專題開發流程：硬體佈局 → 個別感測器測試 → 核心邏輯撰寫 → 邊界測試。
- 初步接觸 `millis()`（非阻塞式計時）的概念，為第 14 週鋪路。
- 學習基本的程式碼註解規範，建立工管系重視的「文件化」素養。

### 🔌 Tinkercad 電路搭建指引（總覽）
本週不新增元件，而是把前 7 週的元件全部整合到同一塊麵包板上：

| 元件 | 建議腳位 |
|---|---|
| LED（照明燈） | Pin 13 |
| 按鈕（緊急停止/確認鈕） | Pin 7 |
| RGB LED（狀態指示） | Pin 9 (R) / 10 (G) / 11 (B) |
| 可變電阻（模擬警戒門檻設定） | A0 |
| 蜂鳴器（警報） | Pin 8 |
| 光敏電阻（環境光偵測） | A1 |
| TMP36（溫度偵測） | A2 |
| 伺服馬達（排煙窗/逃生門） | Pin 6 |

**佈線建議**：先在麵包板左半部佈置所有「輸入類」元件（按鈕、可變電阻、光敏電阻、TMP36），右半部佈置「輸出類」元件（LED、RGB LED、蜂鳴器、伺服馬達），電源正負軌貫穿整塊麵包板共用，減少接線混亂。

---

### 🕐 第 1 小時：系統架構與硬體佈局

**情境故事**：你被指派負責建置一套「無人倉儲環境監控站」——半夜倉庫沒人值班，系統必須自動判斷環境是否安全，並在異常時自主應變。

#### 實作 1：在 Tinkercad 佈置好所有元件
> 這是操作型任務，沒有程式碼。請依照上方腳位總覽表，將 8 個元件全部佈置到麵包板上，並完成所有接線。建議先用鉛筆在紙上畫一次接線草圖，再對照 Tinkercad 操作，養成工程圖面思維。

#### 實作 2：定義各元件 Pin 腳並完成 setup()
```cpp
// ============================================
// 實作 2：宣告所有變數與 setup() 初始化
// 教學重點：好的變數命名 = 好的文件，未來接手的人不用看電路圖就懂程式邏輯
// ============================================

#include <Servo.h>

// --- 輸入元件腳位 ---
int pinButton = 7;        // 緊急停止/確認鈕
int pinPot    = A0;       // 警戒門檻設定旋鈕
int pinLdr    = A1;       // 環境光感測
int pinTemp   = A2;       // 溫度感測

// --- 輸出元件腳位 ---
int pinLed    = 13;       // 照明燈
int pinR = 9, pinG = 10, pinB = 11;  // RGB 狀態燈
int pinBuzzer = 8;        // 警報蜂鳴器

Servo windowServo;        // 排煙窗/逃生門伺服馬達

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
  windowServo.attach(6);
  windowServo.write(0);    // 初始狀態：窗戶關閉
  Serial.begin(9600);
}

void loop() {
  // 本階段先留空，下一步驟開始逐步加入邏輯
}
```

#### 實作 3：各感測器 Serial 監控測試
```cpp
// ============================================
// 實作 3：先確認所有感測器讀值都正常，再開始寫核心邏輯
// 工程原則：先驗證輸入資料正確，再處理邏輯，除錯效率高很多
// ============================================

int pinLdr  = A1;
int pinTemp = A2;
int pinPot  = A0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int lightLevel = analogRead(pinLdr);

  int rawTemp = analogRead(pinTemp);
  float voltage = rawTemp * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  int potValue = analogRead(pinPot);

  Serial.print("光線: ");   Serial.print(lightLevel);
  Serial.print("  溫度: "); Serial.print(celsius);
  Serial.print("  旋鈕: "); Serial.println(potValue);

  delay(300);
}
```

---

### 🕑 第 2 小時：核心邏輯撰寫（條件樹）

**情境故事**：系統上線第一版功能——夜間自動照明、高溫自動排煙、緊急人工介入。這一小時將前面所學的邏輯完整組裝起來。

#### 實作 4：環境光過暗自動開燈
```cpp
// ============================================
// 實作 4：夜間自動照明邏輯
// ============================================

int pinLdr = A1;
int pinLed = 13;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int lightLevel = analogRead(pinLdr);

  if (lightLevel < 300) {
    digitalWrite(pinLed, HIGH);
  } else {
    digitalWrite(pinLed, LOW);
  }
}
```

#### 實作 5：火災感知（高溫觸發排煙窗與警報）
```cpp
// ============================================
// 實作 5：高溫觸發伺服馬達開啟排煙窗，並響起警報
// ============================================

#include <Servo.h>

int pinTemp = A2;
int pinBuzzer = 8;
Servo windowServo;
float fireThreshold = 40.0;   // 假設超過 40°C 視為火災風險

void setup() {
  windowServo.attach(6);
  windowServo.write(0);
}

void loop() {
  int rawTemp = analogRead(pinTemp);
  float voltage = rawTemp * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > fireThreshold) {
    windowServo.write(90);   // 開啟排煙窗
    tone(pinBuzzer, 2000);
  } else {
    windowServo.write(0);    // 關閉排煙窗
    noTone(pinBuzzer);
  }
}
```

#### 實作 6：緊急停止鈕（一鍵關閉所有機構）
```cpp
// ============================================
// 實作 6：緊急停止鈕 - 不論任何感測狀態，一律強制關閉所有輸出
// 工業安全觀念：緊急停止（E-Stop）永遠享有最高優先權
// ============================================

#include <Servo.h>

int pinButton = 7;   // 這裡借用按鈕模擬緊急停止鈕
int pinBuzzer = 8;
int pinLed = 13;
Servo windowServo;

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
  windowServo.attach(6);
}

void loop() {
  bool emergencyStop = digitalRead(pinButton) == HIGH;

  if (emergencyStop) {
    // 最高優先權：不管其他條件，一律關閉
    digitalWrite(pinLed, LOW);
    noTone(pinBuzzer);
    windowServo.write(0);
  } else {
    // 正常運作邏輯（此處省略，代表接續實作 4、5 的邏輯）
  }
}
```
**教學重點**：讓學生思考「為什麼緊急停止判斷要寫在整個 `if-else` 結構的最前面（最高優先權）」，這是控制系統設計中極重要的安全思維。

---

### 🕒 第 3 小時：邊界測試與優化

**情境故事**：系統雛形完成後，QA（品保）階段開始——用各種「極端情境」測試系統是否會出現矛盾或當機的行為，這是任何專案上線前必經的過程。

#### 實作 7：認識 millis()（選修/進階引導，為第 14 週鋪路）
```cpp
// ============================================
// 實作 7：初探 millis() - 不用 delay 也能計時
// 這裡先讓學生看過一次，第 14 週會深入拆解原理
// ============================================

unsigned long previousTime = 0;   // 記錄上一次動作的時間點
int pinLed = 13;
bool ledState = false;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  unsigned long currentTime = millis();  // 取得「開機至今」經過的毫秒數

  // 每經過 500 毫秒才切換一次 LED 狀態（不會卡住整個 loop）
  if (currentTime - previousTime >= 500) {
    previousTime = currentTime;
    ledState = !ledState;
    digitalWrite(pinLed, ledState);
  }
  // 因為沒有用 delay()，loop() 可以持續快速執行去檢查其他感測器
}
```

#### 實作 8：組間互賽極端狀態測試
> 操作型任務：將班級分組，各組將完整整合程式碼上傳到 Tinkercad，交換給另一組進行「刁難測試」，例如：
> - 半夜（光線暗）又同時發生火災（高溫），系統會不會同時開燈又開窗，行為是否合理？
> - 按下緊急停止的同時發生火災，系統是否真的完全靜止（安全優先）？
> - 旋鈕轉到極端值（0 或 1023）時，程式是否會出現異常跳動？

#### 實作 9：程式碼整理與註解規範
```cpp
// ============================================
// 實作 9：完整專題程式碼 - 加上完整規範化註解
// 命名規範：變數用小駝峰命名（camelCase），註解說明「為什麼」而非「做什麼」
// ============================================

#include <Servo.h>

// ---------- 腳位定義區 ----------
int pinButton = 7;     // 緊急停止鈕：具有最高優先權
int pinLdr    = A1;    // 環境光感測：控制照明燈
int pinTemp   = A2;    // 溫度感測：控制排煙窗與警報
int pinLed    = 13;    // 照明燈
int pinBuzzer = 8;     // 警報蜂鳴器
Servo windowServo;     // 排煙窗馬達，接 Pin 6

// ---------- 系統參數區（方便日後調整，不用到處找數字） ----------
int lightThreshold = 300;     // 光線低於此值視為夜間
float fireThreshold = 40.0;   // 溫度高於此值視為火災風險

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
  windowServo.attach(6);
  windowServo.write(0);
  Serial.begin(9600);
}

void loop() {
  bool emergencyStop = digitalRead(pinButton) == HIGH;

  if (emergencyStop) {
    // 安全優先：緊急停止時，強制關閉所有輸出，不執行任何其他邏輯
    digitalWrite(pinLed, LOW);
    noTone(pinBuzzer);
    windowServo.write(0);
    return;   // 提前結束本次 loop()，跳過下面所有程式碼
  }

  // --- 夜間自動照明 ---
  int lightLevel = analogRead(pinLdr);
  digitalWrite(pinLed, lightLevel < lightThreshold ? HIGH : LOW);

  // --- 火災偵測與應變 ---
  int rawTemp = analogRead(pinTemp);
  float voltage = rawTemp * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > fireThreshold) {
    windowServo.write(90);
    tone(pinBuzzer, 2000);
  } else {
    windowServo.write(0);
    noTone(pinBuzzer);
  }
}
```
**`return` 新語法說明**：在 `loop()` 中執行 `return;` 會立刻結束這一輪迴圈，跳過後面尚未執行的程式碼，直接重新從 `loop()` 開頭再跑一次。這是實現「最高優先權」判斷最簡潔的寫法之一。

### 🌶️ 第 8 週期中專題加分挑戰
1. **加入 LCD 預告**：如果現在就想顯示目前溫度與光線數值在螢幕上（不使用 Serial Monitor），你會怎麼設計？（第 11 週會正式教學 LCD，這裡先讓學生規劃介面草圖）
2. **事件記錄**：新增一個變數統計「今晚總共觸發了幾次火災警報」，並在 Serial Monitor 顯示。
3. **分級應變**：溫度超過 35°C 先開窗降溫但不響鈴，超過 40°C 才加上警報聲，模擬真實世界「分級應變」的管理思維。

---

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 7 週：精準角度控制](./week-07.md) | [下一週：第 9 週：空間距離感知 ➡](./week-09.md)
