[⬅ 回目錄](../README.md) | [⬅ 上一週：第 2 週：數位輸入與狀態決策](./week-02.md) | [下一週：第 4 週：類比輸入與旋鈕控制 ➡](./week-04.md)

---

## 第 3 週：類比輸出與混色視覺
**新增元件：RGB LED（共陰極）**

### 📌 學習目標
- 理解「數位」（只有 0 或 1）與「類比（PWM）」（0～255 漸變）訊號的差異。
- 學會使用 `analogWrite()`，並認識哪些腳位支援 PWM（腳位編號旁有 `~` 符號）。
- 學習自訂函式（Function）的寫法，封裝重複邏輯。
- 認識 RGB 三原色混色原理。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **RGB LED（共陰極, Common Cathode）** 元件到麵包板。RGB LED 有 4 隻腳：最長的那隻是共同負極（GND），其餘三隻分別對應 R（紅）、G（綠）、B（藍）。
2. 三個顏色腳分別**各自串接一顆 220Ω 電阻**，再接到 Arduino 支援 PWM 的腳位：**R → Pin 9、G → Pin 10、B → Pin 11**（Tinkercad 上這些腳位標示為 `~9`、`~10`、`~11`）。
3. 最長的共同負極腳，直接接到麵包板負極軌，再接回 Arduino GND。
4. 沿用第 1 週的單顆 LED（Pin 13）作為呼吸燈練習用（實作 1-3）。

---

### 🕐 第 1 小時：PWM 脈衝寬度調變（無段調變）

**情境故事**：倉儲夜間巡邏動線的補光燈，如果全亮或全暗切換會太過刺眼或太過死板，主管希望燈光能像家用調光燈一樣「柔和漸變」。

#### 實作 1：認識 analogWrite()
```cpp
// ============================================
// 實作 1：用 PWM 控制 LED 亮度（0~255）
// PWM 原理：讓燈以極快速度閃爍，透過「亮的時間比例」製造出視覺上的亮度感
// ============================================

int pinLed = 9;  // 必須接在標示 ~ 的 PWM 腳位

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  analogWrite(pinLed, 128);  // 128 大約是最大值 255 的一半亮度
}
```

#### 實作 2：手動測試 5 段不同亮度
```cpp
// ============================================
// 實作 2：測試不同亮度數值的視覺效果
// ============================================

int pinLed = 9;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  analogWrite(pinLed, 0);    delay(800);  // 全暗
  analogWrite(pinLed, 64);   delay(800);  // 微亮
  analogWrite(pinLed, 128);  delay(800);  // 中亮
  analogWrite(pinLed, 192);  delay(800);  // 偏亮
  analogWrite(pinLed, 255);  delay(800);  // 全亮
}
```

#### 實作 3：呼吸燈（平滑漸亮漸暗）
```cpp
// ============================================
// 實作 3：呼吸燈效果 - 倉儲巡邏補光燈
// ============================================

int pinLed = 9;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  // 漸亮：從 0 到 255
  for (int brightness = 0; brightness <= 255; brightness++) {
    analogWrite(pinLed, brightness);
    delay(5);  // 每一階段停留 5 毫秒，數值變化夠慢才會呈現平滑感
  }
  // 漸暗：從 255 到 0
  for (int brightness = 255; brightness >= 0; brightness--) {
    analogWrite(pinLed, brightness);
    delay(5);
  }
}
```

---

### 🕑 第 2 小時：RGB LED 混色原理（狀態指示燈）

**情境故事**：產線需要一套「單一顆燈、多種顏色」的狀態指示系統，取代過去要裝三顆不同顏色燈泡的作法，降低硬體成本。

#### 實作 4：分別控制 R、G、B 腳位
```cpp
// ============================================
// 實作 4：測試 RGB LED 三個顏色腳位
// ============================================

int pinR = 9;
int pinG = 10;
int pinB = 11;

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
}

void loop() {
  analogWrite(pinR, 255); analogWrite(pinG, 0);   analogWrite(pinB, 0);   delay(1000); // 紅
  analogWrite(pinR, 0);   analogWrite(pinG, 255); analogWrite(pinB, 0);   delay(1000); // 綠
  analogWrite(pinR, 0);   analogWrite(pinG, 0);   analogWrite(pinB, 255); delay(1000); // 藍
}
```

