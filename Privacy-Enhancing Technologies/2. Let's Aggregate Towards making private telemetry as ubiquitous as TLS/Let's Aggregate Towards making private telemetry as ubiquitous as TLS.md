# Let's Aggregate? Towards making private telemetry as ubiquitous as TLS

## Session

- **Session:** Privacy-Enhancing Technologies
- **Chair:** Thomas Ristenpart
- **Conference:** RWC 2026

## Speaker(s)

- **Ryan Lehmkuhl** — MIT

## Authors

- Ryan Lehmkuhl (MIT)
- Henry Corrigan-Gibbs (Stanford)
- Emma Dauterman (UT Austin)
- David J. Wu

## Abstract

本研究探討如何推動隱私聚合（private aggregation）技術的大規模採用，使其如同 TLS 一樣普及。研究者提出「Heavy-Light」部署模型，讓非營利組織運行的第二伺服器（light server）的工作量與通訊量與客戶端數量無關，從而大幅降低部署成本。在一千萬模擬使用者的實驗中，light server 的成本比現有方案（Prio、Whisper）低 12 萬倍。

## Summary

### 隱私聚合的動機

開發者為了改善產品，需要了解應用程式的使用狀況。例如 WhatsApp 可能想知道「今天有多少比例的使用者封鎖了聯絡人」。目前的做法是直接從每個使用者收集原始資料，但這讓服務商不僅得到群體統計，更學到每個個別使用者的敏感行為與人口特徵，甚至可以結合社交圖譜進行關聯分析。

**隱私聚合（Private Aggregation）**以原始使用者資料為輸入，僅輸出聚合結果（如加總、直方圖、模型更新）。它具備兩個核心屬性：(1) **隱私性**：即使伺服器是惡意的，也只能學到聚合結果，無法得知個別使用者資料；(2) **正確性**：能偵測並排除提交無效報告的惡意客戶端。

### 從 TLS 採用歷程汲取教訓

講者以 2014-2019 年 TLS 採用率從 30% 躍升至 80% 的成功故事為類比。TLS 普及的兩大驅動力是：(1) **Let's Encrypt** 於 2015 年提供免費憑證，消除付費門檻；(2) **瀏覽器施加壓力**，對未使用 TLS 的網站顯示警告，搜尋引擎降低排名。

目前的隱私聚合系統（如 Prio、Whisper）需要**兩台伺服器**，且兩台必須由不同實體運營才能保障隱私。第二台伺服器的運營需要付費外包，這正如同 TLS 普及前憑證需要付費購買的障礙。

### Heavy-Light 部署模型

解決方案是仿效 Let's Encrypt，由非營利組織免費運行第二台伺服器。但問題在於聚合運算中第二伺服器的工作量與應用伺服器相當，對非營利組織而言無法擴展。

因此提出 **Heavy-Light Aggregation** 模型：
- **Heavy server**（由應用提供商運營）：與所有客戶端通訊
- **Light server**（由非營利組織運營）：**不與客戶端通訊**，工作量與通訊量為**常數**，與客戶端數量無關
- 允許一次性設定階段（one-time setup），讓 light server 與所有客戶端通訊一次

### 技術核心：Key Homomorphic PRF

構建基於三個步驟：

1. **基礎方案**：客戶端將輸入編碼到群組中，進行 secret sharing 給兩台伺服器
2. **以 PRF 降低通訊量**：客戶端使用偽隨機值替代真隨機值進行 secret sharing，只需發送一個 PRF key 給第二伺服器
3. **以 Key Homomorphic PRF 降低計算量**：利用 key homomorphism 特性（F(k1, x) + F(k2, x) = F(k1+k2, x)），第二伺服器在收集所有客戶端的 PRF key 並求和後，每輪聚合只需進行**一次 PRF 運算**

具體實作使用橢圓曲線群上的簡單構造：H(x)^k，其中 H 是到 DDH 困難群的雜湊函數。

### 實驗評估

使用 Rust 實作，基於 Ristretto 255 群組，在 AWS 上對一千萬模擬客戶端進行端到端評估（每個客戶端提交 32 個 1-bit 輸入）：

- **Light server** 成本比 Prio/Whisper 低最高 **2,300 萬倍**
- **Heavy server** 成本增加最多 **60 倍**（瓶頸來自驗證客戶端零知識證明）
- 在 happy path 下，light server 的具體成本為**一次指數運算**，通訊**一個群元素**

### 採用的非技術障礙

即使技術問題完美解決，隱私聚合仍面臨缺乏經濟誘因的問題：相較於標準遙測，隱私聚合成本更高、分析彈性更低、且無法利用資料進行廣告或訓練。需要類似 TLS 時代瀏覽器施壓的外部激勵，最可能的推動者是 iOS/Android 平台在應用商店中對隱私聚合的支持與要求。

### 開放問題

1. 客戶端退出時能否維持 light server 常數開銷，同時不大幅增加 heavy server 負擔？
2. 能否在 heavy-light 設定下加速客戶端證明驗證（讓 light server 協助 heavy server）？
3. 資料匯出的 UI 等價物是什麼？（類似 TLS 的鎖頭圖示）
4. 後量子安全性：作者認為可以用格基 key homomorphic PRF 替換，但尚未正式證明

## 與 TWCA 臺灣網路認證的關聯

本場演講與 TWCA 的業務沒有直接關聯，但有值得關注的間接啟示：

1. **Let's Encrypt 模式的借鑑**：本研究直接以 Let's Encrypt 的成功作為隱私聚合推廣的藍本。TWCA 作為臺灣的憑證機構，可以觀察這種「免費公共服務推動技術普及」的模式是否可能延伸到隱私保護領域。

2. **隱私遙測與法規趨勢**：隨著全球隱私法規趨嚴（如 GDPR、臺灣個資法），企業對隱私聚合技術的需求將增長。TWCA 作為信任服務提供者，未來可能有機會扮演 heavy-light 模型中的 light server 角色，提供隱私聚合的信任基礎設施。

3. **雙伺服器信任模型**：隱私聚合要求兩台伺服器由不同實體運營，這與 TWCA 現有的第三方信任角色定位一致，可作為未來服務擴展的參考方向。
