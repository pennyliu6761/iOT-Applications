[⬅ 回目錄](../README.md) | [下一週：第 2 週：數位輸入與狀態決策 ➡](./week-02.md)

---

# 第一階段：數位基礎與邏輯建立（第 1～4 週）

**階段目標**：克服「看到程式碼就恐懼」的心理障礙，熟悉 `pinMode`、`digitalWrite`、`digitalRead` 與 `if-else` 邏輯，建立「一個變數只做一件事」的程式閱讀習慣。

---


## 第 1 週：Arduino 初探與數位輸出
**新增元件：LED、電阻（220Ω）、麵包板**

### 📌 學習目標
- 認識 Tinkercad Circuits 介面（元件庫、程式碼編輯區、模擬按鈕）。
- 理解「腳位（Pin）」是程式與硬體溝通的窗口。
- 學會用 `int` 變數取代腳位數字，養成良好命名習慣。
- 第一次接觸 `for` 迴圈，體會「重複邏輯」如何簡化程式碼。

### 🔌 Tinkercad 電路搭建指引
> 本週電路會逐步升級，請依照下列順序操作。

**Step A（實作 1-2 用）：最簡電路，不使用麵包板**
1. 在 Tinkercad 元件庫拖曳一個 **Arduino Uno R3** 到工作區。
2. 拖曳一顆 **LED** 到旁邊。
3. 用導線將 LED 的**長腳（正極, Anode）**接到 Arduino 的 **Pin 13**。
4. 用導線將 LED 的**短腳（負極, Cathode）**接到 Arduino 的 **GND**。
5. 點選右上角「Code」開啟程式編輯視窗，切換為「文字模式（Text）」。

> ⚠️ 教學提醒：此步驟省略電阻是為了讓學生「先看見結果」，第 3 小時會補上正確的防護電阻觀念，並解釋長期真實硬體上必須加電阻，模擬環境才允許省略。

**Step B（實作 3 用）：加入麵包板與電阻**
1. 拖曳一塊 **Breadboard（麵包板）** 到工作區。
2. 將 LED 兩腳插入麵包板同一直排的相鄰兩孔（跨越中間凹槽）。
3. LED 長腳（正極）那一側，插入一顆 **220Ω 電阻**，電阻另一端接到 Arduino **Pin 13**。
4. LED 短腳（負極）那一排，用導線接到麵包板的 **藍色負極軌（-）**。
5. 麵包板負極軌（-）用導線接回 Arduino 的 **GND**。

**Step C（實作 4-6 用）：擴充為 2 顆 LED**
- 比照 Step B，在麵包板上再插入第 2 顆 LED + 220Ω 電阻，正極分別接 **Pin 12** 與 **Pin 13**，負極同樣接負極軌。

**Step D（實作 7-9 用）：擴充為 5 顆 LED**
- 依序在麵包板上排列 5 顆 LED + 5 顆電阻，正極依序接 **Pin 2、3、4、5、6**，負極全部共用負極軌接回 GND。

---

### 🕐 第 1 小時：點亮第一盞燈（認識軟硬體環境）

**情境故事**：你剛加入一間自動化設備公司擔任儲備幹部，主管交給你的第一個任務很簡單——讓產線上的「運轉指示燈」亮起來。這顆燈未來會告訴現場作業員：「機台正在運作中」。

#### 實作 1：不用麵包板，直接點亮 LED（內建 Blink 程式）
```cpp
// ============================================
// 實作 1：產線運轉指示燈 - 最基本點燈程式
// 對應硬體：LED 直接接在 Pin 13 與 GND
// ============================================

void setup() {
  // 將 Pin 13 設定為「輸出模式」
  // 白話文：告訴 Arduino「我要用這隻腳去控制東西，不是拿來讀取訊號」
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);  // 機台啟動 → 指示燈亮起（輸出 5V 高電位）
  delay(1000);             // 維持亮燈狀態 1000 毫秒（1 秒）
  digitalWrite(13, LOW);   // 機台暫停 → 指示燈熄滅（輸出 0V）
  delay(1000);             // 維持熄滅狀態 1 秒
  // loop() 會不斷重複執行，所以燈會一直閃爍下去
}
```
**帶討論問題**：`setup()` 和 `loop()` 差在哪？為什麼設定腳位模式只需要做一次，但點燈、關燈要放在會重複的地方？

