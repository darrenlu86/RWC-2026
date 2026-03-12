# (Dis)patches from the Web PKI: Fina, Static CT, MTC, and PLANTS

## Session

- **Session:** PKI
- **Chair:** Paul Grubbs
- **Conference:** RWC 2026

## Speaker(s)

- **Luke Valenta** — Cloudflare Inc.

## Authors

- Luke Valenta (Cloudflare Inc.)
- David Benjamin (Google)
- Matthew McPherrin (Let's Encrypt)
- Filippo Valsorda

## Abstract

回顧 Web PKI 近年演進，從 SSL/TLS 到 Certificate Transparency（CT）再到 Static CT，並深入介紹 Merkle Tree Certificates（MTC）作為後量子時代輕量級憑證方案的核心設計。報告了 Chrome 與 Cloudflare 之間的大規模 MTC 實驗結果，展示 landmark MTC 比傳統簽章鏈快 9%，並介紹 IETF PLANTS 工作小組的標準化進展。

## Summary

### Web PKI 的演進歷程

講者以「網際網路是一系列的 patches」為比喻，回顧 Web PKI 的關鍵里程碑：

- **1995 年 SSL v2**：首次為電子商務提供加密與認證，由 CA 簽發憑證綁定域名與公鑰。但很快就有超過 100 個受信任 CA 進入客戶端根憑證庫，是否應該盲目信任所有 CA 成為問題。2011 年 DigiNotar 事件驗證了此擔憂。
- **2013 年 Certificate Transparency（CT）**：將模型從「盲目信任」改為「信任但驗證」。所有公開 CA 簽發的憑證必須存在於 CT log 中，可供公開稽核。CT 於 RWC 2024 獲 Levchin 獎。2025 年仍有 CA 錯誤簽發 1.1.1.1（Cloudflare 加密 DNS）的憑證，僅透過 CT 才被發現。但營運 CT log 昂貴且困難，一度僅剩五個獨立 CT log 營運商。
- **2024 年 Static CT**：由 Filippo Valsorda 於 RWC 2024 提出，將 CT log 資料改為靜態可快取資源，大幅降低營運成本並已獲生態系統廣泛支持。

### 短期憑證的衝擊

CA/Browser Forum 已通過決議，將公開憑證最大有效期從目前超過一年逐步縮短至 **2029 年的 47 天**。

對 TLS 的影響是正面的：縮小了憑證金鑰被入侵後的影響範圍，並激勵伺服器投資自動化憑證管理。但對 CT 的影響較為負面：憑證簽發量將大幅增加，CT log 需記錄更多條目。加上「憑證爆炸」問題——每張憑證簽發通常導致 10 個以上的 CT log 條目（含預憑證、跨 log 複製等），CT log 消費者必須從所有 log 下載所有條目。

### 後量子簽章的挑戰

後量子密碼學標準已就緒：金鑰交換有 ML-KEM（已保護大量網際網路流量），簽章有 ML-DSA 和 SLH-DSA。但**後量子簽章體積巨大**：P-256 的公鑰和簽章各 64 bytes，而 ML-DSA-44 的公鑰約 1,312 bytes（約 20 倍）、簽章約 2,420 bytes（約 38 倍）。

典型 TLS 握手需傳輸至少 2 個公鑰和 5 個簽章。使用傳統密碼學約 1.2 kB，換成 ML-DSA-44 則膨脹至超過 14.7 kB。Cloudflare 的研究顯示每增加 1 kB 約造成 1.5% 的延遲。Chrome 安全團隊表示增加 7 kB 是「不可行的」，除非量子電腦確實迫在眉睫。

對 CT 的影響：更大的後量子公鑰意味著 log 需儲存更多資料，每條目簽章（SCT）的運算成本也更高。但 CT 在後量子過渡期中仍然必要——它有助於**偵測降級攻擊**。

### PLANTS 工作小組與 Merkle Tree Certificates

IETF 新成立的 **PLANTS（PKI, Logs, And Tree Signatures）** 工作小組目標是：從 TLS 等協定中**剪除**昂貴的後量子簽章，並從一開始就內建透明性。

**Merkle Tree Certificates（MTC）** 是第一個被採納的工作項目，核心理念是「**不要記錄你簽發的東西，而是透過記錄來簽發**」（don't log what you issue, issue by logging）。

MTC 的關鍵設計：
- CA 營運自己的簽發 log（良好的激勵對齊）
- 將「域名-公鑰」綁定關係作為 Merkle 樹的葉節點加入 log
- 透過簽署 tree head 即可一次認證 log 中所有宣告（signature batching）
- 獨立的 co-signers/witnesses 可跟隨 log 並簽署 tree head，提供透明性保證

MTC 產生兩類憑證：
1. **Standalone certificates**：包含 tree head 簽章 + inclusion proof，作為回退選項
2. **Landmark certificates**：**不傳送任何簽章**，僅傳送 inclusion proof（雜湊值組成）。CA 週期性地將 tree head 指定為 landmark，由瀏覽器的受信任更新服務驗證並分發至客戶端。客戶端只需 inclusion proof（預估約 700 bytes）即可驗證憑證

### Chrome-Cloudflare 實驗結果

Cloudflare 作為 TLS 伺服器與 MTC CA，Chrome 作為 TLS 客戶端：
- 已為約 1,000 個 Cloudflare 代理域名啟用
- Chrome Beta 146 的 50% 使用者已啟用（該版本近期已進入穩定版）
- Chrome 營運更新服務，驗證並分發 landmark 至客戶端

**效能結果**：
- Landmark MTC 在中位數比傳統簽章鏈**快約 9%**，P90 快約 8%
- 這僅是與傳統密碼學比較；換成後量子簽章後差距將更大
- 大多數客戶端能在數小時內取得更新，穩定狀態下僅約 1% 的客戶端過期
- 中間盒干擾目前**未構成問題**，因 TLS 1.3 加密了包含憑證的第二輪訊息

### 下一步

持續擴大實驗規模，邀請更多研究者、實作者和 CA 營運商參與 PLANTS 工作小組。尚待解決的問題包括：ACME 流程設計、log 複製、log 輪替等。

## 與 TWCA 臺灣網路認證的關聯

本演講對 TWCA 有**直接且高度相關**的影響：

1. **47 天憑證有效期**：TWCA 作為 CA，必須為 2029 年前的 47 天憑證有效期做好準備。這將大幅增加憑證簽發量，要求 TWCA 強化自動化簽發基礎設施（如 ACME 協定支援），並協助客戶建立自動化憑證管理能力。

2. **Merkle Tree Certificates 的 CA 營運模式**：MTC 要求 CA 自行營運簽發 log，這是對現行 CA 營運模式的根本性變革。TWCA 需要評估建置 MTC 簽發 log 的技術能力與基礎設施需求，並關注 Chrome 的 quantum-resistant root program 是否要求 CA 支援 MTC。

3. **後量子遷移策略**：ML-DSA-44 的簽章體積問題使得直接在傳統 X.509 憑證中使用後量子簽章變得不實際。MTC 提供了一條繞過此問題的路徑。TWCA 在規劃後量子遷移時，應將 MTC 納入技術選項。

4. **CT log 營運壓力**：短期憑證將增加 CT log 負擔。TWCA 若有營運或使用 CT log 進行憑證監控的需求，應關注 Static CT 與 MTC 帶來的架構變化，以降低營運成本。

5. **IETF PLANTS 標準化**：TWCA 應積極追蹤 PLANTS 工作小組的進展，因為其產出將直接影響未來 Web PKI 的憑證格式與簽發流程。
