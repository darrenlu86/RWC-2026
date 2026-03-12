# Zero Knowledge (About) Encryption: On the Security of Cloud-based Password Managers

## Session

- **Session:** Login Security
- **Chair:** Anja Lehmann
- **Conference:** RWC 2026

## Speaker(s)

- **Matteo Scarlata**
- **Giovanni Torrisi**

## Authors

- Matilda Backendal (ETH Zurich)
- Kenneth Paterson
- Matteo Scarlata
- Giovanni Torrisi

## Abstract

針對四大雲端密碼管理器（Bitwarden、LastPass、Dashlane、1Password）進行系統性安全分析，在惡意伺服器威脅模型下發現 27 個攻擊，揭示這些產品宣稱的「零知識加密」遠未達到端對端加密的安全保證，攻擊涵蓋金鑰恢復、部分保險庫同步、共享機制及向後相容性等功能面。

## Summary

### 雲端密碼管理器的運作模型

四大雲端密碼管理器——Bitwarden、LastPass、Dashlane、1Password——共擁有超過 6,000 萬用戶及超過 28% 的市場份額。其基本架構為：使用者僅記憶一組**主密碼（master password）**，透過金鑰衍生（key derivation）與金鑰封裝（key wrapping）產生**保險庫金鑰（vault key）**，用以加密保險庫（vault）中的登入項目，加密後的保險庫上傳至雲端伺服器。

### 「零知識加密」的定義模糊

四家廠商均宣稱提供「零知識加密（zero knowledge encryption）」，但此詞在密碼學中並無嚴格定義。各廠商的說法暗示：即使伺服器被攻破，資料仍安全——這本質上等同於**端對端加密（end-to-end encryption）**的保證。研究團隊設定的威脅模型為：**誠實客戶端 vs 完全惡意伺服器**，測試這些產品是否真正達到 E2EE 的安全性。

### 攻擊類別一：金鑰恢復（Key Recovery）

密碼管理器提供金鑰恢復功能，讓組織管理員（CIS admin）透過公鑰加密機制取回使用者的保險庫金鑰。

**LastPass 與 Bitwarden 的漏洞**：
- 「是否啟用恢復」的設定**未經認證（unauthenticated）**且儲存在伺服器端
- 受信任方的公鑰**未經認證**
- 惡意伺服器可對任何使用者單方面啟用恢復功能，並替換為攻擊者控制的公鑰
- 客戶端會無條件信任此資訊，將保險庫金鑰分享給攻擊者

即使使用者**從未啟用恢復功能**，仍受此攻擊影響。

### 攻擊類別二：部分保險庫同步（Partial Vault Sync）

Dashlane 和 1Password 採用**項目級加密（item-level encryption）**，而 LastPass 和 Bitwarden 採用**欄位級加密（field-level encryption）**。

**欄位級加密的金鑰分離不足**：LastPass 和 Bitwarden 對同一項目內的不同欄位（使用者名稱、密碼、URL）使用相同的加密金鑰，惡意伺服器可**交換欄位**（如將密碼欄位放到 URL 欄位的位置）。

**結合 UI 功能的實際攻擊**：客戶端會向伺服器請求網站圖示（favicon），URL 為「我的密碼內容是什麼？」——惡意伺服器交換欄位後，客戶端會以密碼內容作為 URL 向伺服器請求圖示，密碼因此洩露。

### 攻擊類別三：共享機制（Sharing）

所有四家廠商在共享功能中都存在相同問題：Alice 要分享憑證給 Bob 時，必須向伺服器查詢 Bob 的公鑰。惡意伺服器可替換為自己的公鑰，攔截所有共享資料。

**1Password 的特殊嚴重性**：此攻擊不僅影響共享資料，還影響使用者自己的**非共享保險庫**。

### 攻擊類別四：向後相容性——Heisenberg 攻擊（Bitwarden）

Bitwarden 為支援舊版客戶端，保留了使用 **AES-CBC（無認證加密）** 的舊登入路徑。現代版本使用 AES-CBC-HMAC（有認證加密）。

**攻擊流程**：
1. 客戶端為**無狀態（stateless）**，使用哪條路徑由伺服器告知
2. 此選擇欄位**未經認證**——惡意伺服器只需翻轉一個位元即可降級任何客戶端
3. 降級後密文不再經過認證，伺服器可注入任意內容
4. 研究者發現另一條舊路徑會將**密文-明文對（ciphertext-plaintext pair）**發送給伺服器
5. 利用此對，伺服器可注入已知的保險庫金鑰

**Heisenberg 不確定性**：由於填充修正（padding correction）會破壞部分明文，攻擊者必須在兩種模式間選擇：
- **模式一**：注入短版保險庫金鑰（無 MAC 金鑰）→ 保留機密性但喪失完整性，可篡改所有未來的憑證
- **模式二**：注入長版保險庫金鑰（完整加密金鑰已知，MAC 金鑰被破壞）→ 喪失機密性但保留完整性，可讀取所有未來的憑證

兩者無法同時達成，如同量子力學的測不準原理。

### 修復建議

- **金鑰恢復與共享**：採用 E2E 加密應用的成熟方案，如 HSM 金鑰託管、金鑰透明性（Key Transparency）
- **部分保險庫同步**：正確實施**金鑰分離（key separation）**
- **向後相容性**：效法 TLS 和 WireGuard，直接**放棄對舊版的支援**

### 揭露與修復現況

所有漏洞均已向廠商揭露。截至演講時，27 個攻擊中僅有 **9 個已修復**，許多修復需要重大架構調整。

## 與 TWCA 臺灣網路認證的關聯

本場演講對 TWCA 有直接的技術與業務啟示：

1. **「零知識」行銷用語的風險**：密碼管理器廠商使用「零知識加密」一詞來暗示端對端加密的安全保證，但實際未達此標準。TWCA 在對外溝通安全產品時，應確保技術宣稱有嚴格的密碼學定義支撐，避免類似的信任落差。

2. **公鑰分發的根本問題**：共享功能中公鑰未經認證的問題，正是 PKI 要解決的核心問題。TWCA 的憑證基礎設施能為密碼管理器等應用提供可信的公鑰分發機制，這是潛在的業務擴展方向。

3. **金鑰恢復機制的安全設計**：TWCA 的企業客戶可能使用這些密碼管理器進行憑證與金鑰管理。本研究揭示的金鑰恢復漏洞意味著，即使用戶未啟用恢復功能，惡意伺服器仍可竊取保險庫金鑰——TWCA 應在安全建議中納入此風險。

4. **向後相容性的教訓**：Bitwarden 的 Heisenberg 攻擊源於保留舊版不安全協定路徑。TWCA 在管理憑證生命週期和協定升級時，應以 TLS 和 WireGuard 為榜樣，在適當時機果斷淘汰舊版協定，而非無限期維持向後相容。
