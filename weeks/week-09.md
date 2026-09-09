[⬅ 回目錄](../README.md) | [⬅ 上一週：第 8 週：期中專題 — 智慧倉儲環境監控站](./week-08.md) | [下一週：第 10 週：連續動力驅動 ➡](./week-10.md)

---

# 第三階段：工業物聯網基礎元件（第 9～12 週）

**階段目標**：處理較複雜的通訊介面與動力驅動，模擬無人搬運車（AGV）與產線安燈看板（Andon），讓學生體會工業物聯網系統的實際樣貌。

---


## 第 9 週：空間距離感知
**新增元件：超音波測距模組（HC-SR04）**

### 📌 學習目標
- 理解超音波測距原理：發射脈衝、接收回波、換算時間差為距離。
- 學會使用 `pulseIn()` 函式讀取脈衝時間。
- 練習將距離數值轉換為分級警報邏輯，這是 AGV 避障系統的核心技術。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **Ultrasonic Distance Sensor（HC-SR04）** 到工作區，模組有 4 隻腳：**VCC、Trig、Echo、GND**。
2. VCC 接 **5V**，GND 接 **GND**。
3. Trig 接 Arduino **Pin 9**（負責發射超音波脈衝）。
4. Echo 接 Arduino **Pin 10**（負責接收反射回波，計算時間差）。
5. 沿用第 3 週 RGB LED、第 5 週蜂鳴器、第 7 週伺服馬達。

---

### 🕐 第 1 小時：超音波原理與距離運算

**情境故事**：AGV（自動搬運車）在倉儲走道間穿梭，必須即時偵測前方是否有障礙物，避免撞上貨架或人員，這是自動化物流系統最基本也最關鍵的安全機制。

#### 實作 1：發射脈衝與讀取回波時間
```cpp
// ============================================
// 實作 1：超音波感測器基礎讀取
// 原理：Trig 發出短脈衝聲波，聲波遇到物體反彈回來被 Echo 接收
//      聲波來回所花的「時間」，就能反推出「距離」
// ============================================

int pinTrig = 9;
int pinEcho = 10;

void setup() {
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  Serial.begin(9600);
}

void loop() {
  // 發射一個短脈衝訊號
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);   // 標準規格：Trig 需維持 10 微秒的高電位
  digitalWrite(pinTrig, LOW);

  // pulseIn() 會測量 Echo 腳位維持 HIGH 狀態的時間（微秒）
  long duration = pulseIn(pinEcho, HIGH);

  Serial.println(duration);
  delay(200);
}
```

#### 實作 2：微秒轉換為公分
```cpp
// ============================================
// 實作 2：把時間換算成距離
// 公式：距離(cm) = 時間(微秒) / 58
//（聲速約 340 m/s，來回需除以 2，經單位換算後簡化為除以 58）
// ============================================

int pinTrig = 9;
int pinEcho = 10;

void setup() {
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  Serial.begin(9600);
}

void loop() {
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);

  long duration = pulseIn(pinEcho, HIGH);
  float distanceCm = duration / 58.0;

  Serial.print("距離: ");
  Serial.print(distanceCm);
  Serial.println(" cm");
  delay(200);
}
```

#### 實作 3：安全距離門檻警告
```cpp
// ============================================
// 實作 3：物件小於 10cm 亮起警告燈
// ============================================

int pinTrig = 9;
int pinEcho = 10;
int pinLed = 13;

void setup() {
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  pinMode(pinLed, OUTPUT);
}

void loop() {
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);

  long duration = pulseIn(pinEcho, HIGH);
  float distanceCm = duration / 58.0;

  if (distanceCm < 10) {
    digitalWrite(pinLed, HIGH);
  } else {
    digitalWrite(pinLed, LOW);
  }
}
```

---

### 🕑 第 2 小時：非接觸式互動與報警

**情境故事**：倒車雷達是超音波應用最經典的案例，越靠近障礙物警報聲越急促，接下來我們把這個概念套用到智慧感應垃圾桶（AGV 載具蓋）與 RGB 燈號分級警示。

#### 實作 4：倒車雷達進階版（三級警報頻率）
```cpp
// ============================================
// 實作 4：依距離遠近，設定三個等級的警報音頻率
// ============================================

int pinTrig = 9;
int pinEcho = 10;
int pinBuzzer = 8;

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
}

void loop() {
  float distance = readDistance();

  if (distance < 10) {
    tone(pinBuzzer, 2000);   // 極近：高頻急促
    delay(100);
    noTone(pinBuzzer);
    delay(100);
  } else if (distance < 30) {
    tone(pinBuzzer, 1000);   // 中距離：中頻
    delay(200);
    noTone(pinBuzzer);
    delay(300);
  } else if (distance < 50) {
    tone(pinBuzzer, 500);    // 較遠：低頻緩慢
    delay(100);
    noTone(pinBuzzer);
    delay(700);
  } else {
    noTone(pinBuzzer);       // 安全距離：無聲
  }
}
```
**函式重構提醒**：這裡把讀取距離的重複程式碼封裝成 `readDistance()` 函式，之後只要呼叫這個函式名稱即可取得距離，不用每次都重寫 6 行程式碼。

