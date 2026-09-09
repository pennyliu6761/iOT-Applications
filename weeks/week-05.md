[⬅ 回目錄](../README.md) | [⬅ 上一週：第 4 週：類比輸入與旋鈕控制](./week-04.md) | [下一週：第 6 週：溫度監控與安全防護 ➡](./week-06.md)

---

# 第二階段：感測與動力輸出（第 5～8 週）

**階段目標**：引入環境感測（聲音、光、溫度）與機械動作（伺服馬達），建立「輸入 → 處理 → 輸出」的自動化系統思維，這是工業自動化最核心的觀念骨架。

---


## 第 5 週：聽覺反饋與環境光感知
**新增元件：蜂鳴器（Buzzer）、光敏電阻（LDR）**

### 📌 學習目標
- 學會使用 `tone()` / `noTone()` 控制蜂鳴器發出特定頻率聲音。
- 理解「分壓電路」原理，學會用光敏電阻讀取環境光線。
- 綜合聲音與光線兩種感測/輸出，訓練多元件協同設計。

### 🔌 Tinkercad 電路搭建指引
1. 拖曳一顆 **Buzzer（蜂鳴器）** 到麵包板，正極（通常標示 `+` 或較長腳）接到 Arduino **Pin 8**，負極接麵包板負極軌回 GND。
2. 拖曳一顆 **Photoresistor（光敏電阻, LDR）** 到麵包板，一腳接 **5V**，另一腳同時連接：
   - 一顆 **10kΩ 電阻**接到 GND（這與光敏電阻構成「分壓電路」）
   - 一條線接到 Arduino 的 **A1**（類比輸入腳）
3. 分壓電路原理：光線越亮，LDR 電阻越小，A1 讀到的電壓越接近 5V（數值越大）；光線越暗，LDR 電阻越大，A1 讀到的電壓越接近 0V（數值越小）。
4. 沿用第 1 週 LED（Pin 13）做為照明燈。

---

### 🕐 第 1 小時：蜂鳴器與音頻控制

**情境故事**：主管希望產線設備能發出「提示音」，讓作業員即使沒看螢幕也能靠聲音判斷機台狀態，這是工廠中常見的聽覺化警示系統。

#### 實作 1：認識 tone() 與 noTone()
```cpp
// ============================================
// 實作 1：讓蜂鳴器發出單一頻率聲音
// ============================================

int pinBuzzer = 8;

void setup() {
}

void loop() {
  tone(pinBuzzer, 1000);  // 發出 1000Hz 的聲音（數字越大音調越高）
  delay(500);
  noTone(pinBuzzer);      // 停止發聲
  delay(500);
}
```

#### 實作 2：用陣列寫簡單的提示音效
```cpp
// ============================================
// 實作 2：用音符陣列寫出簡單的「下課鐘聲」旋律
// ============================================

int pinBuzzer = 8;

// 一組簡單的音階頻率（Hz），數字越大音調越高
int melody[] = {523, 659, 784, 1047};  // Do Mi Sol high-Do
int noteDurations[] = {300, 300, 300, 500};

void setup() {
}

void loop() {
  for (int i = 0; i < 4; i++) {
    tone(pinBuzzer, melody[i], noteDurations[i]);
    delay(noteDurations[i] + 50);  // 每個音符之間留一點空隙，聲音才不會黏在一起
  }
  noTone(pinBuzzer);
  delay(2000);  // 播放完畢後暫停 2 秒再重播
}
```

#### 實作 3：倒車雷達式間歇嗶聲
```cpp
// ============================================
// 實作 3：倒車雷達音效 - 間歇性嗶聲提示
// ============================================

int pinBuzzer = 8;

void setup() {
}

void loop() {
  tone(pinBuzzer, 2000);
  delay(100);
  noTone(pinBuzzer);
  delay(400);  // 嗶... 嗶... 嗶... 的節奏
}
```

---

### 🕑 第 2 小時：光敏電阻（LDR）與分壓電路

**情境故事**：倉儲內某些通道沒有窗戶，主管希望走道燈能在光線不足時自動亮起，節省人工開關燈的時間與電費。

#### 實作 4：讀取光敏電阻類比值
```cpp
// ============================================
// 實作 4：讀取環境光線強度（0~1023）
// ============================================

int pinLdr = A1;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int lightLevel = analogRead(pinLdr);
  Serial.println(lightLevel);
  delay(200);
}
```
**教學提醒**：在 Tinkercad 中可以直接點擊光敏電阻元件，用滑桿調整「環境亮度」進行模擬測試，非常適合觀察數值變化。

