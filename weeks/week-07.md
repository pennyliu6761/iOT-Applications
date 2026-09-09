[⬅ 回目錄](../README.md) | [⬅ 上一週：第 6 週：溫度監控與安全防護](./week-06.md) | [下一週：第 8 週：期中專題 — 智慧倉儲環境監控站 ➡](./week-08.md)

---

## 第 7 週：精準角度控制
**新增元件：微型伺服馬達（Servo Motor）**

### 📌 學習目標
- 學會使用 `<Servo.h>` 函式庫控制伺服馬達角度（0°～180°）。
- 理解伺服馬達與直流馬達的差異：伺服馬達能「精準定位角度」，而非單純轉動。
- 練習將類比感測數值映射為機構動作角度，這是自動化「感測 → 致動」的核心橋樑。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **Micro Servo（伺服馬達）** 到工作區，通常有三條線：**橘/黃色（訊號線）、紅色（電源 +5V）、棕/黑色（GND）**。
2. 訊號線接到 Arduino 的 **Pin 6**（伺服馬達函式庫不限定 PWM 腳位，但慣例上仍建議選用 PWM 腳位）。
3. 電源線接 **5V**，接地線接 **GND**。
4. 沿用第 4 週可變電阻（A0）、第 6 週 TMP36（A2）、第 5 週光敏電阻（A1）、第 5 週蜂鳴器（Pin 8）。

---

### 🕐 第 1 小時：伺服馬達函式庫與基礎控制

**情境故事**：物流分揀站需要一個能自動開關的閘門機構，用來管制棧板進出，這正是伺服馬達最常見的工業應用之一。

#### 實作 1：控制馬達轉到指定角度
```cpp
// ============================================
// 實作 1：伺服馬達基礎控制 - 轉到 0°、90°、180°
// ============================================

#include <Servo.h>   // 引入伺服馬達函式庫

Servo myServo;       // 建立一個伺服馬達物件，命名為 myServo

void setup() {
  myServo.attach(6);  // 將 myServo 物件與 Pin 6 綁定
}

void loop() {
  myServo.write(0);     // 轉到 0 度
  delay(1000);
  myServo.write(90);    // 轉到 90 度（中間位置）
  delay(1000);
  myServo.write(180);   // 轉到 180 度
  delay(1000);
}
```

#### 實作 2：緩慢來回掃描動作（Sweep）
```cpp
// ============================================
// 實作 2：伺服馬達平滑來回掃描
// ============================================

#include <Servo.h>

Servo myServo;

void setup() {
  myServo.attach(6);
}

void loop() {
  // 從 0 度掃到 180 度
  for (int angle = 0; angle <= 180; angle++) {
    myServo.write(angle);
    delay(15);   // 每個角度停留一小段時間，馬達移動才會平滑
  }
  // 從 180 度掃回 0 度
  for (int angle = 180; angle >= 0; angle--) {
    myServo.write(angle);
    delay(15);
  }
}
```

#### 實作 3：按鈕控制閘門升降
```cpp
// ============================================
// 實作 3：物流閘門控制 - 按一下升起，再按一下放下
// ============================================

#include <Servo.h>

Servo gateServo;
int pinButton = 7;
int lastButtonState = LOW;
bool gateOpen = false;

void setup() {
  gateServo.attach(6);
  pinMode(pinButton, INPUT);
  gateServo.write(0);   // 初始狀態：閘門放下（0度）
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);  // 防彈跳
    gateOpen = !gateOpen;

    if (gateOpen) {
      gateServo.write(90);   // 升起
    } else {
      gateServo.write(0);    // 放下
    }
  }
  lastButtonState = currentState;
}
```

---

### 🕑 第 2 小時：類比連動與人機互動

**情境故事**：主管希望能有一個「手動測試模式」，讓維修人員可以直接用旋鈕測試閘門機構在各種角度下的運作是否順暢，這是設備調機階段常見的測試手法。

#### 實作 4：旋鈕即時控制馬達角度
```cpp
// ============================================
// 實作 4：旋鈕轉多少，馬達角度就跟著轉多少
// ============================================

#include <Servo.h>

Servo myServo;
int pinPot = A0;

void setup() {
  myServo.attach(6);
}

void loop() {
  int rawValue = analogRead(pinPot);
  int angle = map(rawValue, 0, 1023, 0, 180);
  myServo.write(angle);
  delay(15);   // 給馬達一點反應時間，避免指令下達太快
}
```

