[⬅ 回目錄](../README.md) | [⬅ 上一週：第 10 週：連續動力驅動](./week-10.md) | [下一週：第 12 週：人機介面指令輸入 ➡](./week-12.md)

---

## 第 11 週：產線資訊可視化
**新增元件：LCD 16x2 顯示器**

### 📌 學習目標
- 學會使用 `<LiquidCrystal.h>` 函式庫控制 LCD 顯示文字與數字。
- 理解游標定位（`lcd.setCursor()`）的概念。
- 學會將多個感測器資料整合顯示在同一個介面上，打造簡易人機介面（HMI）。
- 學習自訂字元（Custom Character）技巧。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **LCD 16x2** 顯示器到工作區。
2. 依照下表接線（Tinkercad 元件上已標示每隻腳的名稱，對照即可）：

| LCD 腳位 | 接到 Arduino |
|---|---|
| VSS | GND |
| VDD | 5V |
| V0（對比度調整） | 接一顆可變電阻的中間腳，可變電阻另兩腳分接 5V 與 GND |
| RS | Pin 7 |
| RW | GND |
| E | Pin 8 |
| D4 | Pin 9 |
| D5 | Pin 10 |
| D6 | Pin 11 |
| D7 | Pin 12 |
| A（背光+） | 5V |
| K（背光-） | GND |

3. 沿用第 9 週超音波感測器（Trig=Pin 2, Echo=Pin 3，因腳位需重新配置以避開 LCD 佔用的腳位）、第 6 週 TMP36（A2）。

> ⚠️ 教學提醒：LCD 接線腳位較多，建議提供學生「已完成接線」的 Tinkercad 範例電路連結（教師可自行於 Tinkercad 建立一份範本並分享班級），把教學重點放在程式邏輯而非反覆除錯接線。

---

### 🕐 第 1 小時：LCD 基礎控制

**情境故事**：產線終於要有一塊「看得懂文字」的顯示螢幕了，不再只能靠燈號猜測狀態，這是邁向真正人機介面（HMI）的第一步。

#### 實作 1：顯示 "Hello World" 與學號
```cpp
// ============================================
// 實作 1：LCD 基礎顯示
// ============================================

#include <LiquidCrystal.h>

// 依序對應 RS, E, D4, D5, D6, D7 腳位
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

void setup() {
  lcd.begin(16, 2);          // 初始化 LCD：16 字元 x 2 行
  lcd.print("Hello World");  // 預設從第一行第一格開始印
  lcd.setCursor(0, 1);       // 游標移到第二行（行號從 0 開始）第一格
  lcd.print("Student ID:001");
}

void loop() {
  // LCD 顯示內容通常在 setup() 就設定好，不需要每次 loop 都重印
}
```

#### 實作 2：第二行實作自動遞增計數器
```cpp
// ============================================
// 實作 2：第二行顯示會自動加 1 的計時器
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
int seconds = 0;

void setup() {
  lcd.begin(16, 2);
  lcd.print("Line Status");
}

void loop() {
  lcd.setCursor(0, 1);
  lcd.print("Uptime: ");
  lcd.print(seconds);
  lcd.print("s  ");   // 多印幾個空白字元，蓋掉上一次的殘留數字（例如 100 變成 99 時，殘留的 "0" 不會消失）

  delay(1000);
  seconds++;
}
```

#### 實作 3：按鈕清除螢幕與切換頁面
```cpp
// ============================================
// 實作 3：按鈕切換 LCD 顯示頁面
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
int pinButton = 2;
int lastButtonState = LOW;
int page = 0;   // 0 = 頁面 A, 1 = 頁面 B

void setup() {
  pinMode(pinButton, INPUT);
  lcd.begin(16, 2);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);
    page = (page + 1) % 2;
    lcd.clear();   // 切換頁面前先清空螢幕，避免文字重疊

    if (page == 0) {
      lcd.print("Page A: Status");
    } else {
      lcd.print("Page B: Settings");
    }
  }
  lastButtonState = currentState;
}
```

---

### 🕑 第 2 小時：感測資料即時看板

**情境故事**：把過去幾週學過的感測器資料整合到同一塊看板上，模擬產線現場常見的「機況顯示器」，讓值班人員一眼掌握所有關鍵數據。

#### 實作 4：即時顯示超音波距離
```cpp
// ============================================
// 實作 4：LCD 顯示超音波測得的距離
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
int pinTrig = 2;
int pinEcho = 3;

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
  lcd.begin(16, 2);
  lcd.print("Distance:");
}

void loop() {
  float distance = readDistance();
  lcd.setCursor(0, 1);
  lcd.print("Dist: ");
  lcd.print(distance);
  lcd.print(" cm   ");
  delay(300);
}
```

