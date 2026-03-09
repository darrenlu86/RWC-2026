# TEE.fail: Breaking Trusted Execution Environments via Memory Bus Interposition

## Session

- **Session:** Hardware
- **Conference:** RWC 2026

## Speaker(s)

- **Christina Pöpper**
- **Daniel Gruss**

## Abstract

透過記憶體匯流排攔截（memory bus interposition）技術，對 Intel SGX/TDX 等可信執行環境進行實體攻擊，利用確定性加密（deterministic encryption）的弱點提取 ECDSA 認證金鑰，並展示對區塊鏈、NVIDIA Confidential Compute 等應用的實際影響。

## Summary

### TEE 基礎

**Trusted Execution Environment (TEE)** 是一組硬體功能，旨在由 CPU 本身強制執行資料存取控制與隔離。目標是即使 CPU 以外的一切都被攻破，仍能確保機密性與完整性。

主要 TEE 實作：Intel SGX、Intel TDX、AMD SEV、ARM TrustZone、ARM CCA。

### 記憶體匯流排攔截

研究者直接在伺服器主機板上的 CPU 與 DDR5 RDIMM 之間的線路上焊接攔截裝置：

- DDR5 有 276 個引腳需要連接
- Daniel 負責焊接，Christina 負責供應巧克力和咖啡，並檢查短路
- 設計了 DDR4 和 DDR5 的攔截板（interposer board）加上探針隔離網路
- 成功捕捉到 DRAM 匯流排上的讀取命令與回應資料

### 確定性加密的致命缺陷

Intel Scalable SGX 使用 **AES-XTS 模式**進行記憶體加密。Intel 規格中明確寫道：

> 「此密碼學方案僅能用於緩解對手只能看到密文一次的硬體攻擊」

AES-XTS 原本設計用於**磁碟加密**，不是 RAM 加密。其核心問題：**確定性加密**——相同的明文永遠產生相同的密文（如同「加密企鵝」的經典範例：色彩被擾亂但幾何資訊被保留）。

現場 demo 直接展示了 Intel 系統上的確定性加密：
- 全零明文 → 固定密文 A
- 全一明文 → 固定密文 B
- 再次全零 → 密文 A 再次出現

### 攻擊：提取 ECDSA 認證金鑰

**攻擊目標**：TEE 的認證金鑰（attestation key），用於 ECDSA 簽署 quote（應用程式的雜湊摘要）。

**ECDSA 實作分析**：
1. 產生隨機 nonce k
2. 計算 k × G（純量乘以群生成點），使用標準的 **fixed window method**
3. 預計算表（precomputed table）存放在 RAM 中
4. Nonce k 被切成 5-bit 的區塊（chunks），每個區塊用預計算表查詢

**攻擊流程**：
1. 記憶體攔截器監視預計算表的存取位址
2. 每次存取時，邏輯分析儀捕捉對應的密文
3. 由於預計算表內容已知，其加密結果也已知（確定性加密！）
4. 透過密文查表，反推每個 5-bit slice 的值
5. 組合所有 slice → 還原 nonce k → 還原 ECDSA 私鑰

**現場 demo**：使用約 1,000 美元的邏輯分析儀，在約 **1.5 分鐘**內成功提取 TDX 認證金鑰。

### 攻擊影響

取得認證金鑰後，攻擊者可以**偽裝成合法的 Intel 硬體**，通過任何遠端認證檢查。

**已驗證的攻擊場景**：

1. **區塊鏈 / Web3**：
   - 解密智慧合約網路中的交易
   - 搶先交易（front-running）
   - 偽造分散式儲存網路中的儲存證明

2. **Signal**：使用 SGX 進行密碼恢復，加密備份可能面臨風險

3. **NVIDIA Confidential Compute**：
   - GPU 的認證依賴 CPU TEE 進行代理
   - CPU 與 GPU 的認證報告之間**沒有綁定**（not linked）
   - 攻擊者可用偽造的 TDX 認證 + 從雲端租借的 GPU 認證組合，欺騙客戶以為 AI 模型在安全環境中運行
   - 現場 demo：用一台 Mac + 3 美元雲端 GPU 租金，成功偽造完整的 Confidential Compute 認證

### 廠商回應

- 廠商表示這些攻擊**在 SGX/TDX/SEV 的保護範圍之外**
- 移除完整性與重放保護是**有意識的設計決定**——技術團隊被管理層否決，理由是客戶對**效能**的需求
- 管理層認為實體攻擊太昂貴、太困難，不需要防護
- **無法透過微碼更新修復**——需要硬體重新設計，即使廠商想做也需要多年

### 為何不用隨機化加密

Q&A 中的解釋：若使用 AES-CTR 等隨機化加密，每 64 bytes 需要一個計數器，計數器本身又需要計數器，逐層累加。每次記憶體存取都需要查詢所有這些計數器才能解密，被認為效能代價太高。

### 可能的緩解措施

- 更新認證協議以證明機器的**物理位置**（Proof of Location，講者戲稱為「Proof of not Daniel」）
- 但這是一個根本性的架構問題，短期內沒有完美解法

## 與 TWCA 臺灣網路認證的關聯

本場演講對 TWCA 有直接且重要的影響：

1. **TEE 信任模型的根本性質疑**：TWCA 的 HSM 和伺服器環境可能使用或評估 Intel SGX/TDX 進行金鑰保護或安全運算。本研究揭示，TEE 的記憶體加密使用確定性加密（AES-XTS），物理攻擊者可在 1.5 分鐘內提取認證金鑰。TWCA 在評估任何依賴 TEE 的解決方案時，必須考慮此威脅。

2. **遠端認證的可偽造性**：TWCA 若依賴 TEE 的遠端認證來驗證合作夥伴或客戶的安全環境，本研究證明認證可被完全偽造。這直接影響 TWCA 對「可信運算環境」的信任決策。

3. **NVIDIA Confidential Compute 的認證缺陷**：CPU 與 GPU 認證未綁定的問題，對 TWCA 未來評估 AI + 安全運算的方案有警示作用——不能單純信任認證鏈，需要驗證認證之間的綁定關係。

4. **HSM vs TEE 的安全等級差異**：本研究再次凸顯 HSM（硬體安全模組）與 TEE 在實體攻擊防護上的根本差距。TWCA 的核心金鑰操作應繼續使用經 FIPS 140-2/3 認證的 HSM，而非 TEE。