#### 實作 5：自動夜間照明
```cpp
// ============================================
// 實作 5：光線不足自動點亮走道燈
// ============================================

int pinLdr = A1;
int pinLed = 13;

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int lightLevel = analogRead(pinLdr);

  if (lightLevel < 300) {       // 數值低代表光線暗（門檻依模擬環境微調）
    digitalWrite(pinLed, HIGH); // 自動開燈
  } else {
    digitalWrite(pinLed, LOW);  // 天亮自動關燈
  }
}
```

#### 實作 6：智慧調光（環境越暗、補光越亮）
```cpp
// ============================================
// 實作 6：反向 map 應用 - 環境越暗，LED 補光越強
// ============================================

int pinLdr = A1;
int pinLed = 9;  // PWM 腳位

void setup() {
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int lightLevel = analogRead(pinLdr);
  // 注意這裡刻意把目標範圍寫成「反向」：光線值越小 → 補光越大
  int brightness = map(lightLevel, 0, 1023, 255, 0);
  analogWrite(pinLed, brightness);
}
```

---

### 🕒 第 3 小時：聲光整合應用

**情境故事**：期中專題即將登場，這一小時讓學生練習「感測器觸發聲音警報」的整合能力，這正是防盜／異常偵測系統的雛形。

#### 實作 7：光控特雷門琴
```cpp
// ============================================
// 實作 7：手在光敏電阻上方晃動，蜂鳴器音調隨之改變
// 用途：訓練學生「感測數值直接映射到另一個裝置參數」的通用手法
// ============================================

int pinLdr = A1;
int pinBuzzer = 8;

void setup() {
}

void loop() {
  int lightLevel = analogRead(pinLdr);
  int frequency = map(lightLevel, 0, 1023, 100, 2000);  // 光線轉換為音頻
  tone(pinBuzzer, frequency);
}
```

#### 實作 8：雷射防盜網模擬
```cpp
// ============================================
// 實作 8：光線被遮蔽瞬間觸發警報
// 情境：模擬倉儲入侵偵測系統（實體世界常用雷射+光敏電阻做這件事）
// ============================================

int pinLdr = A1;
int pinBuzzer = 8;
int threshold = 200;  // 光線門檻，低於此值視為「被遮蔽」

void setup() {
}

void loop() {
  int lightLevel = analogRead(pinLdr);

  if (lightLevel < threshold) {
    tone(pinBuzzer, 1500);   // 觸發警報聲
  } else {
    noTone(pinBuzzer);
  }
}
```

#### 實作 9：產線計數器 2.0（用遮光代替按鈕）
```cpp
// ============================================
// 實作 9：光遮斷式產品計數器
// 情境：物體通過輸送帶瞬間遮住光線，觸發計數 +1
// 技術上與第 2 週按鈕計數器相同架構，只是把輸入源換成感測器
// ============================================

int pinLdr = A1;
int threshold = 200;
int productCount = 0;
bool wasBlocked = false;   // 記錄「上一輪」是否處於被遮蔽狀態

void setup() {
  Serial.begin(9600);
}

void loop() {
  int lightLevel = analogRead(pinLdr);
  bool isBlocked = lightLevel < threshold;

  // 偵測「由未遮蔽變成遮蔽」的瞬間，才計數一次（避免物體停留時重複計數）
  if (isBlocked && !wasBlocked) {
    productCount++;
    Serial.print("累計通過數量：");
    Serial.println(productCount);
  }

  wasBlocked = isBlocked;
}
```

### 🌶️ 第 5 週進階挑戰題
1. **音量與音調雙重警報**：異常時讓蜂鳴器交替播放兩種頻率（如 800Hz 與 1200Hz），比單一音調更具急迫感。
2. **雙感測聯動**：結合第 4 週的可變電阻，用旋鈕設定「觸發警報所需的光線門檻」，不必修改程式碼即可調整靈敏度。
3. **計數效率統計**：在 `productCount` 達到 10 的倍數時，額外播放一段慶祝音效，模擬「整箱包裝完成提示音」。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 4 週：類比輸入與旋鈕控制](./week-04.md) | [下一週：第 6 週：溫度監控與安全防護 ➡](./week-06.md)
