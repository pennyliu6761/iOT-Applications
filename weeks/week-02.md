[⬅ 回目錄](../README.md) | [⬅ 上一週：第 1 週：Arduino 初探與數位輸出](./week-01.md) | [下一週：第 3 週：類比輸出與混色視覺 ➡](./week-03.md)

---

## 第 2 週：數位輸入與狀態決策
**新增元件：按鈕開關（Pushbutton）、10kΩ 下拉電阻**

### 📌 學習目標
- 理解「輸入」與「輸出」的差異：`digitalRead()` vs `digitalWrite()`。
- 認識序列埠監控視窗（Serial Monitor），學會用 `Serial.println()` 除錯。
- 學習 `if-else` 條件判斷，以及「狀態變數」的記憶概念（Toggle）。
- 認識「按鍵彈跳（Bouncing）」現象與簡易解法。
- 學習 `&&`（AND）邏輯運算子。

### 🔌 Tinkercad 電路搭建指引
1. 沿用第 1 週的麵包板，拖曳一顆 **Pushbutton** 元件到麵包板上（跨越中間凹槽放置，四隻腳分成兩組，每組兩腳互通）。
2. 按鈕的一側接一顆 **10kΩ 電阻** 到麵包板的**負極軌（GND）**，這就是「下拉電阻」，作用是在按鈕沒被按下時，讓輸入腳位有一個穩定的 LOW（0V），避免腳位「飄浮」產生亂數雜訊。
3. 同一側再接一條線到 Arduino 的 **Pin 7**。
4. 按鈕另一側接到麵包板的**正極軌（5V）**，正極軌接回 Arduino 的 **5V**。
5. 邏輯：沒按下時 Pin 7 讀到 LOW（被電阻拉到 GND）；按下時電流改走按鈕接通到 5V，Pin 7 讀到 HIGH。
6. 實作 7-9 會再加入第二顆按鈕，比照上述方式接在 **Pin 8**。

---

### 🕐 第 1 小時：聽見硬體的聲音（按鈕與 Serial Monitor）

**情境故事**：品管站需要一顆「確認鈕」，作業員檢查完一件產品沒問題就按一下。在寫出完整邏輯前，你得先確認：這顆按鈕的訊號，電腦真的收得到嗎？

#### 實作 1：按鈕接線，學習 digitalRead()
```cpp
// ============================================
// 實作 1：讀取按鈕狀態（先不做任何輸出動作）
// ============================================

int pinButton = 7;

void setup() {
  pinMode(pinButton, INPUT);  // 設定為輸入模式：這隻腳位負責「聽」外界訊號
}

void loop() {
  int state = digitalRead(pinButton);  // 讀取目前是 HIGH 還是 LOW
  // 這裡先不做事，下一步驟才會顯示出來
}
```

#### 實作 2：開啟 Serial Monitor，印出按鈕狀態
```cpp
// ============================================
// 實作 2：把按鈕狀態「印」出來，肉眼確認電路是否正常
// ============================================

int pinButton = 7;

void setup() {
  pinMode(pinButton, INPUT);
  Serial.begin(9600);  // 開啟序列埠通訊，速率 9600 bps
}

void loop() {
  int state = digitalRead(pinButton);
  Serial.println(state);  // 印出 0（沒按）或 1（按下）
  delay(100);  // 稍微延遲，避免洗版太快看不清楚
}
```
**操作提醒**：點開 Tinkercad 右側的「Serial Monitor」視窗，開始模擬後，用滑鼠點擊按鈕，觀察數值從 0 變成 1。

#### 實作 3：加入 if-else，按住亮、放開滅
```cpp
// ============================================
// 實作 3：品管確認燈 - 按住時亮起代表「確認中」
// ============================================

int pinButton = 7;
int pinLed = 13;

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int state = digitalRead(pinButton);

  if (state == HIGH) {
    digitalWrite(pinLed, HIGH);  // 按下 → 燈亮，代表正在確認
  } else {
    digitalWrite(pinLed, LOW);   // 放開 → 燈滅
  }
}
```

---

### 🕑 第 2 小時：狀態反轉與變數記憶（設備開關邏輯）

**情境故事**：主管覺得「按著才亮」不實用，希望改成像電燈開關一樣——「按一下開機、再按一下關機」。這就是設備電源鍵常見的 Toggle（切換）邏輯。

#### 實作 4：Toggle 功能（需要記憶「目前狀態」）
```cpp
// ============================================
// 實作 4：設備電源鍵 - 按一下開、再按一下關
// 關鍵觀念：需要一個變數「記住」目前是開還是關
// ============================================

int pinButton = 7;
int pinLed = 13;

bool machineOn = false;       // 記錄機台目前開/關狀態
int lastButtonState = LOW;    // 記錄「上一次」讀到的按鈕狀態，用來偵測「剛按下的瞬間」

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int currentState = digitalRead(pinButton);

  // 偵測「由 LOW 變成 HIGH」的瞬間（而不是持續按著）
  if (currentState == HIGH && lastButtonState == LOW) {
    machineOn = !machineOn;  // 反轉狀態：true 變 false，false 變 true
  }

  digitalWrite(pinLed, machineOn ? HIGH : LOW);

  lastButtonState = currentState;  // 更新「上一次狀態」，供下一輪迴圈比對
}
```