#### 實作 5：同時顯示溫度，打造綜合儀表板
```cpp
// ============================================
// 實作 5：綜合儀表板 - 第一行溫度、第二行距離
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
int pinTrig = 2;
int pinEcho = 3;
int pinTemp = A2;

float readDistance() {
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);
  long duration = pulseIn(pinEcho, HIGH);
  return duration / 58.0;
}

float readTemperature() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  return (voltage - 0.5) * 100.0;
}

void setup() {
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  lcd.begin(16, 2);
}

void loop() {
  float temp = readTemperature();
  float distance = readDistance();

  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(temp);
  lcd.print("C   ");

  lcd.setCursor(0, 1);
  lcd.print("Dist: ");
  lcd.print(distance);
  lcd.print("cm   ");

  delay(300);
}
```

#### 實作 6：解決螢幕更新閃爍問題
```cpp
// ============================================
// 實作 6：只在數值真正改變時才更新畫面，避免不必要的閃爍
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
int pinTemp = A2;
float lastDisplayedTemp = -999;  // 給一個不可能出現的初始值，確保第一次一定會顯示

float readTemperature() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  return (voltage - 0.5) * 100.0;
}

void setup() {
  lcd.begin(16, 2);
  lcd.print("Temp Monitor");
}

void loop() {
  float temp = readTemperature();

  // 只有溫度變化超過 0.5 度，才重新印出（避免小數點跳動造成畫面一直閃）
  if (abs(temp - lastDisplayedTemp) > 0.5) {
    lcd.setCursor(0, 1);
    lcd.print("Temp: ");
    lcd.print(temp);
    lcd.print("C   ");
    lastDisplayedTemp = temp;
  }
  delay(200);
}
```

---

### 🕒 第 3 小時：產線安燈系統（Andon）模擬

**情境故事**：Andon（安燈系統）是精實生產（Lean Manufacturing）中著名的異常呼叫看板，當產線發生問題時，現場能立即用視覺化方式讓所有人知道問題所在。

#### 實作 7：自訂字元（電池符號）
```cpp
// ============================================
// 實作 7：LCD 自訂字元 - 畫出簡易電池圖示
// LCD 每個字元由 5x8 個像素組成，用 0/1 的 byte 陣列定義形狀
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

// 定義一個簡易電池符號（5 欄 x 8 列的點陣圖）
byte batteryIcon[8] = {
  B01110,
  B11011,
  B10001,
  B10001,
  B10001,
  B10001,
  B10001,
  B11111
};

void setup() {
  lcd.begin(16, 2);
  lcd.createChar(0, batteryIcon);  // 將圖案註冊為編號 0 的自訂字元
  lcd.print("Battery: ");
  lcd.write(byte(0));              // 顯示編號 0 的自訂字元
}

void loop() {
}
```

#### 實作 8：文字跑馬燈提醒功能
```cpp
// ============================================
// 實作 8：警報發生時，文字左右滑動提醒（跑馬燈效果）
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
String message = "  WARNING: HIGH TEMPERATURE DETECTED  ";

void setup() {
  lcd.begin(16, 2);
}

void loop() {
  // lcd.scrollDisplayLeft() 讓整個畫面向左捲動一格
  for (int i = 0; i < message.length(); i++) {
    lcd.setCursor(0, 0);
    lcd.print(message.substring(i, i + 16));  // 每次只截取畫面能容納的 16 個字元
    delay(300);
  }
}
```

#### 實作 9：正常顯示良品數，異常鎖定顯示錯誤代碼
```cpp
// ============================================
// 實作 9：完整 Andon 系統 - 正常/異常雙模式顯示
// ============================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(7, 8, 9, 10, 11, 12);
int pinTemp = A2;
int goodCount = 128;         // 假設目前累計良品數（示範用固定值）
float alarmThreshold = 35.0;

float readTemperature() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  return (voltage - 0.5) * 100.0;
}

void setup() {
  lcd.begin(16, 2);
}

void loop() {
  float temp = readTemperature();

  if (temp > alarmThreshold) {
    // 異常狀態：畫面鎖定顯示錯誤代碼，不顯示良品數
    lcd.setCursor(0, 0);
    lcd.print("*** ERROR ***   ");
    lcd.setCursor(0, 1);
    lcd.print("E01: High Temp  ");
  } else {
    // 正常狀態：顯示良品數量
    lcd.setCursor(0, 0);
    lcd.print("Status: Normal  ");
    lcd.setCursor(0, 1);
    lcd.print("Good Qty: ");
    lcd.print(goodCount);
    lcd.print("   ");
  }
  delay(300);
}
```

### 🌶️ 第 11 週進階挑戰題
1. **多頁面儀表板**：用按鈕切換三個頁面：溫度頁、距離頁、良品數頁，每頁各自呈現對應資訊。
2. **自訂溫度計圖示**：設計一個自訂字元的溫度計圖案，依溫度高低顯示不同「液面高度」的圖示（需要多個自訂字元）。
3. **歷史錯誤代碼記錄**：當異常發生時，把當下時間（可用簡單的累加計數器代替真實時間）與錯誤代碼一起印在 Serial Monitor，模擬簡易的錯誤日誌（Log）。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 10 週：連續動力驅動](./week-10.md) | [下一週：第 12 週：人機介面指令輸入 ➡](./week-12.md)
