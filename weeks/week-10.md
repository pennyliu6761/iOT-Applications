[⬅ 回目錄](../README.md) | [⬅ 上一週：第 9 週：空間距離感知](./week-09.md) | [下一週：第 11 週：產線資訊可視化 ➡](./week-11.md)

---

## 第 10 週：連續動力驅動
**新增元件：直流馬達（DC Motor）與 L293D 馬達驅動晶片**

### 📌 學習目標
- 理解為什麼馬達不能直接接在 Arduino 腳位上，需要透過驅動晶片（L293D）。
- 學會控制馬達正轉、反轉、停止。
- 學會用 PWM 控制馬達轉速，並實作緩啟動/緩停止。
- 整合超音波感測器，實作簡易 AGV 自走避障邏輯。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **L293D** 晶片（雙 H 橋馬達驅動 IC）到麵包板，並拖曳一顆 **DC Motor**。
2. L293D 是 16 隻腳的晶片，關鍵接法如下（Tinkercad 元件上有腳位編號標示，建議搭配元件說明面板逐一核對）：
   - **Pin 1（Enable 1）**：接 Arduino PWM 腳位 **~5**（控制轉速）
   - **Pin 2（Input 1）**：接 Arduino **Pin 3**（控制轉向 A）
   - **Pin 7（Input 2）**：接 Arduino **Pin 4**（控制轉向 B）
   - **Pin 3（Output 1）／Pin 6（Output 2）**：接馬達的兩隻接腳
   - **Pin 8（VCC2, 馬達電源）**：接 **5V**（Tinkercad 模擬環境可直接使用 Arduino 5V，真實硬體通常會使用獨立電源）
   - **Pin 16（VCC1, 邏輯電源）**：接 **5V**
   - **Pin 4, 5, 12, 13（GND）**：全部接 **GND**
3. 沿用第 9 週超音波感測器（Trig=Pin 9, Echo=Pin 10）與第 5 週蜂鳴器（Pin 8，如衝突請改接其他腳位如 Pin 12）。

> ⚠️ 教學提醒：L293D 接線是本課程最複雜的一次，建議教師花額外 10-15 分鐘用簡化圖解說明「H橋」的基本概念——兩個輸入腳位決定電流流動方向，進而決定馬達正轉或反轉。

---

### 🕐 第 1 小時：馬達驅動基礎

**情境故事**：輸送帶的核心動力來源就是直流馬達，這一小時我們從最基本的「轉動與停止」開始建立控制邏輯。

#### 實作 1：控制馬達單向旋轉與停止
```cpp
// ============================================
// 實作 1：馬達基礎控制 - 單向轉動與停止
// ============================================

int pinIn1 = 3;   // 對應 L293D Input 1
int pinIn2 = 4;   // 對應 L293D Input 2
int pinEnable = 5; // 對應 L293D Enable（控制轉速，此階段先固定全速）

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
}

void loop() {
  // 正轉：Input1 = HIGH, Input2 = LOW
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
  analogWrite(pinEnable, 255);   // 全速

  delay(2000);

  // 停止：Enable 設為 0
  analogWrite(pinEnable, 0);
  delay(2000);
}
```

#### 實作 2：正反轉切換
```cpp
// ============================================
// 實作 2：前進 2 秒、後退 2 秒
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
}

void loop() {
  // 正轉（前進）
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
  analogWrite(pinEnable, 255);
  delay(2000);

  // 反轉（後退）：交換 Input1、Input2 的高低電位
  digitalWrite(pinIn1, LOW);
  digitalWrite(pinIn2, HIGH);
  analogWrite(pinEnable, 255);
  delay(2000);
}
```

#### 實作 3：按鈕控制正轉、反轉與急煞
```cpp
// ============================================
// 實作 3：三顆按鈕分別控制馬達正轉、反轉、急煞
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;
int pinBtnForward = 7;
int pinBtnBackward = 8;
int pinBtnStop = 12;

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  pinMode(pinBtnForward, INPUT);
  pinMode(pinBtnBackward, INPUT);
  pinMode(pinBtnStop, INPUT);
}

void loop() {
  if (digitalRead(pinBtnForward) == HIGH) {
    digitalWrite(pinIn1, HIGH);
    digitalWrite(pinIn2, LOW);
    analogWrite(pinEnable, 255);
  } else if (digitalRead(pinBtnBackward) == HIGH) {
    digitalWrite(pinIn1, LOW);
    digitalWrite(pinIn2, HIGH);
    analogWrite(pinEnable, 255);
  } else if (digitalRead(pinBtnStop) == HIGH) {
    analogWrite(pinEnable, 0);   // 急停
  }
}
```

---

### 🕑 第 2 小時：無段變速輸送帶

