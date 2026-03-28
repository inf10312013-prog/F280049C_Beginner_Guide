# TI F280049C 入門手冊（數位電源 / LLC / PFC 取向）

## 1. 這份手冊是寫給誰的
這份手冊是寫給剛接觸 TI C2000、但想往數位電源控制發展的人。

如果你過去比較熟的是一般 MCU 韌體、協定流程、量測自動化，現在想補：
- LLC / PFC 的數位控制觀念
- PWM、ADC、ISR 的關係
- C2000 為什麼常被用在 digital power
- CLA、Trip Zone、dead-band、phase sync 等模組怎麼看

那麼 F280049C 是一顆很適合當作入門主角的 MCU。

這份手冊不追求把 datasheet 或 TRM 全部翻譯完，而是希望用比較像工程學習路線的方式，幫你先抓住主幹。

---

## 2. F280049C 到底是什麼
F280049C 是 TI C2000 家族中的即時控制 MCU。它的定位不是單純的通用微控制器，而是偏向 real-time control，特別適合：
- digital power
- motor control
- inverter / converter
- grid-tied control
- power stage 保護與控制整合

對入門者來說，不要先把它想成「規格很多的 MCU」，而要先把它想成：

**一顆很擅長把 PWM、ADC、控制運算、保護機制串成閉迴路的控制器。**

---

## 3. 你要先建立的核心觀念
學 F280049C 時，最重要的不是先背寄存器，而是先理解這條流程：

**ePWM 定節拍 → ADC 在指定時刻取樣 → ISR / CLA 做控制運算 → 更新 PWM → fault 路徑獨立保護**

只要這條主線有建立起來，你後面看：
- 中斷頻率
- ADC SOC
- CLA 任務分工
- dead-band
- phase shift
- Trip Zone

就不會覺得每個模組都像孤島。

---

## 4. 新手最該先學的模組

### 4.1 ePWM
這是數位電源最核心的周邊之一。

你可以先把 ePWM 理解成：
- 產生開關波形
- 決定 switching 節拍
- 觸發 ADC 取樣
- 建立多組 PWM 的同步與相位差
- 搭配 dead-band 與 Trip Zone 完成時序與保護

如果你未來做 LLC、PFC、PSFB、interleaved converter，幾乎一定會用到。

### 4.2 ADC
ADC 不只是把電壓或電流讀進來而已。

在數位電源裡，更重要的是：
- 什麼時候取樣
- 由誰觸發取樣
- 哪一筆結果拿來做控制
- 取樣雜訊怎麼處理

所以你學 ADC 時，不要只看 resolution，要更在意：
- SOC 是怎麼設的
- 取樣點跟 PWM 的關係
- interrupt 是由哪個 conversion 完成後觸發

### 4.3 ISR / PIE
中斷是控制迴路的節拍入口。

你可以先把 ISR 理解成：
- 取樣完成後進來算控制器的地方
- 也可能是 fault / event 的反應入口

PIE 則是把不同 peripheral interrupt 統整進 CPU 的擴充中斷架構。

剛入門時，你只要先知道：
- 哪個事件會進 ISR
- ISR 裡做什麼
- ISR 結尾要清哪些 flag
- PIEACK 要不要回

先不要一開始就死背整張向量表。

### 4.4 CLA
CLA 是 Control Law Accelerator。

你可以先把它理解成：
**用來分擔 fast control task 的控制加速器。**

它的價值通常不是「取代主 CPU」，而是把一些對 latency / jitter 很敏感的計算工作切出去，讓主 CPU 比較能專心處理：
- state machine
- communication
- background task
- logging
- housekeeping

初學時請記住兩件事：
1. CLA 很重要，但不要直接把它講成一般意義的 dual core
2. 不一定每個專案都要先用 CLA，先把 CPU 版本控制流程跑通更重要

### 4.5 dead-band
dead-band 是互補開關控制的重要模組。

它的用途是：
- 避免上下橋臂同時導通
- 調整互補訊號之間的 dead time
- 幫助半橋 / 全橋 / LLC 類拓樸安排更安全的 switching 時序

學 dead-band 時要記得：
- 太短可能有 shoot-through 風險
- 太長可能影響效率或軟切換條件

### 4.6 phase / sync
如果系統裡有多組 PWM，常常需要處理：
- 同步啟動
- phase shift
- 多相交錯
- bridge leg 之間的對應關係

這時 ePWM 的 sync / phase 機制就很重要。

對初學者來說，先能回答下面問題就夠：
- 誰是 master PWM
- 誰跟著同步
- 相位差是怎麼定義的
- 更新 phase 之後對波形會有什麼影響

### 4.7 Trip Zone
Trip Zone 是保護相關的核心模組。

它不是慢慢進 ISR 再決定怎麼辦，而是能更快地改變 PWM 輸出狀態，用來保護功率級。

你可以先把它理解成：
- 異常時快速關閉或改變 PWM
- fault path 通常要比一般控制路徑更快

對數位電源來說，這個觀念很重要：

**控制是讓系統表現更好，保護是避免系統先壞掉。**

### 4.8 SysConfig
SysConfig 是 TI 提供的圖形化設定工具。

它的價值是幫你：
- 設 pin mux
- 設 peripheral initialization
- 配中斷
- 減少基本設定打錯的機率

新手很適合用它快速 bring-up。

但要記住：
SysConfig 是加速器，不是替代你理解底層架構的魔法。

---

