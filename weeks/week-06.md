[⬅ 回目錄](../README.md) | [⬅ 上一週：第 5 週：聽覺反饋與環境光感知](./week-05.md) | [下一週：第 7 週：精準角度控制 ➡](./week-07.md)

---

## 第 6 週：溫度監控與安全防護
**新增元件：TMP36 溫度感測器**

### 📌 學習目標
- 學會將感測器讀值透過數學公式轉換為實際物理單位（電壓 → 攝氏溫度）。
- 建立多段溫度狀態指示邏輯。
- 認識工業控制中重要的「警報鎖定（Latching）」與「人工復歸（Reset）」設計理念。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **TMP36** 溫度感測器（外觀類似電晶體，三隻腳）到麵包板。
2. 面對感測器平面朝向自己時，由左至右依序為：**電源腳（5V）、訊號輸出腳（接 A2）、接地腳（GND）**（實際腳位排列請以 Tinkercad 元件說明為準，滑鼠移到元件上通常會顯示腳位標籤）。
3. 沿用第 3 週 RGB LED（Pin 9/10/11）、第 5 週蜂鳴器（Pin 8）、第 2 週按鈕（Pin 7）。

---

### 🕐 第 1 小時：溫度訊號轉換（數學運算）

**情境故事**：冷藏物流倉儲需要 24 小時監控溫度，一旦溫度異常上升就必須立即示警，避免生鮮商品損壞造成公司損失。

#### 實作 1：讀取並轉換為攝氏溫度
```cpp
// ============================================
// 實作 1：TMP36 感測器讀值轉換公式
// TMP36 特性：攝氏 0 度時輸出 0.5V，之後每上升 1 度輸出增加 10mV
// ============================================

int pinTemp = A2;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int rawValue = analogRead(pinTemp);            // 讀取 0~1023
  float voltage = rawValue * (5.0 / 1023.0);     // 轉換為實際電壓（0~5V）
  float celsius = (voltage - 0.5) * 100.0;       // 套用 TMP36 轉換公式

  Serial.print("溫度: ");
  Serial.print(celsius);
  Serial.println(" °C");
  delay(500);
}
```
**教學重點**：這裡首次出現 `float`（浮點數）型別，因為溫度不會是整數。可以簡單提及 `int` 只能存整數，計算溫度這種有小數點的資料需要用 `float`。

#### 實作 2：手指觸碰感測器觀察溫度上升
```cpp
// ============================================
// 實作 2：程式碼相同，操作 Tinkercad 模擬環境的「溫度滑桿」
// 觀察數值即時變化，建立感測器與現實世界的連結感
// ============================================

int pinTemp = A2;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  Serial.println(celsius);
  delay(300);
}
```
**Tinkercad 操作提醒**：點擊 TMP36 元件，可以直接拖曳「溫度」滑桿模擬真實環境溫度變化，不需要真的用手指觸摸感測器。

#### 實作 3：冷藏庫超溫警報（雙態指示）
```cpp
// ============================================
// 實作 3：溫度超過門檻亮紅燈，否則亮綠燈
// ============================================

int pinTemp = A2;
int pinRed = 2;
int pinGreen = 3;
float threshold = 8.0;  // 冷藏庫警戒溫度 8°C

void setup() {
  pinMode(pinRed, OUTPUT);
  pinMode(pinGreen, OUTPUT);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > threshold) {
    digitalWrite(pinRed, HIGH);
    digitalWrite(pinGreen, LOW);
  } else {
    digitalWrite(pinRed, LOW);
    digitalWrite(pinGreen, HIGH);
  }
}
```

---

### 🕑 第 2 小時：進階溫控邏輯

**情境故事**：品保主管要求不能只有「正常/異常」兩段，而要有「正常、警戒、危險」三段式管理，讓現場人員能提早介入處理，而非等到真正出事才反應。

#### 實作 4：三段溫度指示（RGB 呈現）
```cpp
// ============================================
// 實作 4：三段溫度狀態 - 正常(綠)、警戒(黃)、危險(紅)
// ============================================

int pinTemp = A2;
int pinR = 9, pinG = 10, pinB = 11;

void setColor(int r, int g, int b) {
  analogWrite(pinR, r);
  analogWrite(pinG, g);
  analogWrite(pinB, b);
}

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius < 25.0) {
    setColor(0, 255, 0);         // 正常：綠燈
  } else if (celsius < 32.0) {
    setColor(255, 255, 0);       // 警戒：黃燈
  } else {
    setColor(255, 0, 0);         // 危險：紅燈
  }
}
```

#### 實作 5：高溫警報器（聲光連動）
```cpp
// ============================================
// 實作 5：過熱時亮紅燈並且蜂鳴器發出高頻警鈴
// ============================================

int pinTemp = A2;
int pinR = 9, pinG = 10, pinB = 11;
int pinBuzzer = 8;

void setColor(int r, int g, int b) {
  analogWrite(pinR, r);
  analogWrite(pinG, g);
  analogWrite(pinB, b);
}

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius < 25.0) {
    setColor(0, 255, 0);
    noTone(pinBuzzer);
  } else if (celsius < 32.0) {
    setColor(255, 255, 0);
    noTone(pinBuzzer);
  } else {
    setColor(255, 0, 0);
    tone(pinBuzzer, 2000);       // 危險狀態才響鈴
  }
}
```

