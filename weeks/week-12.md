[⬅ 回目錄](../README.md) | [⬅ 上一週：第 11 週：產線資訊可視化](./week-11.md) | [下一週：第 13 週：全彩視覺化看板 ➡](./week-13.md)

---

## 第 12 週：人機介面指令輸入
**新增元件：4x4 薄膜矩陣鍵盤（Keypad）**

### 📌 學習目標
- 理解矩陣鍵盤的「行列掃描」原理：如何用 8 隻腳位讀取 16 個按鍵。
- 學會使用 `<Keypad.h>` 函式庫。
- 練習字元組合與字串比對邏輯，實作簡易電子密碼鎖。
- 建立「選單層級（Menu）」的設計概念，為期末專題做準備。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **Keypad 4x4** 元件到工作區，共有 8 隻接腳（4 條行線 Row + 4 條列線 Column）。
2. 依序接到 Arduino 的 **Pin 2, 3, 4, 5**（列, Row）與 **Pin 6, A0, A1, A2**（行, Column，類比腳位在此可當一般數位腳位使用）。實際腳位對應請依 Tinkercad 元件標示為準，並在程式碼中對應設定。
3. 沿用第 11 週 LCD（RS=7 需與 Keypad 錯開，建議 LCD 改接 **RS=8, E=9, D4=10, D5=11, D6=12, D7=13**）、第 7 週伺服馬達（Pin A3 或其他未使用腳位）、第 5 週蜂鳴器。

> ⚠️ 教學提醒：本週腳位使用量最大，強烈建議提供「已完成接線」的 Tinkercad 範本電路，讓學生專注在程式邏輯與鍵盤 + LCD 的互動設計。

---

### 🕐 第 1 小時：矩陣掃描原理與實作

**情境故事**：產線設備需要一個能輸入數字指令的操作面板，取代單純的按鈕，讓現場人員能輸入更複雜的參數或密碼。

#### 實作 1：讀取按鍵字元並印到 Serial
```cpp
// ============================================
// 實作 1：矩陣鍵盤基礎讀取
// ============================================

#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;

// 定義 4x4 鍵盤的按鍵配置
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {2, 3, 4, 5};   // 連接 4 條列線的腳位
byte colPins[COLS] = {6, A0, A1, A2}; // 連接 4 條行線的腳位

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

void setup() {
  Serial.begin(9600);
}

void loop() {
  char key = keypad.getKey();  // 讀取目前被按下的按鍵，若無按鍵則回傳空值

  if (key) {   // 如果有讀到按鍵（非空值）
    Serial.println(key);
  }
}
```

#### 實作 2：結合 LCD 即時顯示按下的字元
```cpp
// ============================================
// 實作 2：打字機效果 - 按下的字元即時顯示在 LCD 上
// ============================================

#include <Keypad.h>
#include <LiquidCrystal.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

LiquidCrystal lcd(8, 9, 10, 11, 12, 13);

void setup() {
  lcd.begin(16, 2);
}

void loop() {
  char key = keypad.getKey();

  if (key) {
    lcd.print(key);   // 每按一次，就在 LCD 目前游標位置印出該字元
  }
}
```

#### 實作 3：產線模式選擇器
```cpp
// ============================================
// 實作 3：按 'A' 啟動全速、'B' 半速、'C' 停止
// ============================================

#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

void setup() {
  Serial.begin(9600);
}

void loop() {
  char key = keypad.getKey();

  if (key == 'A') {
    Serial.println("模式：全速運轉");
  } else if (key == 'B') {
    Serial.println("模式：半速運轉");
  } else if (key == 'C') {
    Serial.println("模式：停止");
  }
}
```

---

### 🕑 第 2 小時：字元組合與邏輯驗證

**情境故事**：單一字元的指令太簡單，真實的密碼鎖需要輸入「一組」數字才能驗證，這一小時我們學習如何把多個字元組合成一個字串進行比對。

#### 實作 4：組合輸入 4 個字元
```cpp
// ============================================
// 實作 4：連續輸入 4 個字元並儲存於字串
// ============================================

#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

String inputCode = "";   // 用來累積使用者輸入的字元

void setup() {
  Serial.begin(9600);
}

void loop() {
  char key = keypad.getKey();

  if (key) {
    inputCode = inputCode + key;   // 把新字元接在字串後面

    if (inputCode.length() == 4) {   // 累積滿 4 位數
      Serial.print("你輸入的密碼是：");
      Serial.println(inputCode);
      inputCode = "";   // 清空，準備下一次輸入
    }
  }
}
```

#### 實作 5：密碼比對邏輯
```cpp
// ============================================
// 實作 5：檢查輸入密碼是否正確
// ============================================

#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

String inputCode = "";
String correctPassword = "1234";

void setup() {
  Serial.begin(9600);
}

void loop() {
  char key = keypad.getKey();

  if (key) {
    inputCode = inputCode + key;

    if (inputCode.length() == 4) {
      if (inputCode == correctPassword) {
        Serial.println("密碼正確！");
      } else {
        Serial.println("密碼錯誤！");
      }
      inputCode = "";
    }
  }
}
```