## 5. 新手一開始不要急著鑽的東西
如果你是為了數位電源入門，以下主題先知道名字就好，不要一開始花最多時間：
- Flash API
- linker command file 細節
- ECC 細節
- boot mode 細節
- CLB 深挖
- 很細的 driverlib API 差異

這些之後都可能有用，但不是第一週最該啃的內容。

---

## 6. 建議的學習順序
這是比較適合入門的順序：

### 第 1 階段：先把主幹建起來
1. C2000 / F280049C 整體定位
2. ePWM
3. ADC SOC
4. ADC interrupt
5. PIE / ISR

### 第 2 階段：開始接近 power control
6. dead-band
7. sync / phase
8. Trip Zone
9. 簡單閉迴路範例

### 第 3 階段：往高階控制延伸
10. CLA
11. HRPWM
12. Digital Power SDK
13. TMU、dq transform、PLL

這個順序的好處是：
你會先學會「控制系統怎麼活起來」，再去學「怎麼把它做更快、更穩、更漂亮」。

---

## 7. 你真正要會回答的問題
學 F280049C 時，請不要只追求「看過很多周邊」。

你至少要練到自己能回答下面這些問題：

1. 為什麼 C2000 常被用在 digital power？
2. PWM、ADC、ISR 的關係是什麼？
3. ADC 為什麼常由 PWM 觸發？
4. ISR 裡什麼該做，什麼不該做？
5. CLA 適合做什麼，不適合怎麼亂講？
6. dead-band、phase、Trip Zone 各自在 power stage 裡扮演什麼角色？
7. 控制路徑跟保護路徑，為什麼不能混為一談？

只要你能回答這 7 題，對 F280049C 的理解就已經脫離單純抄範例了。

---

## 8. 建議的最小練習題目
如果你想真的把 F280049C 用起來，而不是只看影片，建議照下面順序做：

### 練習 1：打出一組固定 PWM
目標：
- 知道 ePWM 在哪裡設 period / compare
- 知道波形是怎麼出來的

### 練習 2：讓 ePWM 觸發 ADC
目標：
- 知道 ADC 不是只能用軟體觸發
- 知道取樣節拍可以綁 PWM

### 練習 3：讓 ADC conversion 完成後進 ISR
目標：
- 知道真正控制節拍通常來自中斷
- 知道 ISR 不是萬能，但很關鍵

### 練習 4：在 ISR 裡更新 PWM
目標：
- 把 sample → compute → update 串起來
- 建立最簡單的數位控制感覺

### 練習 5：加入 dead-band 與 Trip
目標：
- 開始建立「控制 + 保護」一體化觀念

### 練習 6：再考慮導入 CLA 或更完整的 power SDK
目標：
- 從能跑進化到跑得更像產品級架構

---

## 9. 新手常見誤解

### 誤解 1：我應該先背完整 datasheet
錯。
你應該先抓控制流程，再回頭查 datasheet / TRM。

### 誤解 2：CLA 就是 dual core
不精確。
CLA 很重要，但比較好的說法是控制加速器或協同處理單元。

### 誤解 3：看到 ADC 就只是在想解析度
不夠。
對數位電源來說，取樣時機 often 比單純 bit 數更關鍵。

### 誤解 4：ISR 越快越好
不一定。
重點是整個控制節拍、計算負擔、量測雜訊與 update timing 是否合理。

### 誤解 5：Trip Zone 只是附屬功能
錯。
對 power stage 來說，保護路徑常常跟控制路徑同等重要，甚至更優先。

---

## 10. 你可以怎麼規劃自己的學習筆記
建議每學一個模組，都固定用同一種格式做筆記：

### 模組名稱
例如：ePWM

### 這個模組在做什麼
例如：產生 PWM、建立節拍、觸發 ADC

### 在數位電源裡為什麼重要
例如：它決定 switching 與 sampling 的節奏

### 常見會跟哪些模組配合
例如：ADC、Trip Zone、dead-band、phase sync

### 面試時可怎麼白話講
例如：ePWM 不只是打波形，也是很多控制節拍與保護控制的基礎

這樣你的筆記會比「抄一堆寄存器名稱」更有長期價值。

---

## 11. 建議使用的開發資源
入門時建議至少熟悉下面這些工具或資源：
- CCS
- C2000Ware
- SysConfig
- F28004x 相關範例
- Digital Power SDK

你不一定要一次全懂，但請至少知道它們分別扮演什麼角色：
- CCS：開發與除錯
- C2000Ware：基本範例、driver、裝置支援
- SysConfig：圖形化設定
- Digital Power SDK：更完整的電源應用參考

---

## 12. 給入門者的最後建議
如果你是從一般韌體、測試、協定或自動化背景轉進來，學 F280049C 時最容易焦慮的點，是看到太多新名詞。

其實你真正要做的不是一次全部懂，而是先把這條線練順：

**PWM 怎麼出來 → ADC 什麼時候取樣 → ISR 怎麼進來 → 控制器在哪裡算 → 新的 PWM 怎麼更新 → fault 發生時怎麼保護**

只要這條線順了，你之後再學：
- LLC
- PFC
- dq transform
- CLA
- HRPWM
- Digital Power SDK

都會快很多。

這顆 MCU 的難，不在於資料太多；真正的難，是你要把它跟 power stage 的控制邏輯連起來。

一旦這件事做到了，F280049C 就不再只是「一顆很多周邊的 MCU」，而是一個很有力量的數位電源控制平台。
