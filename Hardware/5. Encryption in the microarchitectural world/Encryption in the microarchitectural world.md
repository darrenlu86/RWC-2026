# Encryption in the Microarchitectural World

## Session

- **Session:** Hardware
- **Conference:** RWC 2026

## Speaker(s)

- **Ping-Lun Wang** — Carnegie Mellon University (CMU)

## Authors

Ping-Lun Wang, Fraser Brown, Riccardo Paccagnella, Eyal Ronen, Riad S. Wahby, Yuval Yarom

**發表：**
- Flexo: *Bending Microarchitectural Weird Machines towards Practicality* [USENIX Security 2024]
- Transient Architectural Execution: *From Weird Gates to Weird Programs* [IEEE S&P 2026]

## Abstract

本演講介紹微架構怪機器（microarchitectural weird machines, μWMs）的最新進展。μWMs 利用 CPU 微架構元件（快取、分支預測器）的硬體副作用進行隱藏計算，使程式碼分析、除錯器與模擬器失效。研究者提出兩項關鍵改進：Flexo 透過差分編碼大幅提升 weird gate 的速度與準確度（543 倍加速），並開發首個 μWM 編譯器與二進位打包器；Transient Architectural Execution 則將微架構狀態轉換為架構狀態，在暫態執行視窗中使用完整 ISA 指令（包括 AES-NI），從 weird gate 進化到 weird program，使 μWM 成為實際的程式混淆與攻擊威脅。

## Summary

### 背景：架構 vs. 微架構

CPU 由兩層組成：
- **ISA（指令集架構）**：架構層，程式可見的指令與暫存器
- **μArch（微架構）**：快取階層、分支預測器等硬體實作細節，對程式不可見

**微架構怪機器（μWMs）**是利用微架構元件進行隱藏計算的「CPU 中的隱藏電腦」。其特性：
- 防止程式碼分析（分析者看不到微架構層的計算）
- 隱藏計算狀態（狀態存在快取等微架構結構中）
- 擊敗除錯器與模擬器（它們不模擬微架構行為）

### Weird Gate 的運作原理

Weird gate 在**暫態執行**（transient execution，如分支預測錯誤）中進行計算：

1. **觸發暫態執行**：利用分支預測錯誤（branch misprediction）等機制，讓 CPU 投機性地執行不應執行的指令
2. **以輸入延遲計算**：輸入值以**快取狀態**表示（cached = 1, not cached = 0）。快取命中延遲短、快取未命中延遲長，這種延遲差異驅動計算
3. **存取輸出 weird register**：將計算結果寫入輸出的快取狀態（記憶體載入操作會改變快取狀態）

**AND gate 範例**：
```c
if (condition()) {
    tmp += out[in1[0] + in2[0]];
}
```
- in1 和 in2 同時被載入（無依賴），加法在載入完成後計算
- 只有當兩個輸入都快取命中（延遲短）時，out 的載入才能在暫態視窗結束前完成

先前工作也實作了 OR gate（序列載入兩個輸入）和 NOT gate（利用延遲函數）。多個 weird gate 可組合成 weird circuit——例如 XOR 需要 5 個 weird gate。

### Flexo：更快更準確的 Weird Gate

#### 差分編碼（Differential Encoding）

Flexo 的核心創新是用**兩個 weird register**（plus 和 minus）表示一個二進位值：
- plus = 1, minus = 0 → 編碼值 **1**
- plus = 0, minus = 1 → 編碼值 **0**
- plus = minus → **無效狀態**（用於錯誤偵測）

雖然使用了雙倍的 weird register，但帶來重大優勢：

**消除 NOT 運算**：在差分編碼下，NOT 操作只需交換 plus 和 minus register，不需要額外的 weird gate。任何邏輯函數都可以只用 AND 和 OR 運算表示。

**更強大的 gate**：直接從真值表建構 weird gate，支援**最多 4 個輸入**的任意邏輯函數。例如一個 Flexo weird gate 即可完成 4-bit XOR 或 2-bit 加法器，而先前工作只能建構基礎的 AND/OR/NOT gate。

#### 動態投票錯誤校正

先前工作使用 3-out-of-5 多數決投票（每個 gate 固定執行 5 次）。Flexo 改用**動態投票**：只在偵測到錯誤（無效狀態）時才重新執行，正常情況下只需一次。實驗證明準確度更高、開銷更低。

#### 首個 μWM 編譯器

