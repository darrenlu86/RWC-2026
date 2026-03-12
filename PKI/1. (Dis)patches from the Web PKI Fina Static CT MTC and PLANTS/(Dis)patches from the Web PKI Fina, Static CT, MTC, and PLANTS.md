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
- Filippo Valsorda (Geomys)

## Abstract

回顧 Web PKI 從 SSL/TLS 到 Certificate Transparency（CT）、Static CT 的演進歷程，並深入介紹 Merkle Tree Certificates（MTC）作為後量子時代輕量級憑證方案的核心設計與運作原理。講者報告了 Chrome 與 Cloudflare 之間進行中的大規模 MTC 實驗結果，量化展示 landmark MTC 在 QUIC/TLS 握手中比傳統簽章鏈快約 9%（P50: 105ms vs 116ms），同時介紹 IETF 新成立的 PLANTS 工作小組如何推動標準化，以解決短期憑證與後量子簽章體積帶來的雙重挑戰。

## Summary

### Web PKI 的演進歷程：一系列的 Patches

講者以「網際網路不是一系列的管子（tubes），而是一系列的修補（patches）」為開場比喻，回顧 Web PKI 的四個關鍵里程碑：

- **1980 年代——Wild West 時期**：早期網際網路沒有廣泛的加密或認證機制，傳送敏感資料完全是「自負風險」。
- **1995 年——SSL/TLS 的誕生**：SSL v2 首次為電子商務提供加密與認證。由 CA（Certification Authority）簽發憑證，用以證明「域名與公鑰」之間的綁定關係。然而很快就有超過 100 個受信任 CA 進入客戶端根憑證庫，是否應該盲目信任所有 CA 成為根本性問題。2011 年 DigiNotar 事件——一個受信任 CA 錯誤簽發了 Google 域名的憑證——驗證了此擔憂。
- **2013 年——Certificate Transparency（CT）**：將信任模型從「盲目信任」改為「信任但驗證」（trust but verify）。所有公開 CA 簽發的憑證必須存在於 CT log 中，可供公開稽核。CT 於 RWC 2024 獲頒 Levchin 獎。其重要性在 2025 年再次獲得印證：名為 **Fina** 的 CA 錯誤簽發了 1.1.1.1（Cloudflare 加密 DNS 服務）的憑證，僅透過 CT 監控才被發現。但營運 CT log 昂貴且困難，一度僅剩五個獨立 CT log 營運商，CT 基礎設施幾乎陷入「維生系統」狀態。
- **2024 年——Static CT**：由 Filippo Valsorda 於 RWC 2024 提出（演講標題「Modern transparency logs」），將 CT log API 改為向後相容的靜態可快取資源（static, cacheable assets）。這大幅降低了營運成本，並已獲得 CT 生態系統中多個 log 實作與客戶端透明度計畫的支持。然而，即將到來的變化將使基於 CT 的 PKI 面臨更大挑戰。

### 短期憑證的衝擊與 CT 的擴展性困境

CA/Browser Forum 已通過決議（ballot），將公開 TLS 憑證最大有效期從目前的 **398 天**逐步縮短至 **2029 年的 47 天**。講者提到，就在演講當週，第一步已生效——憑證有效期降至 200 天。

**對 TLS 的正面影響**：
- 縮小了憑證金鑰被入侵後的影響範圍（blast radius）
- 強烈激勵伺服器投資自動化憑證管理，因為沒有人會想每個月手動輪替金鑰

**對 CT 的負面影響**：
- 憑證簽發量將大幅增加——短期憑證意味著需要更多張憑證
- CT log 需記錄更多條目

即使有 Static CT 的幫助，仍存在根本性的擴展問題：
- **憑證爆炸（cert explosion）**：每次 CA 簽發一張憑證，通常在整個 CT 生態系統中產生 **10 個以上的 log 條目**——包含預憑證（pre-certificate）、跨 log 複製（log spammers 將條目從一個 log 複製到另一個 log）等
- **消費端負擔**：任何 CT log 消費者（如 CT monitor）必須從**所有 log** 下載**所有條目**，以確保不遺漏任何內容

