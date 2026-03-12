# Counter Galois Onion (CGO): Fast Non-Malleable Onion Encryption for Tor

## Session

- **Session:** Privacy-Enhancing Technologies
- **Chair:** Thomas Ristenpart
- **Conference:** RWC 2026

## Speaker(s)

- (Not specified)

## Authors

- Jean Paul Degabriele (Technology Innovation Institute)
- Alessandro Melloni (Simula UiB)
- Jean-Pierre Münch (TU Darmstadt)
- Martijn Stam

## Abstract

本研究提出 Counter Galois Onion (CGO)，一種針對 Tor 匿名網路設計的新型洋蔥加密方案，旨在解決現行 Tor 使用 AES-CTR 加密所帶來的可延展性（malleability）問題，特別是標記攻擊（tagging attack）對使用者匿名性的威脅。CGO 基於 GCM 元件構建，提供非可延展性（non-malleability）保護，同時保持與 Tor 現有架構的相容性，並具備優異的效能表現。（本摘要基於 PDF 簡報內容）

## Summary

> 注意：本場演講無逐字稿，以下摘要基於 PDF 簡報內容。

### Tor 洋蔥加密的基本架構

Tor 網路中，資料透過**迴路（circuit）**傳輸，由一個**洋蔥代理（Onion Proxy, OP）**和一系列**洋蔥路由器（Onion Router, OR）**組成。OP 與每個 OR 共享一把對稱金鑰（K1, K2, K3），每個 OR 負責「剝除」一層加密，使進入的密文與離開的密文去關聯化（decorrelation），這是 Tor 匿名性的核心機制。

迴路是對稱通道的類比，支援**漏管（Leaky Pipes）**特性：OP 可以對迴路中的任何 OR 發送訊息，反之亦然。每個 OR 透過檢查 Rx 欄位與 Digest 來判斷自己是否為目標接收者，若非則將 cell 繼續轉發。

### Relay Cell 格式與處理

Tor 的 Relay Cell 為 514 bytes（v4+），結構包含：
- **CircID**（4 bytes）：迴路識別碼
- **CMD**（1 byte）：Cell 類型，RELAY (3) 或 RELAY_EARLY (9)
- **Cell Payload**（509 bytes）：包含 rCMD、Rx（認知欄位，固定為 0x0000）、SID、Digest（截斷的 SHA-1 running hash，4 bytes）、Len、Data

加密過程使用 **AES-CTR** 模式，OP 依序以 K3、K1 等金鑰加密 Cell Payload，產生加密後的 Cell。

### 現行方案的致命缺陷：標記攻擊

AES-CTR 的**可延展性**是核心問題。攻擊者可以在 OR1 翻轉密文中的特定位元，這些修改會**穿透多層加密傳播**，到達 OR3 時檢查解密是否成功，藉此判斷兩段密文是否屬於同一迴路。這就是所謂的**標記攻擊（tagging attack）**，能夠**去匿名化**使用者。

### 歷史演進與改進嘗試

- **2012 年 Tor Proposal 202**（Nick Matthewson）：明確提出洋蔥加密需要更新，要求：(1) 防禦標記攻擊、(2) 非擴展性（non-expanding）以免洩漏迴路大小與相對位置、(3) 更強的端到端完整性保護、(4) 不使用 MAC-then-Encrypt。

- **2016 年 Prop 261**：提議使用**寬輸入可調整密碼（Wide-Input Tweakable Cipher, WITC）**作為每層加密，WITC 天然具備非擴展性與非可延展性，可透過附加冗餘位元實現完整性。

- **[ADL17] GCM-RUP**：基於 GCM 元件的 AEAD 方案，在**釋放未驗證明文（release of unverified plaintext）**的情境下仍保持安全，這對 Tor 的逐層解密架構至關重要。

### CGO 的設計理念

CGO（Counter Galois Onion）結合了上述歷史方案的優點，使用 GCM 的核心元件（AES-CTR + GHASH）來構建非可延展的洋蔥加密層。關鍵設計考量包括：

1. **非可延展性**：任何對密文的篡改都會被偵測到，從根本上防止標記攻擊
2. **非擴展性**：密文大小不會因加密層數增加而膨脹，避免洩漏迴路拓撲資訊
3. **與 Tor 架構相容**：保留 Leaky Pipes 特性，支援現有的 cell 格式
4. **高效能**：利用現代 CPU 對 AES-NI 和 CLMUL 指令集的硬體加速，達到接近 AES-CTR 的效能
5. **混合部署相容性**：可在過渡期間與現有方案並存使用

### 安全性與效能

CGO 的安全性建立在 GCM 元件的已知安全特性之上，並針對洋蔥加密的特殊需求（多層解密、中繼轉發）進行了形式化分析。在效能方面，由於直接利用 AES-CTR 和 GHASH 的硬體加速實作，CGO 相較於基於 WITC 的方案有顯著的效能優勢，同時也可能應用於 MixNet 等類似的匿名通訊系統。

## 與 TWCA 臺灣網路認證的關聯

本場演講與 TWCA 的業務沒有直接關聯。CGO 是針對 Tor 匿名網路的洋蔥加密改進方案，屬於匿名通訊基礎設施的密碼學研究。TWCA 的核心業務為 PKI/SSL 憑證、電子簽章與身分驗證，與匿名通訊的目標方向不同。不過，本研究展示的幾個間接啟示值得關注：(1) AES-CTR 的可延展性問題提醒 TWCA 在評估加密方案時需注意模式選擇對完整性的影響；(2) GCM 元件在非標準場景（如多層加密）中的創新應用，展示了密碼學元件的靈活運用方式；(3) 向後相容的漸進式遷移策略，對 TWCA 未來進行密碼學升級（如後量子遷移）具有參考價值。