#### 實作 5：溫度儀表板（伺服馬達當指針）
```cpp
// ============================================
// 實作 5：用伺服馬達模擬指針式溫度計
// ============================================

#include <Servo.h>

Servo gaugeServo;
int pinTemp = A2;

void setup() {
  gaugeServo.attach(6);
}

void loop() {
  int rawValue = analogRead(pinTemp);
  float voltage = rawValue * (5.0 / 1023.0);
  float celsius = (voltage - 0.5) * 100.0;

  // 假設儀表範圍設定為 0°C ~ 50°C，對應指針角度 0°~180°
  int angle = map(celsius, 0, 50, 0, 180);
  angle = constrain(angle, 0, 180);  // constrain() 確保角度不會超出 0~180 範圍（避免感測誤差造成馬達報錯）
  gaugeServo.write(angle);
}
```
**新函式提醒**：`constrain(數值, 最小值, 最大值)` 會把超出範圍的數值「夾」在邊界內，這在處理真實世界雜訊資料時非常實用。

#### 實作 6：光線追蹤儀
```cpp
// ============================================
// 實作 6：依光敏電阻數值改變馬達角度（簡易向光性模擬）
// ============================================

#include <Servo.h>

Servo trackerServo;
int pinLdr = A1;

void setup() {
  trackerServo.attach(6);
}

void loop() {
  int lightLevel = analogRead(pinLdr);
  int angle = map(lightLevel, 0, 1023, 0, 180);
  trackerServo.write(angle);
  delay(15);
}
```

---

### 🕒 第 3 小時：智慧閘門系統

**情境故事**：期中專題進入倒數，這一小時整合閘門控制、計數管制與聲光提示，模擬完整的「智慧道閘管制系統」。

#### 實作 7：自動道閘（延遲自動降下）
```cpp
// ============================================
// 實作 7：按下升起，延遲 3 秒後自動降下
// ============================================

#include <Servo.h>

Servo gateServo;
int pinButton = 7;
int lastButtonState = LOW;

void setup() {
  gateServo.attach(6);
  pinMode(pinButton, INPUT);
  gateServo.write(0);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);
    gateServo.write(90);   // 升起
    delay(3000);           // 維持升起狀態 3 秒（車輛通過時間）
    gateServo.write(0);    // 自動放下
  }
  lastButtonState = currentState;
}
```

#### 實作 8：進出場計數管制（滿載拒絕開啟）
```cpp
// ============================================
// 實作 8：場內車輛數量管制 - 滿載時閘門拒絕開啟
// ============================================

#include <Servo.h>

Servo gateServo;
int pinButton = 7;
int lastButtonState = LOW;
int carCount = 0;
int maxCapacity = 5;   // 場內最大容納數量

void setup() {
  gateServo.attach(6);
  pinMode(pinButton, INPUT);
  gateServo.write(0);
  Serial.begin(9600);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);

    if (carCount < maxCapacity) {
      carCount++;
      Serial.print("車輛進場，目前數量：");
      Serial.println(carCount);
      gateServo.write(90);
      delay(2000);
      gateServo.write(0);
    } else {
      Serial.println("場內已滿，閘門拒絕開啟！");
      // 閘門保持不動，不執行升起動作
    }
  }
  lastButtonState = currentState;
}
```

#### 實作 9：加入蜂鳴器提示音（重型機具運作警示）
```cpp
// ============================================
// 實作 9：閘門作動期間發出提示音，模擬重型機具運作警示
// ============================================

#include <Servo.h>

Servo gateServo;
int pinButton = 7;
int pinBuzzer = 8;
int lastButtonState = LOW;

void setup() {
  gateServo.attach(6);
  pinMode(pinButton, INPUT);
  gateServo.write(0);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);

    // 閘門升起前先發出提示音，模擬機具啟動警示
    tone(pinBuzzer, 1000);
    delay(300);
    noTone(pinBuzzer);

    gateServo.write(90);
    delay(3000);

    tone(pinBuzzer, 1000);
    delay(300);
    noTone(pinBuzzer);

    gateServo.write(0);
  }
  lastButtonState = currentState;
}
```

### 🌶️ 第 7 週進階挑戰題
1. **超音波預告（先修先玩）**：如果告訴你可以用「距離」判斷有無車輛接近（下週會學到的超音波感測器），你會怎麼設計「車輛靠近才允許按鈕生效」的邏輯？先讓學生用文字或虛擬碼構思。
2. **場內剩餘車位顯示**：結合第 3 週 RGB LED，剩餘車位充足顯示綠色，車位剩 1 個顯示黃色，已滿顯示紅色。
3. **雙閘門管制系統**：新增第二顆伺服馬達模擬「入口閘」與「出口閘」，各自獨立控制，並共用同一組場內計數變數。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 6 週：溫度監控與安全防護](./week-06.md) | [下一週：第 8 週：期中專題 — 智慧倉儲環境監控站 ➡](./week-08.md)
