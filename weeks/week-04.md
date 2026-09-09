[⬅ 回目錄](../README.md) | [⬅ 上一週：第 3 週：類比輸出與混色視覺](./week-03.md) | [下一週：第 5 週：聽覺反饋與環境光感知 ➡](./week-05.md)

---

## 第 4 週：類比輸入與旋鈕控制
**新增元件：可變電阻（Potentiometer）**

### 📌 學習目標
- 理解 ADC（類比轉數位）概念：`analogRead()` 讀取範圍為 0～1023。
- 學會使用 `map()` 函式做數值範圍轉換，這是工業感測資料處理的核心技巧。
- 用 Serial Plotter 觀察連續數值的變化波形。
- 整合前三週元件，訓練學生「多元件協同運作」的系統思維。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **Potentiometer（可變電阻，旋鈕造型）** 到工作區。
2. 可變電阻有 3 隻腳：左右兩腳分別接 **5V** 與 **GND**（方向不影響邏輯，只影響轉動方向對應的數值增減）。
3. 中間腳（訊號輸出腳）接到 Arduino 的 **類比輸入腳 A0**（類比腳位不需要加 `~`，是獨立的一組腳位，通常標示 A0～A5）。
4. 沿用第 1 週的單顆 LED（Pin 13）與第 3 週的 RGB LED（Pin 9/10/11），第 3 小時會加入第 3 週跑馬燈的 5 顆 LED 與第 2 週的按鈕。

---

### 🕐 第 1 小時：ADC 類比數位轉換（讀取旋鈕值）

**情境故事**：產線上的「進料速度旋鈕」讓現場作業員能手動微調輸送帶速度，這是最基礎的人機介面（HMI）雛形。

#### 實作 1：讀取可變電阻數值
```cpp
// ============================================
// 實作 1：讀取旋鈕的類比數值（0~1023）
// ============================================

int pinPot = A0;  // 類比輸入腳位

void setup() {
  Serial.begin(9600);
}

void loop() {
  int value = analogRead(pinPot);  // 讀取範圍：0（轉到底一邊）~ 1023（轉到底另一邊）
  Serial.println(value);
  delay(100);
}
```

#### 實作 2：用 Serial Plotter 觀察波形
```cpp
// ============================================
// 實作 2：程式碼不變，改用 Serial Plotter 視覺化觀察
// 操作：Tinkercad 右側切換 Serial Monitor → Serial Plotter
// ============================================

int pinPot = A0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int value = analogRead(pinPot);
  Serial.println(value);
  delay(50);
}
```
**教學重點**：緩慢轉動旋鈕，觀察 Plotter 畫出平滑的曲線；快速轉動，觀察數值跳動的樣子。這能建立學生對「連續類比訊號」的直覺。

#### 實作 3：數值門檻判斷
```cpp
// ============================================
// 實作 3：旋鈕轉超過一半（>512）才啟動指示燈
// 情境：速度旋鈕轉超過中位數，代表產線進入「高速模式」
// ============================================

int pinPot = A0;
int pinLed = 13;

void setup() {
  pinMode(pinLed, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int value = analogRead(pinPot);

  if (value > 512) {
    digitalWrite(pinLed, HIGH);  // 高速模式指示燈亮起
  } else {
    digitalWrite(pinLed, LOW);
  }

  Serial.println(value);
  delay(50);
}
```

---

### 🕑 第 2 小時：數值映射與即時連動

**情境故事**：旋鈕讀到的是 0~1023 這種不直覺的數字，但 LED 亮度只吃 0~255，甚至馬達轉速可能要用「毫秒」單位。工業上經常需要把感測器的原始數值「轉換」成設備能理解的單位，這就是 `map()` 函式存在的意義。

#### 實作 4：介紹 map() 函式
```cpp
// ============================================
// 實作 4：認識 map() - 數值範圍轉換函式
// 語法：map(原始值, 原始最小, 原始最大, 目標最小, 目標最大)
// ============================================

int pinPot = A0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int rawValue = analogRead(pinPot);              // 原始範圍 0~1023
  int mapped = map(rawValue, 0, 1023, 0, 255);    // 轉換為 0~255

  Serial.print("原始值: ");
  Serial.print(rawValue);
  Serial.print("  轉換後: ");
  Serial.println(mapped);

  delay(100);
}
```

#### 實作 5：旋鈕即時無段控制 LED 亮度
```cpp
// ============================================
// 實作 5：旋鈕即時調光 - 手動調光台燈概念
// ============================================

int pinPot = A0;
int pinLed = 9;  // 注意：調光需要 PWM 腳位

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int rawValue = analogRead(pinPot);
  int brightness = map(rawValue, 0, 1023, 0, 255);
  analogWrite(pinLed, brightness);
}
```

