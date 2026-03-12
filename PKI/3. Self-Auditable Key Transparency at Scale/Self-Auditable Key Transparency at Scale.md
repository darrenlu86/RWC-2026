# Self-Auditable Key Transparency at Scale

## Session

- **Session:** PKI
- **Chair:** Paul Grubbs
- **Conference:** RWC 2026

## Speaker(s)

- **Hossein Hafezi** — New York University

## Authors

- Hossein Hafezi (New York University)
- Alireza Shirzad (University of Pennsylvania)
- Benedikt Bunz
- Joseph Bonneu

## Abstract

針對端對端加密通訊中公鑰分發的信任問題，提出基於多項式承諾方案（polynomial commitment schemes）的新型 Key Transparency 設計。新方案 ArND 和 Agon 實現了自我稽核能力（self-auditability）——將 WhatsApp 現行方案中每分鐘需下載 80 MB 的稽核負擔降至常數大小的 8 kB 證明，同時達成完美隱私與歷史修剪，可擴展至數十億用戶規模。

## Summary

### Key Transparency 的動機

在端對端加密通訊應用（如 WhatsApp、iMessage、Signal）中，核心問題是**公鑰分發**：當 John 知道 Egret 的手機號碼，如何安全地取得 Egret 的公鑰？如果信任服務提供者來分發公鑰，服務提供者可能惡意替換公鑰（例如給 John 一個攻擊者的公鑰而非 Egret 的），導致中間人攻擊。

目前 iMessage、WhatsApp 和 Signal 支援手動的帶外驗證（掃描 QR 碼或口頭比對字串），但這些方案不實用：需要親自見面，且每次公鑰變更都需重新驗證所有聯絡人。

**Key Transparency** 提出自動化解決方案，已被 Keybase、iMessage 和 WhatsApp 部署。

### Key Transparency 運作模型

核心架構包含：
- **服務提供者**維護公鑰目錄，週期性地在公開佈告欄（bulletin board）上發布對目錄的**簽署承諾（signed commitment）**及輔助資料（invariance proof）
- **查詢（Lookup）**：John 查詢 Egret 的最新公鑰，收到公鑰和證明，可驗證收到的公鑰與佈告欄上的承諾一致
- **一致性證明（Proof of Consistency）**：證明某公鑰在 epoch i 到 epoch j 之間未被變更

一致性證明至關重要，因為需要防禦**幽靈金鑰攻擊（ghost key attack）**：當用戶離線時，服務提供者惡意將其公鑰替換為攻擊者的公鑰，在用戶上線前再換回來。若用戶僅檢查離線前後的公鑰是否相同，將無法偵測此攻擊。

### 現有方案的三大缺陷

**1. 缺乏自我稽核能力（Self-Auditability）**

現有部署方案（Seamless、Parakeet、Optics）使用**版本不變量（version invariant）**來實現一致性證明：每個公鑰有版本號，更新時版本加一，檢查兩個 epoch 的版本是否相同即可。但確保版本號誠實遞增的**不變量證明（invariance proof）**規模與該 epoch 的更新數量成線性關係。

具體數字：WhatsApp Key Transparency 目前的稽核負擔相當於**每分鐘下載和雜湊 80 MB 資料**，對手機用戶完全不可行。因此 WhatsApp 將稽核任務委託給 **Cloudflare**——數十億用戶信任單一第三方。雖有使用 zk-SNARK 壓縮不變量證明的嘗試（Verdict、Hackathon），但因 prover 成本過高而無法擴展。

**2. 缺乏完美隱私（Perfect Privacy）**

基於 Merkle 樹的方案在查詢時會洩漏額外資訊。例如 Seamless（WhatsApp 的基礎方案）在每次查詢時會揭露**金鑰最後一次更新的時間**。攻擊者可藉此識別長期不更換金鑰的用戶作為潛在目標。

**3. 缺乏歷史修剪（History Pruning）**

基於 Merkle 樹的方案使用 append-only 結構，所有歷史資料永久保留，資料結構無限增長。WhatsApp 目前有 30-40 億用戶，但底層資料結構已超過 **1,000 億個節點**。

### ArND 與 Agon：基於多項式的新範式

研究者提出以**多項式和多項式承諾方案（polynomial commitment schemes）**為基礎的新設計範式，利用多項式的代數性質同時解決上述三個問題。

**ArND** 是第一個作品：
- **自我稽核**：對於 10 億用戶規模的字典，不變量證明為**常數大小的 8 kB**，與 epoch 中更新數量無關
- **快速前進（Fast Forwarding）**：稽核者可離線任意時間，上線後只需驗證一個常數大小的證明即可追上整個歷史——稽核者不需一直在線
- **完美隱私**：查詢和一致性證明不洩漏額外資訊（類似零知識的定義）
- 可擴展至 10-20 億用戶，但不支援水平擴展

**Agon** 在 ArND 基礎上解決了所有剩餘問題：
- 支援**水平擴展**，可擴展至 1,000 億級別
- 降低了 epoch 延遲
- 改善了儲存效率
- 不需要線性大小的 setup

### 技術核心思路

將每個 epoch 的字典建模為公鑰向量，定義 delta 向量為連續兩個狀態的差。一致性證明等價於檢查所有中間 delta 向量在特定索引處為零。透過將 delta 向量編碼為多項式，利用多項式承諾方案的代數性質（如 KZG），可將多個 epoch 的一致性檢查壓縮為常數大小的證明。

引入**隨機版本不變量（random version invariant）**概念：每個版本分配一個隨機值（而非遞增整數），利用隨機性將版本檢查轉化為多項式等式檢查，進一步實現完美隱私和高效的聚合證明。

## 與 TWCA 臺灣網路認證的關聯

本演講與 TWCA 的核心 PKI/SSL 憑證業務無直接關聯——Key Transparency 主要應用於端對端加密通訊應用的公鑰分發問題，而非 Web PKI 中的憑證簽發與管理。

然而，存在以下間接啟示：

1. **透明性架構的技術趨勢**：Key Transparency 與 Certificate Transparency 共享「透明性」的核心設計理念——透過公開可稽核的資料結構建立信任。本研究展示了從 Merkle 樹到多項式承諾方案的技術演進，這些密碼學工具未來可能也會影響 CT 或其他 PKI 透明性機制的設計。

2. **身分驗證服務的延伸**：TWCA 的身分驗證業務若未來延伸至端對端加密通訊的公鑰託管或驗證服務，Key Transparency 的技術將直接相關。自我稽核能力意味著不再需要依賴單一第三方進行稽核，這與 TWCA 作為受信任第三方的角色定位有潛在互動。

3. **隱私保護的設計啟示**：完美隱私的設計目標——查詢和證明不洩漏額外資訊——對 TWCA 在設計任何涉及用戶查詢的透明性服務時，提供了隱私保護的參考架構。