#### 實作 6：電子密碼鎖（伺服馬達開門）
```cpp
// ============================================
// 實作 6：完整電子密碼鎖 - 正確開門，錯誤響蜂鳴器
// ============================================

#include <Keypad.h>
#include <Servo.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

Servo lockServo;
int pinBuzzer = 13;
String inputCode = "";
String correctPassword = "1234";

void setup() {
  lockServo.attach(A3);
  lockServo.write(0);   // 初始：鎖定
}

void loop() {
  char key = keypad.getKey();

  if (key) {
    inputCode = inputCode + key;

    if (inputCode.length() == 4) {
      if (inputCode == correctPassword) {
        lockServo.write(90);   // 開門
        delay(3000);
        lockServo.write(0);    // 自動再次上鎖
      } else {
        tone(pinBuzzer, 1500);
        delay(500);
        noTone(pinBuzzer);
      }
      inputCode = "";
    }
  }
}
```

---

### 🕒 第 3 小時：工業設備參數設定介面

**情境故事**：真正的工業設備通常有「設定模式」，讓維修人員能修改警報門檻等參數，這一小時我們模擬這種「按 * 進入設定、按 # 確認」的介面設計模式。

#### 實作 7：進入設定模式輸入溫度門檻
```cpp
// ============================================
// 實作 7：按 '*' 進入設定模式，輸入兩位數設定溫度門檻
// ============================================

#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

bool settingMode = false;
String tempInput = "";

void setup() {
  Serial.begin(9600);
}

void loop() {
  char key = keypad.getKey();

  if (key == '*') {
    settingMode = true;
    tempInput = "";
    Serial.println("進入設定模式，請輸入兩位數溫度門檻");
  } else if (settingMode && key >= '0' && key <= '9') {
    tempInput = tempInput + key;
    Serial.print("目前輸入：");
    Serial.println(tempInput);
  }
}
```

#### 實作 8：按 # 確認並更新系統門檻
```cpp
// ============================================
// 實作 8：按 '#' 確認輸入，正式套用新門檻值
// ============================================

#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

bool settingMode = false;
String tempInput = "";
int alarmThreshold = 30;   // 系統目前使用的溫度門檻

void setup() {
  Serial.begin(9600);
}

void loop() {
  char key = keypad.getKey();

  if (key == '*') {
    settingMode = true;
    tempInput = "";
    Serial.println("進入設定模式");
  } else if (key == '#' && settingMode) {
    alarmThreshold = tempInput.toInt();   // 把字串轉換成整數
    Serial.print("新門檻已套用：");
    Serial.println(alarmThreshold);
    settingMode = false;
  } else if (settingMode && key >= '0' && key <= '9') {
    tempInput = tempInput + key;
  }
}
```

#### 實作 9：整合 LCD 做出具備選單層級的設定機台雛形
```cpp
// ============================================
// 實作 9：期末專題前哨戰 - 完整選單層級介面
// 平時顯示狀態，進入設定模式後顯示輸入過程，確認後顯示套用結果
// ============================================

#include <Keypad.h>
#include <LiquidCrystal.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};
byte colPins[COLS] = {6, A0, A1, A2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

LiquidCrystal lcd(8, 9, 10, 11, 12, 13);

bool settingMode = false;
String tempInput = "";
int alarmThreshold = 30;

void setup() {
  lcd.begin(16, 2);
  lcd.print("Threshold:");
  lcd.setCursor(0, 1);
  lcd.print(alarmThreshold);
}

void loop() {
  char key = keypad.getKey();

  if (key == '*') {
    settingMode = true;
    tempInput = "";
    lcd.clear();
    lcd.print("Setting Mode:");
  } else if (key == '#' && settingMode) {
    alarmThreshold = tempInput.toInt();
    settingMode = false;
    lcd.clear();
    lcd.print("Threshold:");
    lcd.setCursor(0, 1);
    lcd.print(alarmThreshold);
  } else if (settingMode && key >= '0' && key <= '9') {
    tempInput = tempInput + key;
    lcd.setCursor(0, 1);
    lcd.print(tempInput);
  }
}
```

### 🌶️ 第 12 週進階挑戰題
1. **密碼錯誤鎖定機制**：連續輸入錯誤密碼 3 次，鎖定鍵盤 10 秒（結合 `millis()` 概念，可先用簡化的計數方式實作）。
2. **多層選單設計**：設計「主選單 → 溫度設定 / 速度設定 / 密碼設定」三個子選單，用 'A'、'B'、'C' 進入不同子選單。
3. **輸入退格功能**：按下 'D' 鍵時，能刪除最後輸入的一個字元（提示：`String` 有 `substring()` 方法可以擷取部分字串）。

---

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 11 週：產線資訊可視化](./week-11.md) | [下一週：第 13 週：全彩視覺化看板 ➡](./week-13.md)
