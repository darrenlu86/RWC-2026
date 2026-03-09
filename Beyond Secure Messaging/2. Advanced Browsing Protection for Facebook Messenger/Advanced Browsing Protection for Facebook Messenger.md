# Advanced Browsing Protection for Facebook Messenger

## Session

- **Session:** Beyond Secure Messaging
- **Chair:** Cas Cremers
- **Conference:** RWC 2026

## Speaker(s)

- **Emma Connor** — Meta
- **Kevin Lewi** — Meta

## Authors

- Emma Connor (Meta)
- Artem Ignatyev (Meta)
- Kevin Lewi (Meta)

## Abstract

Facebook Messenger recently added a new feature called Advanced Browsing Protection, which is based around a private information retrieval (PIR) protocol that is triggered when a user clicks on a link in a Messenger end-to-end encrypted chat thread. The user's link is matched against a large server-side database of malicious links, without revealing the plaintext link or the outcome of the match to the server.

We will discuss the design decisions we made and also the challenges we faced when realizing this cryptographic primitive in practice.

## Summary

### 要解決的問題

Facebook Messenger 已全面支援端對端加密（E2EE），Meta 無法讀取訊息內容。但這也代表無法在伺服器端掃描惡意連結。需要一個方案既能阻擋惡意 URL，又不洩露使用者點了什麼連結。

### 核心設計：隱私保護的 URL 過濾

**截斷雜湊 + OPRF**

- 將惡意 URL 資料庫依 16-bit 雜湊前綴分成 bucket
- 用戶端傳送 bucket ID，伺服器回傳整個 bucket 內容
- 用 OPRF（Oblivious PRF）防止用戶端反推資料庫其他條目，阻止離線枚舉攻擊

**前綴匹配（Prefix Matching）**

- 封鎖名單常是整個 domain 或路徑前綴，而非單一精確 URL
- 將同一 domain 的 URL 組織到同一 bucket 中，用戶端只需請求一次
- 額外的 URL segments 以 blinded OPRF values 傳送，不洩露明文
- 設計 **Hashing Rule Set**（約 10-20KB，用戶端預存）拆分短網址服務等問題 domain 的超大 bucket

### 隱藏 Bucket ID 的多層防護

1. **TEE（可信執行環境）**：查詢伺服器運行在 AMD SEV-SNP 機密虛擬機（CVM）中，用戶端用 TEE 公鑰加密 bucket ID
2. **Attestation 驗證**：AMD 硬體認證 + Cloudflare append-only log 確保用戶端可驗證 TEE 的可信性
3. **Path ORAM**：隱藏記憶體存取模式，防止旁路攻擊推測查詢了哪個 bucket（實作已開源）
4. **Oblivious HTTP**：透過第三方 proxy 剝離用戶端 IP，伺服器不知道是誰發出的查詢

### 安全保證

即使攻擊者完全攻破 TEE，最多只能得到 **16-bit 雜湊前綴**，而非實際 URL。

### 未來方向

- 探索更先進的 PIR 密碼學方案，將前綴從 16 bits 縮小到 8 bits 甚至更少
- 改進 ORAM 效率

## 與 TWCA 臺灣網路認證的關聯

本場演講的信任架構雖以硬體 TEE 為核心，但其中多個環節與 TWCA 的業務領域有交集：

1. **Attestation 與憑證信任鏈的同構性**：TEE attestation 的本質是「硬體簽發憑證證明自身可信」，形成 AMD 硬體根 → attestation report → 應用層公鑰的信任鏈，這與 CA 體系的根憑證 → 中繼憑證 → 終端憑證結構同構。TWCA 在憑證信任鏈的設計與管理經驗，可直接應用於評估或參與 TEE attestation 的驗證架構。

2. **Append-only Log 與 Certificate Transparency**：Meta 使用 Cloudflare 的 append-only log 來記錄 TEE attestation 紀錄，讓用戶端可以驗證 TEE 的可信性。這與 TWCA 已參與的 Certificate Transparency（CT）log 運作模式完全一致——都是透過公開可稽核的日誌來防止信任方（CA 或 TEE）的不當行為。

3. **Oblivious HTTP 的信任第三方**：方案中的 OHTTP proxy 負責剝離用戶端 IP，這個角色需要獨立於服務提供者的可信第三方。TWCA 作為具備公信力的第三方機構，在類似的隱私保護架構中可擔任 proxy 營運者角色。