#### 實作 5：自訂函式 setColor(r, g, b)
```cpp
// ============================================
// 實作 5：把三行 analogWrite 封裝成一個函式
// 好處：往後只要呼叫 setColor(255, 0, 0) 就能設定紅色，程式碼更好讀
// ============================================

int pinR = 9;
int pinG = 10;
int pinB = 11;

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
}

// 自訂函式：輸入紅、綠、藍三個數值，直接設定 RGB LED 顏色
void setColor(int r, int g, int b) {
  analogWrite(pinR, r);
  analogWrite(pinG, g);
  analogWrite(pinB, b);
}

void loop() {
  setColor(255, 0, 0);   delay(1000);  // 紅
  setColor(0, 255, 0);   delay(1000);  // 綠
  setColor(0, 0, 255);   delay(1000);  // 藍
  setColor(255, 255, 0); delay(1000);  // 黃（紅+綠）
}
```

#### 實作 6：按鈕切換產線五種狀態顏色
```cpp
// ============================================
// 實作 6：按鈕每按一次，切換到下一個產線狀態顏色
// 紅→綠→藍→黃→紫，循環顯示
// ============================================

int pinR = 9, pinG = 10, pinB = 11;
int pinButton = 7;
int lastButtonState = LOW;
int stateIndex = 0;  // 目前是第幾個狀態（0~4）

void setColor(int r, int g, int b) {
  analogWrite(pinR, r);
  analogWrite(pinG, g);
  analogWrite(pinB, b);
}

void setup() {
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
  pinMode(pinButton, INPUT);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);  // 防彈跳
    stateIndex = (stateIndex + 1) % 5;  // % 取餘數，讓數字在 0~4 之間循環
  }
  lastButtonState = currentState;

  // 依照目前狀態編號，顯示對應顏色
  if (stateIndex == 0) setColor(255, 0, 0);     // 紅：待機
  else if (stateIndex == 1) setColor(0, 255, 0);   // 綠：正常運轉
  else if (stateIndex == 2) setColor(0, 0, 255);   // 藍：保養中
  else if (stateIndex == 3) setColor(255, 255, 0); // 黃：警戒
  else if (stateIndex == 4) setColor(128, 0, 128); // 紫：異常停機
}
```

---

### 🕒 第 3 小時：視覺化動態特效

**情境故事**：主管希望「狀態指示燈」不只是靜態顏色，異常狀態時要有明顯的動態警示效果，才能在嘈雜的產線環境中第一時間吸引作業員注意。

#### 實作 7：RGB 單色呼吸燈
```cpp
// ============================================
// 實作 7：把呼吸燈效果套用在 RGB LED 的其中一色
// ============================================

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
  for (int i = 0; i <= 255; i++) { setColor(0, i, 0); delay(4); }  // 綠色漸亮
  for (int i = 255; i >= 0; i--) { setColor(0, i, 0); delay(4); } // 綠色漸暗
}
```

#### 實作 8：三色平滑過渡（紅→綠→藍）
```cpp
// ============================================
// 實作 8：顏色之間平滑交叉淡入淡出
// ============================================

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
  // 紅 → 綠：紅逐漸減弱，綠逐漸增強
  for (int i = 0; i <= 255; i++) {
    setColor(255 - i, i, 0);
    delay(6);
  }
  // 綠 → 藍
  for (int i = 0; i <= 255; i++) {
    setColor(0, 255 - i, i);
    delay(6);
  }
}
```

#### 實作 9：警報閃爍模式（紅藍快速交替）
```cpp
// ============================================
// 實作 9：機台當機警報 - 模擬警車燈紅藍交替閃爍
// ============================================

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
  setColor(255, 0, 0);  // 紅
  delay(150);
  setColor(0, 0, 255);  // 藍
  delay(150);
}
```

### 🌶️ 第 3 週進階挑戰題
1. **彩虹循環（Color Wheel）**：撰寫一個函式讓 RGB LED 呈現連續的彩虹漸變效果（提示：可分成紅→黃→綠→青→藍→紫→紅六個階段）。
2. **依狀態決定閃爍速度**：正常運轉時呼吸燈速度慢，警戒時中等速度，異常時快速閃爍——用同一顆 RGB LED 呈現三種急迫程度。
3. **雙態警示器**：異常時 RGB LED 閃爍的同時，讓第 1 週的白色 LED 也同步閃爍，模擬「聲光連動」（先跳過蜂鳴器，用 LED 代替聲音提示）。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 2 週：數位輸入與狀態決策](./week-02.md) | [下一週：第 4 週：類比輸入與旋鈕控制 ➡](./week-04.md)