#### 實作 6：旋鈕控制跑馬燈延遲時間（調速旋鈕）
```cpp
// ============================================
// 實作 6：輸送帶調速旋鈕 - 控制跑馬燈的移動速度
// ============================================

int pinPot = A0;
int ledPins[] = {2, 3, 4, 5, 6};
int numLeds = 5;

void setup() {
  for (int i = 0; i < numLeds; i++) {
    pinMode(ledPins[i], OUTPUT);
  }
}

void loop() {
  int rawValue = analogRead(pinPot);
  // 轉速旋鈕：轉越多，delay 越短，跑馬燈移動越快（範圍 20~300 毫秒）
  int speedDelay = map(rawValue, 0, 1023, 20, 300);

  for (int i = 0; i < numLeds; i++) {
    digitalWrite(ledPins[i], HIGH);
    delay(speedDelay);
    digitalWrite(ledPins[i], LOW);
  }
}
```

---

### 🕒 第 3 小時：多區間判斷（儀表板燈號）

**情境故事**：期中專題前的整合演練——把旋鈕想像成「產線負載感測器」，依照負載高低顯示不同燈號，並整合前幾週學過的所有元件。

#### 實作 7：三區間負載指示燈
```cpp
// ============================================
// 實作 7：低中高負載三段指示（模擬產能負載監控）
// ============================================

int pinPot = A0;
int pinLow = 2;    // 低負載燈（綠）
int pinMid = 3;    // 中負載燈（黃）
int pinHigh = 4;   // 高負載燈（紅）

void setup() {
  pinMode(pinLow, OUTPUT);
  pinMode(pinMid, OUTPUT);
  pinMode(pinHigh, OUTPUT);
}

void loop() {
  int value = analogRead(pinPot);

  // 先全部熄滅，再依區間點亮對應的燈
  digitalWrite(pinLow, LOW);
  digitalWrite(pinMid, LOW);
  digitalWrite(pinHigh, LOW);

  if (value < 341) {
    digitalWrite(pinLow, HIGH);        // 0~340：低負載
  } else if (value < 682) {
    digitalWrite(pinMid, HIGH);        // 341~681：中負載
  } else {
    digitalWrite(pinHigh, HIGH);       // 682~1023：高負載
  }
}
```

#### 實作 8：旋鈕與 RGB LED 連動改變光色
```cpp
// ============================================
// 實作 8：旋鈕連續控制 RGB LED 顏色（從綠到紅漸變）
// 情境：負載儀表以顏色連續漸變呈現，比三顆分開的燈更直覺
// ============================================

int pinPot = A0;
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
  int value = analogRead(pinPot);
  int red   = map(value, 0, 1023, 0, 255);    // 負載越高，紅色越強
  int green = map(value, 0, 1023, 255, 0);    // 負載越高，綠色越弱
  setColor(red, green, 0);
}
```

#### 實作 9：雙條件整合（旋鈕調亮度 + 按鈕開關）
```cpp
// ============================================
// 實作 9：期中週前置整合練習
// 按鈕控制總開關，旋鈕控制亮度（兩者需同時滿足才會有輸出）
// ============================================

int pinPot = A0;
int pinButton = 7;
int pinLed = 9;

int lastButtonState = LOW;
bool machineOn = false;

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int currentState = digitalRead(pinButton);
  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);
    machineOn = !machineOn;   // 按鈕切換總開關狀態
  }
  lastButtonState = currentState;

  if (machineOn) {
    int rawValue = analogRead(pinPot);
    int brightness = map(rawValue, 0, 1023, 0, 255);
    analogWrite(pinLed, brightness);   // 開機時才受旋鈕控制亮度
  } else {
    analogWrite(pinLed, 0);            // 關機時強制熄滅，旋鈕無效
  }
}
```

### 🌶️ 第 4 週進階挑戰題
1. **五段負載儀表**：把三區間擴充為五區間，並讓 5 顆 LED 跑馬燈組（第 1 週電路）依旋鈕數值「點亮對應數量」的燈（提示：這其實就是簡易長條圖／progress bar 的原理）。
2. **旋鈕控制 RGB 循環速度**：旋鈕數值越大，第 3 週的呼吸燈效果速度越快。
3. **反向映射練習**：嘗試把 `map()` 的目標範圍寫成反向（如 `map(value, 0, 1023, 255, 0)`），觀察並解釋輸出結果為何相反。

---

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 3 週：類比輸出與混色視覺](./week-03.md) | [下一週：第 5 週：聽覺反饋與環境光感知 ➡](./week-05.md)