#### 實作 5：智能感應垃圾桶（自動開蓋）
```cpp
// ============================================
// 實作 5：手靠近小於 15cm，伺服馬達自動開蓋
// ============================================

#include <Servo.h>

int pinTrig = 9;
int pinEcho = 10;
Servo lidServo;

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
  lidServo.attach(6);
  lidServo.write(0);   // 初始：蓋子關閉
}

void loop() {
  float distance = readDistance();

  if (distance < 15) {
    lidServo.write(90);   // 開蓋
  } else {
    lidServo.write(0);    // 關蓋
  }
  delay(100);
}
```

#### 實作 6：距離連動 RGB 燈色
```cpp
// ============================================
// 實作 6：遠(綠)、中(黃)、近(紅) 三段距離顏色指示
// ============================================

int pinTrig = 9;
int pinEcho = 10;
int pinR = 2, pinG = 3, pinB = 4;  // 本週改用一般數位腳位示範三顆獨立 LED 亦可，若用 RGB LED 請改回 PWM 腳位

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
  pinMode(pinR, OUTPUT);
  pinMode(pinG, OUTPUT);
  pinMode(pinB, OUTPUT);
}

void loop() {
  float distance = readDistance();

  digitalWrite(pinR, LOW);
  digitalWrite(pinG, LOW);
  digitalWrite(pinB, LOW);

  if (distance < 15) {
    digitalWrite(pinR, HIGH);        // 近：紅
  } else if (distance < 40) {
    digitalWrite(pinG, HIGH);        // 中：黃（此處先用綠色腳位示範，可依教學需求調整）
  } else {
    digitalWrite(pinB, HIGH);        // 遠：藍（象徵安全）
  }
}
```

---

### 🕒 第 3 小時：物流堆疊高度檢測

**情境故事**：輸送帶上方架設固定的超音波感測器，用來檢測經過的貨箱高度是否超過安全規範，這是自動化品檢分類線常見的應用。

#### 實作 7：測量經過箱子的高度
```cpp
// ============================================
// 實作 7：固定式超音波偵測經過物體高度
// 假設感測器固定裝設在輸送帶正上方 50cm 處
// ============================================

int pinTrig = 9;
int pinEcho = 10;
float sensorHeight = 50.0;   // 感測器安裝高度（公分）

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
  Serial.begin(9600);
}

void loop() {
  float distanceToObject = readDistance();
  float objectHeight = sensorHeight - distanceToObject;  // 感測器高度減去偵測到的距離 = 物體高度

  Serial.print("物體高度: ");
  Serial.print(objectHeight);
  Serial.println(" cm");
  delay(200);
}
```

#### 實作 8：分類機構（超高箱體推離主線）
```cpp
// ============================================
// 實作 8：高度超過規範的箱子，伺服馬達推桿將其推離主線
// ============================================

#include <Servo.h>

int pinTrig = 9;
int pinEcho = 10;
Servo pusherServo;
float sensorHeight = 50.0;
float maxAllowedHeight = 20.0;   // 允許的最大箱體高度

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
  pusherServo.attach(6);
  pusherServo.write(0);
}

void loop() {
  float distanceToObject = readDistance();
  float objectHeight = sensorHeight - distanceToObject;

  if (objectHeight > maxAllowedHeight) {
    pusherServo.write(90);   // 推桿動作，將超高箱體推離
    delay(500);
    pusherServo.write(0);    // 推桿歸位
  }
}
```

#### 實作 9：連續異常累計觸發大警報
```cpp
// ============================================
// 實作 9：連續 3 個不良品通過，觸發系統大警報要求人工介入
// 教學重點：偶發異常可自動處理，但「連續」異常代表系統性問題，需要人工查驗
// ============================================

#include <Servo.h>

int pinTrig = 9;
int pinEcho = 10;
int pinBuzzer = 8;
Servo pusherServo;
float sensorHeight = 50.0;
float maxAllowedHeight = 20.0;

int consecutiveDefects = 0;   // 連續異常計數
bool wasDefect = false;       // 記錄上一輪是否為異常，避免同一箱子被重複計數

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
  pusherServo.attach(6);
  pusherServo.write(0);
  Serial.begin(9600);
}

void loop() {
  float distanceToObject = readDistance();
  float objectHeight = sensorHeight - distanceToObject;
  bool isDefect = objectHeight > maxAllowedHeight;

  if (isDefect && !wasDefect) {
    consecutiveDefects++;
    Serial.print("連續異常次數：");
    Serial.println(consecutiveDefects);

    pusherServo.write(90);
    delay(500);
    pusherServo.write(0);

    if (consecutiveDefects >= 3) {
      tone(pinBuzzer, 2000);   // 觸發大警報
      Serial.println("！！連續異常，請人工介入檢查！！");
    }
  }

  if (!isDefect) {
    consecutiveDefects = 0;    // 只要出現一個正常品，連續計數歸零
    noTone(pinBuzzer);
  }

  wasDefect = isDefect;
}
```

### 🌶️ 第 9 週進階挑戰題
1. **平均值濾波**：超音波感測器有時會讀到雜訊，能否連續讀取 5 次距離值取平均，讓數值更穩定？
2. **自訂大警報解除機制**：結合第 6 週學過的 Reset 按鈕，讓連續異常警報需要人工按鈕確認才能解除。
3. **AGV 三段減速模擬**：距離越近，讓一顆 LED 的閃爍速度越快，模擬 AGV 靠近障礙物時的「視覺化減速提示」。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 8 週：期中專題 — 智慧倉儲環境監控站](./week-08.md) | [下一週：第 10 週：連續動力驅動 ➡](./week-10.md)