### 後量子簽章的體積挑戰

量子電腦將能破解 RSA 和 ECDSA，時間不確定但確實會來臨。

**好消息——後量子密碼學標準已就緒**：
- 金鑰交換：ML-KEM 已保護 Cloudflare **65% 以上的非機器人請求**
- 簽章：ML-DSA、SLH-DSA（以及更多在標準化管線中的方案）

**壞消息——後量子簽章體積巨大**：

| 演算法 | 公鑰大小 | 簽章大小 |
|--------|---------|--------|
| P-256 (ECDSA) | 64 bytes | 64 bytes |
| ML-DSA-44 | 1,312 bytes (~20x) | 2,420 bytes (~38x) |

**對 TLS 的影響**（投影片展示了完整的憑證鏈圖示：Root certificate -> Intermediate certificate -> Leaf certificate -> Transcript hash，涉及 2 個公鑰和 5 個簽章）：
- 協定設計者已經「上癮」於小簽章。典型 TLS 握手需傳輸至少 2 個公鑰和 5 個簽章
- 使用傳統密碼學（RSA + P-256）：約 **1.2 kB**
- 換成 ML-DSA-44（目前最佳的全方位後量子選項）：膨脹至約 **14.7 kB**
- Cloudflare 部落格研究顯示：TLS 握手每增加 **1 kB** 約造成 **1.5%** 的延遲
- Chrome 安全團隊明確表示：「增加約 7 kB 是**不可行的（implausible）**，除非密碼學相關量子電腦（CRQC）**確實迫在眉睫（tangibly imminent）**」

**對 CT 的影響**：
- 更大的後量子公鑰意味著 log 需儲存更多資料，CT log 營運商與消費者的成本增加
- 每條目簽章（SCT）的運算成本更高
- 但 CT 在後量子過渡期仍然不可或缺——Chrome 安全團隊指出 CT 有助於**偵測降級攻擊**（detect downgrades in PQ transition）

### WebPKI 需要「換盆」（Repotting），不只是更多修補

投影片的關鍵主張是：Web PKI 需要的不只是更多 patches，而是一次根本性的 **repotting**（換盆——延續植物的比喻）。

IETF 新成立的 **PLANTS（PKI, Logs, And Tree Signatures）** 工作小組正在推動此工作。講者引用了工作小組的口號：「Come for the PQ PKI, stay for the puns.」

PLANTS 的目標：
1. 從 TLS 等協定中**剪除（prune）**昂貴的後量子簽章
2. 從一開始就**內建透明性**（build in transparency from the start）

**Merkle Tree Certificates（MTC）** 是第一個被採納的工作項目。

### Merkle Tree Certificates 的核心設計

MTC 的核心理念是「**不要記錄你簽發的東西，而是透過記錄來簽發**」（Don't log what you issue, issue by logging）——類似於 Go checksum DB 的概念：如果一個 checksum 在樹中，那就是你可以信任它的方式。

**關鍵架構**：
- CA 營運自己的簽發 log（issuance log），而非將憑證提交到外部 CT log。這帶來良好的**激勵對齊**（incentive alignment）——CA 自己負責維護 log 的完整性
- 將「域名與公鑰」的綁定關係（claim）作為 Merkle 樹的葉節點加入 log
- 透過簽署 tree head 即可一次認證 log 中所有宣告——實現**簽章批次處理**（signature batching）
- 獨立的 **co-signers / witnesses / mirrors** 可跟隨 log 並簽署 tree head，提供透明性保證，確保 CA 誠實行事

**兩類 MTC 憑證**：