**情境故事**：不同產品需要不同的輸送帶速度，太快容易讓貨物掉落，太慢則影響產能，這一小時練習用旋鈕實現無段調速。

#### 實作 4：PWM 控制馬達轉速
```cpp
// ============================================
// 實作 4：透過 Enable 腳位的 PWM 值控制轉速快慢
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
}

void loop() {
  analogWrite(pinEnable, 100);   // 慢速
  delay(2000);
  analogWrite(pinEnable, 255);   // 快速
  delay(2000);
}
```

#### 實作 5：旋鈕調速輸送帶
```cpp
// ============================================
// 實作 5：可變電阻即時控制輸送帶速度
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;
int pinPot = A0;

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
}

void loop() {
  int rawValue = analogRead(pinPot);
  int speed = map(rawValue, 0, 1023, 0, 255);
  analogWrite(pinEnable, speed);
}
```

#### 實作 6：緩啟動與緩停止
```cpp
// ============================================
// 實作 6：逐漸加速、逐漸減速，減少硬體衝擊
// 工業意義：馬達瞬間全速啟動會產生機械應力，緩啟動能延長設備壽命
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
}

void loop() {
  // 緩啟動：PWM 值由 0 逐漸增加到 255
  for (int speed = 0; speed <= 255; speed++) {
    analogWrite(pinEnable, speed);
    delay(10);
  }

  delay(1000);   // 全速運轉維持 1 秒

  // 緩停止：PWM 值由 255 逐漸減少到 0
  for (int speed = 255; speed >= 0; speed--) {
    analogWrite(pinEnable, speed);
    delay(10);
  }

  delay(1000);   // 停止狀態維持 1 秒
}
```

---

### 🕒 第 3 小時：智慧避障自走車（AGV）基礎

**情境故事**：期末專題進入複雜整合的前哨戰——結合馬達與超音波感測器，做出一台會自動偵測障礙物並減速煞車的簡易 AGV 邏輯（雖然沒有實體車輪，但邏輯完全通用）。

#### 實作 7：整合超音波與馬達（平時全速前進）
```cpp
// ============================================
// 實作 7：AGV 平時全速運轉前進
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;
int pinTrig = 9;
int pinEcho = 10;

void setup() {
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);

  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
  analogWrite(pinEnable, 255);   // 平時全速前進
}

void loop() {
  // 本階段先讓馬達持續全速運轉，下一步驟才加入避障邏輯
}
```

#### 實作 8：遇障礙物自動減速、極近距離緊急煞車
```cpp
// ============================================
// 實作 8：AGV 避障邏輯 - 依距離分三段反應
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;
int pinTrig = 9;
int pinEcho = 10;

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
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
}

void loop() {
  float distance = readDistance();

  if (distance < 5) {
    analogWrite(pinEnable, 0);     // 極近：緊急煞車
  } else if (distance < 20) {
    analogWrite(pinEnable, 100);   // 接近：減速
  } else {
    analogWrite(pinEnable, 255);   // 安全：全速
  }
}
```

#### 實作 9：結合蜂鳴器發出工程車警示音
```cpp
// ============================================
// 實作 9：完整 AGV 避障系統 - 加上聲音警示
// ============================================

int pinIn1 = 3;
int pinIn2 = 4;
int pinEnable = 5;
int pinTrig = 9;
int pinEcho = 10;
int pinBuzzer = 12;

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
  pinMode(pinIn1, OUTPUT);
  pinMode(pinIn2, OUTPUT);
  pinMode(pinEnable, OUTPUT);
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  digitalWrite(pinIn1, HIGH);
  digitalWrite(pinIn2, LOW);
}

void loop() {
  float distance = readDistance();

  if (distance < 5) {
    analogWrite(pinEnable, 0);
    tone(pinBuzzer, 1500);         // 煞車時發出警示音
  } else if (distance < 20) {
    analogWrite(pinEnable, 100);
    tone(pinBuzzer, 800);
    delay(100);
    noTone(pinBuzzer);
    delay(200);                    // 減速時間歇性提示音
  } else {
    analogWrite(pinEnable, 255);
    noTone(pinBuzzer);
  }
}
```

### 🌶️ 第 10 週進階挑戰題
1. **自動迴避轉向**：偵測到極近障礙物時，先反轉 1 秒再改變方向重新前進（模擬簡易自走避障，而非單純停止）。
2. **速度與距離連續映射**：不用三段式判斷，改用 `map()` 讓速度隨距離「連續」變化，而非跳躍式的三段切換。
3. **雙馬達差速轉向（延伸思考）**：如果有兩顆馬達分別控制左右輪，你會如何設計「左轉/右轉」的邏輯？（可先用文字構思，不必實作）

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 9 週：空間距離感知](./week-09.md) | [下一週：第 11 週：產線資訊可視化 ➡](./week-11.md)