#### 實作 6：可調式溫度門檻（旋鈕設定）
```cpp
// ============================================
// 實作 6：讓使用者用旋鈕手動設定「警報觸發溫度」
// 情境：不同冷藏商品有不同保存溫度要求，門檻需要能現場調整
// ============================================

int pinTemp = A2;
int pinPot = A0;
int pinR = 9, pinG = 10, pinB = 11;
int pinBuzzer = 8;

void setColor(int r, int g, int b) {
  analogWrite(pinR, r);
  analogWrite(pinG, g);
  analogWrite(pinB, b);
}

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  // 旋鈕設定門檻範圍：0~50°C
  int potValue = analogRead(pinPot);
  float threshold = map(potValue, 0, 1023, 0, 50);

  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > threshold) {
    setColor(255, 0, 0);
    tone(pinBuzzer, 2000);
  } else {
    setColor(0, 255, 0);
    noTone(pinBuzzer);
  }

  Serial.print("目前門檻: ");
  Serial.print(threshold);
  Serial.print("  目前溫度: ");
  Serial.println(celsius);
}
```

---

### 🕒 第 3 小時：狀態鎖定與解除（工業機台常見邏輯）

**情境故事**：真實工廠中，警報響起後即使問題瞬間消失，管理規範通常要求「必須有人到場確認並手動解除」，避免問題被輕忽——這是工安管理中「事件必須被記錄與處理」的重要精神。

#### 實作 7：警報鎖定（Latching）
```cpp
// ============================================
// 實作 7：一旦超溫觸發警報，即使溫度下降，警報仍持續響
// 關鍵觀念：用一個變數「鎖住」異常狀態，不隨感測值即時變動
// ============================================

int pinTemp = A2;
int pinBuzzer = 8;
float threshold = 30.0;
bool alarmLatched = false;   // 警報鎖定旗標

void setup() {
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > threshold) {
    alarmLatched = true;   // 一旦超過門檻，永久鎖定為 true（直到程式重置前不會自動變回 false）
  }

  if (alarmLatched) {
    tone(pinBuzzer, 2000);
  } else {
    noTone(pinBuzzer);
  }
}
```

#### 實作 8：加入復歸按鈕（Reset）
```cpp
// ============================================
// 實作 8：必須人工按下 Reset 鈕才能解除警報鎖定
// ============================================

int pinTemp = A2;
int pinBuzzer = 8;
int pinReset = 7;
float threshold = 30.0;
bool alarmLatched = false;

void setup() {
  pinMode(pinReset, INPUT);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > threshold) {
    alarmLatched = true;
  }

  // 人工復歸：只有按下 Reset 鈕，且此時溫度已經恢復正常，才允許解除警報
  if (digitalRead(pinReset) == HIGH && celsius <= threshold) {
    alarmLatched = false;
  }

  if (alarmLatched) {
    tone(pinBuzzer, 2000);
  } else {
    noTone(pinBuzzer);
  }
}
```
**教學討論重點**：為什麼 Reset 邏輯要加上「且溫度已恢復正常」這個條件？如果不加會發生什麼問題？（引導學生思考：若不加，代表可以在異常狀態下強制消音卻不解決問題）

#### 實作 9：完整溫度異常停機模擬（整合演練）
```cpp
// ============================================
// 實作 9：完整整合 - 聲光警報 + Reset 機制
// 情境：冷藏庫溫控系統完整演練，作為期中專題前的收斂練習
// ============================================

int pinTemp = A2;
int pinR = 9, pinG = 10, pinB = 11;
int pinBuzzer = 8;
int pinReset = 7;
float threshold = 30.0;
bool alarmLatched = false;

void setColor(int r, int g, int b) {
  analogWrite(pinR, r);
  analogWrite(pinG, g);
  analogWrite(pinB, b);
}

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
  pinMode(pinReset, INPUT);
  Serial.begin(9600);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  if (celsius > threshold) {
    alarmLatched = true;
  }
  if (digitalRead(pinReset) == HIGH && celsius <= threshold) {
    alarmLatched = false;
    Serial.println("警報已人工解除");
  }

  if (alarmLatched) {
    setColor(255, 0, 0);
    tone(pinBuzzer, 2000);
  } else {
    setColor(0, 255, 0);
    noTone(pinBuzzer);
  }
}
```

### 🌶️ 第 6 週進階挑戰題
1. **溫度記錄器**：新增一個變數記錄「歷史最高溫度」，即使目前溫度下降，仍持續顯示曾經出現過的最高值。
2. **雙門檻警報（分級鎖定）**：分別設定「警戒鎖定」與「危險鎖定」兩種等級，各自獨立記錄與解除。
3. **低溫異常判斷**：目前只判斷「過熱」，能否加入「過冷」判斷（例如低於 -5°C 也要警報，模擬冷凍設備故障失溫）？

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 5 週：聽覺反饋與環境光感知](./week-05.md) | [下一週：第 7 週：精準角度控制 ➡](./week-07.md)