#### 實作 2：修改 delay() 參數，觀察閃爍頻率變化
```cpp
// ============================================
// 實作 2：調整運轉指示燈的閃爍速度
// 情境：主管說「閃太慢了，工人看不出機台是否當機，加快速度！」
// ============================================

void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(200);   // 從 1000 改為 200 毫秒 → 閃爍變快
  digitalWrite(13, LOW);
  delay(200);
}
```
**🌶️ 隨堂小挑戰**：能不能做出「亮的時間短、暗的時間長」的閃爍效果（像心跳訊號燈）？提示：兩個 `delay()` 數值不必相同。

#### 實作 3：移至麵包板，加入電阻防燒毀
```cpp
// ============================================
// 實作 3：程式碼完全不變，只有硬體升級
// 重點：理解「軟體邏輯」與「硬體保護」是兩件事
// ============================================

void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(500);
  digitalWrite(13, LOW);
  delay(500);
}
```
**教師講解重點（麵包板原理）**：麵包板中間有一條凹槽分隔左右兩區，每一直排的 5 個孔在內部是互通的（同一條金屬夾片），而最外側的紅色（+）、藍色（-）軌則是整排橫向互通，專門用來拉電源線，讓多個元件共用同一個正極/負極，不用每個元件都各自接一條線回 Arduino。

---

### 🕑 第 2 小時：自訂變數與多燈控制（流水線指示燈）

**情境故事**：產線擴編了，現在有「進料區」與「出貨區」兩個指示燈，兩邊的號誌需要交替運作，模擬平交道的警示概念——避免兩區同時動作發生碰撞。

#### 實作 4：宣告 int 變數取代腳位數字
```cpp
// ============================================
// 實作 4：用有意義的名字取代數字腳位
// 情境：進料區 LED（Pin 12）、出貨區 LED（Pin 13）
// ============================================

// 用 int（整數）變數儲存腳位編號，往後程式碼可讀性大幅提升
int pinInfeed  = 12;   // 進料區指示燈
int pinOutfeed = 13;   // 出貨區指示燈

void setup() {
  pinMode(pinInfeed, OUTPUT);
  pinMode(pinOutfeed, OUTPUT);
}

void loop() {
  digitalWrite(pinInfeed, HIGH);   // 進料區亮
  digitalWrite(pinOutfeed, HIGH);  // 出貨區也亮（本階段先同步測試）
  delay(500);
  digitalWrite(pinInfeed, LOW);
  digitalWrite(pinOutfeed, LOW);
  delay(500);
}
```

#### 實作 5：雙燈交替閃爍（平交道號誌）
```cpp
// ============================================
// 實作 5：兩區交替運作，避免同時動作（碰撞風險）
// ============================================

int pinInfeed  = 12;
int pinOutfeed = 13;

void setup() {
  pinMode(pinInfeed, OUTPUT);
  pinMode(pinOutfeed, OUTPUT);
}

void loop() {
  digitalWrite(pinInfeed, HIGH);   // 進料區作業中
  digitalWrite(pinOutfeed, LOW);   // 出貨區暫停
  delay(500);

  digitalWrite(pinInfeed, LOW);    // 進料區暫停
  digitalWrite(pinOutfeed, HIGH);  // 出貨區作業中
  delay(500);
}
```

#### 實作 6：3 顆 LED 紅綠燈時相模擬
```cpp
// ============================================
// 實作 6：模擬十字路口紅綠燈（3 種狀態循序切換）
// 情境：廠區出入口的車輛通行號誌
// ============================================

int pinRed    = 2;
int pinYellow = 3;
int pinGreen  = 4;

void setup() {
  pinMode(pinRed, OUTPUT);
  pinMode(pinYellow, OUTPUT);
  pinMode(pinGreen, OUTPUT);
}

void loop() {
  digitalWrite(pinGreen, HIGH);   // 綠燈：可通行
  delay(2000);
  digitalWrite(pinGreen, LOW);

  digitalWrite(pinYellow, HIGH);  // 黃燈：準備停止
  delay(500);
  digitalWrite(pinYellow, LOW);

  digitalWrite(pinRed, HIGH);     // 紅燈：禁止通行
  delay(2000);
  digitalWrite(pinRed, LOW);
}
```