1. **Standalone certificates（獨立憑證）**：包含 tree head 上的簽章 + inclusion proof。作為回退選項，當客戶端未能及時更新 landmark 時使用。額外需傳輸 2-3 個簽章，成本接近傳統 X.509 + CT
2. **Landmark certificates（地標憑證）**——真正的最佳化所在：
   - **完全不傳送任何簽章**，僅傳送 inclusion proof（由雜湊值組成）
   - CA 週期性地（例如每小時）將當前 tree head 指定為 landmark
   - Landmark tree head 由瀏覽器的受信任更新服務（如 Chrome 的更新服務）驗證並**帶外分發**（out of band）至客戶端
   - 客戶端只要擁有最新的 tree head，僅需一個 inclusion proof（預估約 **700 bytes**，不到 1 kB）即可驗證憑證
   - David Benjamin（Google）有一份深入的技術概述可供參考

### MTC 對 TLS 和透明性的影響

**對 TLS 的影響**（投影片展示了與傳統憑證鏈的對比圖）：
- **Landmark 憑證（常見情況）**：握手僅需傳輸 1 個公鑰、1 個簽章、1 個 inclusion proof（< 1 kB）。整體約 3-4 kB，大幅優於後量子簽章的 14.7 kB
- **Standalone 憑證（回退情況）**：需額外傳輸 2-3 個簽章，成本接近 X.509 + CT

**對透明性的影響——不必支付「後量子稅」**：
- Log 僅儲存公鑰的**雜湊值**（hash），不隨公鑰大小而擴展
- 不再有每條目簽章（per-entry signature）——僅需在 tree head 上簽署一次，涵蓋整個 log
- 不需要保留舊簽章——只需儲存最新的 tree head 簽章
- **消除了憑證爆炸問題**：CA 的簽發 log 就是該 CA 所有已簽發憑證的唯一事實來源（single source of truth），log 消費者只需從一個 CA log（或其鏡像）取得每張憑證的單一副本
- CA 簽發 log 的實作與 Static CT log **非常相似**，可共享靜態可快取資源的架構優勢

### Chrome-Cloudflare 大規模實驗

為了儘快取得 MTC 的營運經驗，Chrome 與 Cloudflare 展開了大規模實驗，目標是在投入過多標準化工作之前驗證方案的**可行性（feasibility）**與**效能（performance）**。

**Cloudflare 端（TLS 伺服器 + MTC CA）**：
- 已為約 **1,000 個** Cloudflare 代理域名啟用
- 由於 Cloudflare 本身並非傳統 CA，採用「**bootstrap**」MTC CA 模式：僅簽發有傳統憑證鏈背書的 claim
- 此限制使實驗只能使用**傳統簽章**（classical signatures），而非後量子簽章
- 僅測試 **landmark MTC**

**Chrome 端（TLS 客戶端）**：
- Chrome Beta **146 版本的 50%** 使用者已啟用（該版本在演講期間已進入穩定版）
- Chrome 營運更新服務，負責驗證並分發 landmark 至客戶端
- 講者展示了 Chrome 憑證檢視器中 joedeblasio.com 的真實 MTC 憑證截圖——「Certificate Signature Value」欄位顯示的實際上就是 inclusion proof，沒有實際簽章被傳送

**效能量測結果**（投影片包含 QUIC/TLS 握手時間的 CDF 圖，Control 組 n=44,396、MTC 組 n=40,601）：
- **P50（中位數）**：MTC 約 **105ms** vs 傳統 **116ms**，**快約 9%**
- **P90**：MTC 約 **348ms** vs 傳統 **380ms**，**快約 8%**
- 這僅是與**傳統密碼學**比較的結果；換成後量子簽章後，效能差距預期將**更大**
- 講者認為，即使沒有量子電腦的威脅，僅憑效能提升就足以成為部署 MTC 的正當理由

**客戶端 Landmark 更新狀況**（投影片包含過期率隨時間變化的折線圖）：
- 大多數客戶端能在 **2-3 小時內**取得更新
- 實驗中 MTC 有效期為 **7 天**，因此只要客戶端在幾天內能更新即可
- 穩定狀態下約 **0.5-1.5%** 的客戶端過期（此為上界，受實驗環境特殊因素影響，實際部署預期更低）

