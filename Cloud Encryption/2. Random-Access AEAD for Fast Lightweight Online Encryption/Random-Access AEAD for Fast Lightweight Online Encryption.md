# Random-Access AEAD for Fast Lightweight Online Encryption

## Session

- **Session:** Cloud Encryption
- **Chair:** Joseph Bonneau
- **Conference:** RWC 2026

## Speaker(s)

- **Andres Fabrega** — Cornell University
- **Gregory Rubin**

## Authors

- Andres Fabrega (Cornell University)
- Greg Rubin (Snowflake)
- Julia Len (UNC Chapel Hill)
- Thomas Ristenpart (Cornell Tech)

## Abstract

提出 FLOE（Fast Lightweight Online Encryption）演算法及隨機存取認證加密（Random-Access AEAD）的新安全定義 RA$-RoR。FLOE 支援串流處理、隨機存取加解密、FIPS 140-3 合規，並具備承諾性（committing），解決了現有線上加密方案在大型資料集場景中的安全性與合規性不足問題。

## Summary

### 問題背景

Snowflake 的內部團隊被要求使用 AES-GCM 保護大型資料集時發現根本無法運作。AES-GCM 等認證加密需要檢查整個密文才能回傳任何明文——10 GB 的密文需要約 20 GB 記憶體才能解密，無法擴展。

### 設計需求

1. **串流處理（Streaming/Online）**：以常數記憶體開銷順序處理任意大小的明文/密文
2. **隨機存取（Random Access）**：支援寫入一次、多次讀取模式下，對資料子集進行加解密，支援多執行緒處理
3. **FIPS 140-3 合規**：使用 AES-GCM 搭配隨機 nonce、使用經核准的金鑰衍生函式（如 HKDF-Expand）
4. **安全易用**：可交給一般開發者使用、易於實作、提供安全且有用的錯誤訊息
5. **承諾性（Committing）**：單一密文只能解密為單一明文

### 現有方案的問題

**樸素分段方案**：將大檔案切成多段分別加密。問題：無法防禦跨段攻擊——攻擊者可交換段落順序（「I like apples but not cake」變成「I like cake but not apples」）、刪除段落、或截斷密文。

**STREAM 與 Tink Streaming**：將段落資訊（位置）編碼進每段的初始化向量（IV），解決了跨段攻擊問題。但 IV 不再是隨機的，因此**不符合 FIPS 合規**。且隨機存取加密的安全性是未解問題。

### FLOE 演算法設計

FLOE 的完整建構非常簡潔：

- **段落資訊**移入每段 AES-GCM 加密的附加認證資料（AAD），而非 IV，保持 IV 為隨機值（FIPS 合規）
- **段落標頭**：包含隨機 IV 及前一段的元資料，簡化解密流程
- **密文標頭**：包含所有解密所需參數、每訊息 IV、以及 header tag（提供承諾性與參數完整性）
- **金鑰排程**：使用 HKDF-Expand 每 2^20 段重新衍生金鑰，支援 EB 等級的資料量在單一頂層金鑰下運作，解決 GCM 在 96-bit 隨機 IV 下每 2^32 訊息需輪換金鑰的限制

### 新安全定義：RA$-RoR

**既有定義的問題**：
1. Hoang 等人的 NOAE（2015）限制為順序加解密
2. Hoang & Chen 的 NOA2 加入隨機存取解密，但安全定義複雜且不直觀
3. 均無隨機存取加密的安全保證
4. 完全缺乏承諾性定義

**新原語——隨機存取認證加密方案（RA-AE）**：
- 位置作為顯式參數
- 單一固定狀態跨所有段落加密
- 支援全域附加認證資料（global AAD）和密文標頭

**RA$-RoR 安全定義**：基於標準 AEAD 的 Real-or-Random 安全性，逐段適用。對手擁有任意順序加解密段落的 oracle 存取權。此定義嚴格強於 NOA2（RA$-RoR ⊂ NOA2，但反向不成立）。STREAM 和 Tink 在 RA$-RoR 下仍然安全。

**承諾性定義**：引入隨機存取上下文承諾（RA Context Commitment），區分 position 和 positionless 兩種變體。STREAM/Tink 的承諾性取決於底層 AEAD；FLOE 的 positionless 承諾性歸約到 KDF 的碰撞抗性，與 AEAD 解耦。

### 效能表現

**Java 基準測試（1 GB 檔案）**：
- 加密：FLOE ≈ AES-GCM（時間與記憶體幾乎相同）
- 解密：AES-GCM 需 90 秒 + 16 GB 記憶體；FLOE 與加密效能一致

**隨機存取測試（10 GB 檔案，雲端儲存）**：
- 順序單執行緒：每方向約 3.5 分鐘（CPU 瓶頸）
- 16 MB 區塊多執行緒隨機存取：每方向約 1.5 分鐘（57% 速度提升），達到約 1 Gbps 網路飽和

### 開放資源

- 規格發布於 GitHub 及 C2SP
- 四種程式語言的參考實作
- 第三方已在 400 行程式碼內移植至新語言

## 與 TWCA 臺灣網路認證的關聯

1. **大型憑證與日誌資料的加密儲存**：TWCA 運營 PKI 基礎設施，涉及大量憑證資料、稽核日誌、CRL 等大型檔案的加密儲存與備份。FLOE 的串流處理與隨機存取能力，可大幅改善這些場景的加解密效能與記憶體使用。

2. **FIPS 合規性**：TWCA 的安全操作需符合國際標準。FLOE 原生設計為 FIPS 140-3 合規（使用 AES-GCM + 隨機 nonce + HKDF），可直接應用於需要合規性的場景，無需擔心現有線上加密方案的 IV 非隨機問題。

3. **承諾性對 PKI 的重要性**：FLOE 的承諾性保證（單一密文只能解密為單一明文）對於 TWCA 處理憑證簽發請求、時間戳記等需要明確對應關係的場景尤為重要，可防止密文歧義攻擊。

4. **雲端備份與災難復原**：TWCA 的異地備份與災難復原方案可採用 FLOE 實現高效、安全且支援部分恢復的加密備份，無需解密整個備份檔案即可存取特定資料區段。