---

### 🕒 第 3 小時：程式結構優化（陣列與迴圈初探）

**情境故事**：出貨區擴充成 5 道閘門的跑馬燈式指示看板，如果每道燈都要手寫 `digitalWrite`，程式碼會變得又臭又長——這正是體會「迴圈」價值的最佳時機。

#### 實作 7：5 顆 LED 逐一點亮（先體驗冗長寫法）
```cpp
// ============================================
// 實作 7：5 道閘門指示燈，尚未使用迴圈（刻意讓學生體會冗長）
// ============================================

void setup() {
  pinMode(2, OUTPUT);
  pinMode(3, OUTPUT);
  pinMode(4, OUTPUT);
  pinMode(5, OUTPUT);
  pinMode(6, OUTPUT);
}

void loop() {
  digitalWrite(2, HIGH); delay(200); digitalWrite(2, LOW);
  digitalWrite(3, HIGH); delay(200); digitalWrite(3, LOW);
  digitalWrite(4, HIGH); delay(200); digitalWrite(4, LOW);
  digitalWrite(5, HIGH); delay(200); digitalWrite(5, LOW);
  digitalWrite(6, HIGH); delay(200); digitalWrite(6, LOW);
}
```
**教師引導提問**：如果現在要擴充到 20 道閘門，這樣寫程式碼要花多久？有沒有更聰明的寫法？

#### 實作 8：導入 for 迴圈與陣列（跑馬燈）
```cpp
// ============================================
// 實作 8：用陣列 + for 迴圈簡化程式碼
// 觀念：陣列 = 一排有編號的置物櫃，可以用迴圈依序打開
// ============================================

int ledPins[] = {2, 3, 4, 5, 6};  // 宣告陣列，存放 5 個腳位編號
int numLeds = 5;                  // 陣列長度（燈的數量）

void setup() {
  // 用 for 迴圈自動設定所有腳位模式，不用寫 5 次 pinMode
  for (int i = 0; i < numLeds; i++) {
    pinMode(ledPins[i], OUTPUT);
  }
}

void loop() {
  // 依序點亮 → 熄滅，模擬產線跑馬燈效果
  for (int i = 0; i < numLeds; i++) {
    digitalWrite(ledPins[i], HIGH);
    delay(150);
    digitalWrite(ledPins[i], LOW);
  }
}
```

#### 實作 9：來回掃描跑馬燈（霹靂車燈 / 產線狀態掃描）
```cpp
// ============================================
// 實作 9：來回掃描效果，模擬巡檢掃描動畫
// ============================================

int ledPins[] = {2, 3, 4, 5, 6};
int numLeds = 5;

void setup() {
  for (int i = 0; i < numLeds; i++) {
    pinMode(ledPins[i], OUTPUT);
  }
}

void loop() {
  // 由左到右
  for (int i = 0; i < numLeds; i++) {
    digitalWrite(ledPins[i], HIGH);
    delay(120);
    digitalWrite(ledPins[i], LOW);
  }
  // 由右到左（i-- 遞減，注意起始值要避開重複點亮最後一顆）
  for (int i = numLeds - 2; i >= 0; i--) {
    digitalWrite(ledPins[i], HIGH);
    delay(120);
    digitalWrite(ledPins[i], LOW);
  }
}
```

### 🌶️ 第 1 週進階挑戰題
1. **雙向對稱跑馬燈**：讓燈光從中間往兩側同時擴散再收回，像呼吸一樣。
2. **速度漸變**：讓跑馬燈從快到慢、再從慢到快，提示：可以用一個變數控制 `delay()` 的數值，並在迴圈中遞增/遞減它。
3. **善用函式（進階）**：把「點亮再熄滅」寫成一個自訂函式 `blinkOnce(int pin, int duration)`，讓主程式呼叫更簡潔。

---


---

[⬅ 回目錄](../README.md) | [下一週：第 2 週：數位輸入與狀態決策 ➡](./week-02.md)