Flexo 提出了首個 μWM 編譯器——開發者只需撰寫高階程式碼（如 SHA-1 round 的 21 行 C 程式），編譯器自動處理底層細節（原本需要 128 行手寫微架構操作碼）並進行最佳化。

#### μWM 二進位打包器

結合 UPX 打包器與 Flexo 加密，建構首個 μWM 二進位打包器。使用 Simon 密碼解包 132KB 約需 1 分鐘（AES 約 5 分鐘）。雖然速度較慢，但其複雜性可擊敗多種現有防毒工具。

整體性能提升：相比先前工作達到 **543 倍加速**。

### Transient Architectural Execution：從 Weird Gate 到 Weird Program

#### Weird Gate 的侷限

即使有 Flexo 的改進，weird gate 仍有根本限制：
- 分支和記憶體操作的開銷呈指數增長
- 一個 weird register 只能存 1 bit
- 一個 weird gate 最多計算 4 bit

#### 核心思想：μArch 狀態 + Arch 計算

Transient Architectural Execution 的突破在於：在一個暫態執行視窗中完成三步驟：
1. 將微架構狀態（快取狀態）轉換為架構狀態（暫存器值）
2. 使用完整 ISA 指令進行架構計算
3. 將結果轉回微架構狀態

#### cache2arch：快取狀態到架構狀態的轉換

```c
int cache2arch(int *input) {
    if (input[0]) return 0;
    else return 1;
}
```

- `input[0]` 在架構層被設為 0，所以正常應返回 1
- 但在暫態執行中：
  - input **已快取** → CPU 快速判斷值為 0 → 執行 else → 返回 **1**
  - input **未快取** → 載入延遲長 → CPU 詢問分支預測器 → 預測 if 為 true → 返回 **0**

實際實作使用 **Branch Target Buffer (BTB)** 而非分支預測器，可免去訓練過程，無額外開銷。

在一個暫態執行視窗中，可轉換超過 **128 bit** 的微架構狀態到架構狀態。

#### Weird Function 的強大能力

有了 cache2arch，weird function 獲得全新的計算能力：
- 使用**強大的 ISA 指令**（如 AES-NI）——一個 weird function 即可加密一個 AES block
- 執行**分支指令**
- **動態存取** weird register
- 存取主記憶體（唯讀）
- 可組合成 **weird program** 進行大規模計算

#### 跨平台相容性

μWM 利用的是微架構的基本機制（快取、分支預測），而非特定微架構的細節。實驗證明可在 **5–6 個世代**的 Intel/AMD 處理器上運作，對微架構的小幅變更不敏感。

### Q&A 重點

- **暫態視窗中可轉換多少 bit？**取決於微架構，實作中可在一個暫態視窗轉換超過 128 bit
- **是否需要針對每代處理器重新開發？**利用的是基本機制，可跨 5–6 代 Intel/AMD 處理器運作
- **分支預測器如何確保預測方向正確？**投影片中的分支預測器程式碼僅為說明用途，實際使用 BTB，無需訓練過程
- **差分編碼的其他用途？**有聽眾提到差分方法過去也被用於旁通道防護

## 與 TWCA 臺灣網路認證的關聯

1. **惡意軟體混淆的新威脅**：μWM 使程式碼分析、除錯器、模擬器全部失效，且 Flexo 已提供編譯器與打包器。TWCA 在進行惡意程式分析或安全審計時，需注意 μWM 混淆的可能性——傳統的靜態與動態分析工具可能無法偵測隱藏在微架構層的加解密運算。

2. **密碼運算的隱藏執行**：μWM 可在微架構層執行完整的 AES 加密（利用 AES-NI），且計算過程對架構層完全不可見。若攻擊者在 TWCA 的伺服器上部署利用 μWM 的惡意程式，傳統的執行軌跡監控將無法偵測密碼運算的發生。

3. **暫態執行攻擊的防禦**：μWM 的基礎是暫態執行（Spectre 類攻擊的變體）。TWCA 的伺服器環境應確認已部署 Spectre 緩解措施（微碼更新、核心修補），但需注意 μWM 利用的是基本的分支預測與快取機制，完全消除可能影響效能。

4. **與同場演講的對照**：Hardware #1（TEE.fail）利用微架構的記憶體加密弱點提取金鑰，本場則展示微架構可被用來隱藏加密運算。兩者共同指出：微架構是安全攻防的新戰場，TWCA 在評估系統安全時不能只關注架構層的行為。