#### 實作 5：發現按鍵彈跳問題，加入簡易防彈跳
```cpp
// ============================================
// 實作 5：解決「按一次卻切換好幾次」的彈跳問題
// 原因：機械按鈕接觸瞬間會有極短暫的高頻電氣抖動
// ============================================

int pinButton = 7;
int pinLed = 13;

bool machineOn = false;
int lastButtonState = LOW;

void setup() {
  pinMode(pinButton, INPUT);
  pinMode(pinLed, OUTPUT);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);  // 簡易防彈跳：偵測到變化後，暫停 50 毫秒讓訊號穩定下來
    machineOn = !machineOn;
  }

  digitalWrite(pinLed, machineOn ? HIGH : LOW);
  lastButtonState = currentState;
}
```

#### 實作 6：產線計數器（累加良品數量）
```cpp
// ============================================
// 實作 6：每按一次，良品數量 +1，並印出目前累計數
// ============================================

int pinButton = 7;
int goodCount = 0;        // 良品累計數量
int lastButtonState = LOW;

void setup() {
  pinMode(pinButton, INPUT);
  Serial.begin(9600);
}

void loop() {
  int currentState = digitalRead(pinButton);

  if (currentState == HIGH && lastButtonState == LOW) {
    delay(50);  // 防彈跳
    goodCount = goodCount + 1;   // 也可寫成 goodCount++;
    Serial.print("目前良品數量：");
    Serial.println(goodCount);
  }

  lastButtonState = currentState;
}
```

---

### 🕒 第 3 小時：多重條件與防呆設計（安全防護邏輯）

**情境故事**：沖床機台是工廠中高危險性設備，必須確保作業員雙手都離開危險區域才能啟動——這正是產業安全中著名的「雙手啟動裝置」設計理念。

#### 實作 7：雙按鈕控制單一 LED（A 開、B 關）
```cpp
// ============================================
// 實作 7：兩顆按鈕分別控制開/關（各司其職）
// ============================================

int pinButtonA = 7;  // 啟動鈕
int pinButtonB = 8;  // 停止鈕
int pinLed = 13;

void setup() {
  pinMode(pinButtonA, INPUT);
  pinMode(pinButtonB, INPUT);
  pinMode(pinLed, OUTPUT);
}

void loop() {
  if (digitalRead(pinButtonA) == HIGH) {
    digitalWrite(pinLed, HIGH);   // A 鈕按下 → 啟動
  }
  if (digitalRead(pinButtonB) == HIGH) {
    digitalWrite(pinLed, LOW);    // B 鈕按下 → 停止
  }
}
```

#### 實作 8：AND 邏輯 — 雙手安全啟動機制
```cpp
// ============================================
// 實作 8：沖床防斷手安全機制
// 必須「兩顆按鈕同時按下」機台才會啟動
// ============================================

int pinButtonA = 7;   // 左手按鈕
int pinButtonB = 8;   // 右手按鈕
int pinMachine = 13;  // 機台動作指示（模擬沖壓動作）

void setup() {
  pinMode(pinButtonA, INPUT);
  pinMode(pinButtonB, INPUT);
  pinMode(pinMachine, OUTPUT);
}

void loop() {
  bool leftHand  = digitalRead(pinButtonA) == HIGH;
  bool rightHand = digitalRead(pinButtonB) == HIGH;

  // && 代表「同時成立」，任一手放開就會立刻停止，確保安全
  if (leftHand && rightHand) {
    digitalWrite(pinMachine, HIGH);
  } else {
    digitalWrite(pinMachine, LOW);
  }
}
```
**產業連結討論**：這就是真實沖床設備「Two-Hand Control」的簡化模擬，能有效避免作業員單手還在模具內時機台就啟動。

#### 實作 9：搶答器邏輯（誰先按下就鎖定誰）
```cpp
// ============================================
// 實作 9：搶答器 - 展現「狀態鎖定」的邏輯設計
// 情境：品管異常通報搶修，先通報者取得處理權，避免混亂
// ============================================

int pinButtonA = 7;
int pinButtonB = 8;
int pinLedA = 12;   // A 隊指示燈
int pinLedB = 13;   // B 隊指示燈

int winner = 0;  // 0 = 尚未有人搶到, 1 = A 隊, 2 = B 隊

void setup() {
  pinMode(pinButtonA, INPUT);
  pinMode(pinButtonB, INPUT);
  pinMode(pinLedA, OUTPUT);
  pinMode(pinLedB, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  // 只有在「尚未有人搶到」的狀態下，才允許判定新的贏家
  if (winner == 0) {
    if (digitalRead(pinButtonA) == HIGH) {
      winner = 1;
      Serial.println("A 隊搶到通報權！");
    } else if (digitalRead(pinButtonB) == HIGH) {
      winner = 2;
      Serial.println("B 隊搶到通報權！");
    }
  }

  digitalWrite(pinLedA, winner == 1 ? HIGH : LOW);
  digitalWrite(pinLedB, winner == 2 ? HIGH : LOW);

  // 注意：這裡刻意不寫「重置」邏輯，留給學生思考挑戰題
}
```

### 🌶️ 第 2 週進階挑戰題
1. **搶答器加上重置鈕**：新增第三顆按鈕作為裁判重置鍵，按下後 `winner` 歸零，兩燈熄滅。
2. **長按 vs 短按辨識**：能否分辨「按一下」跟「按住 2 秒以上」，做出不同反應？（提示：需要用 `millis()` 記錄按下的起始時間，這個技巧會在第 14 週深入介紹，這裡可先讓學生用 `delay()` 搭配計數器土法煉鋼嘗試）
3. **計數器加上上限**：良品計數器數量達到 10 時，自動觸發 LED 閃爍提醒「達到一箱數量，請更換棧板」。

---


---

[⬅ 回目錄](../README.md) | [⬅ 上一週：第 1 週：Arduino 初探與數位輸出](./week-01.md) | [下一週：第 3 週：類比輸出與混色視覺 ➡](./week-03.md)