**營運經驗**：
- 客戶端和伺服器端都排除了一些 bug，但沒有嚴重問題
- **中間盒（middlebox）干擾未構成問題**——因為 TLS 1.3 加密了第二輪（second flight）訊息，包含憑證在內，中間盒無法看到

### 下一步與社群參與

實驗將持續擴大規模，後續會發布更多部落格文章與數據。產業與社群的支持正在增長。

**如何參與**：
- **研究者、實作者、協定設計者**：加入 PLANTS 工作小組（IETF 125，2026 年 3 月 16 日開會）。尚待解決的規範工作包括：**ACME 流程設計**、**撤銷（revocation）**、**log 輪替（log rotation）**。也需要更多人幫助實作與**互通性測試（cross-testing）**
- **CA 營運商**：部署實驗性簽發系統，回報發現的問題
- Ubuntu **upki** 專案正在為 Linux 環境中的 MTC 開闢路徑，MTC 的應用不限於瀏覽器

## 與 TWCA 臺灣網路認證的關聯

本演講對 TWCA 有**直接且高度相關**的影響，涵蓋 CA 營運模式的根本性變革：

1. **47 天憑證有效期的迫切準備**：CA/Browser Forum 已通過決議，2029 年前公開 TLS 憑證有效期將縮短至 47 天，而第一步（200 天）在演講當週已生效。TWCA 作為 CA，必須大幅強化自動化簽發基礎設施（尤其是 ACME 協定支援），並主動協助客戶建立自動化憑證管理能力。憑證簽發量將成倍增長，對簽發系統的吞吐量和穩定性提出更高要求。

2. **MTC 作為 CA 營運模式的根本變革**：MTC 要求 CA 自行營運 Merkle 樹簽發 log——這不是在傳統 CT log 之外的額外要求，而是取代了傳統的 CT 提交流程。CA 的 log 成為其所有已簽發憑證的唯一事實來源。Chrome 已明確宣布其 **quantum-resistant root program 將僅支援 MTC**，這意味著 TWCA 若希望在後量子時代繼續被 Chrome 信任，必須具備營運 MTC 簽發 log 的技術能力。好消息是 MTC 簽發 log 的實作與 Static CT log 非常相似，可借鑑現有技術。

3. **後量子遷移的實務路徑**：ML-DSA-44 的簽章體積（公鑰 1,312 bytes、簽章 2,420 bytes）使得直接在傳統 X.509 憑證中使用後量子簽章變得不實際——Chrome 已明確表示增加 7 kB 是不可行的。MTC 的 landmark 憑證模式提供了一條繞過此體積問題的路徑，將握手負擔控制在 3-4 kB。TWCA 在規劃後量子遷移策略時，應將 MTC 作為核心技術選項而非備選方案。

4. **CT log 營運成本與架構轉型**：短期憑證將導致憑證爆炸問題加劇（每張憑證在 CT 生態系統中產生 10+ 條目）。MTC 從根本上消除了此問題——CA 的簽發 log 即為唯一來源，消費者只需取得一份副本。TWCA 若有營運或使用 CT log 進行憑證監控的需求，應關注從傳統 CT 到 MTC 的架構轉型，以大幅降低長期營運成本。

5. **IETF PLANTS 標準化的積極追蹤**：PLANTS 工作小組的產出將直接決定未來 Web PKI 的憑證格式與簽發流程。尚待定義的規範包括 ACME 流程（影響自動化簽發）、撤銷機制、log 輪替等。TWCA 應考慮派員參與或密切追蹤 PLANTS WG 的進展，以確保在標準定案前掌握技術方向並準備合規。

6. **實驗性部署的機會**：講者明確呼籲 CA 營運商部署實驗性 MTC 簽發系統。Chrome-Cloudflare 實驗已證明 MTC 可行且效能優於傳統方案（P50 快 9%）。TWCA 可考慮以小規模試點方式參與，累積第一手營運經驗，並在標準正式定案前搶得先機。
