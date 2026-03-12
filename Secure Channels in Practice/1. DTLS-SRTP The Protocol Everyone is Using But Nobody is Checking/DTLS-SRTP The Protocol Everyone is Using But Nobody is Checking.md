# DTLS-SRTP: The Protocol Everyone is Using But Nobody is Checking

## Session

- **Session:** Secure Channels in Practice
- **Chair:** Marie-Sarah Lacharité
- **Conference:** RWC 2026

## Speaker(s)

- **Robert Merget**

## Authors

- Martin Bach (TU Darmstadt)
- Jean Paul Degabriele (Technology Innovation Institute)
- Vukašin Karadžić (Ruhr-University Bochum)
- Lukas Knittel
- Robert Merget

## Abstract

本研究對 WebRTC 生態系統中 DTLS-SRTP 協議進行系統性安全分析，涵蓋 24 個即時通訊應用的 53 個 DTLS 實例。研究發現多項基本但嚴重的安全缺陷，包括未要求客戶端憑證、接受空憑證、未驗證指紋等問題，並成功對 WebEx、Steam、RingCentral、Vonage 等平台實施中間人攻擊，恢復通話媒體內容。Zoom 亦被廠商確認存在高嚴重性問題。

## Summary

### WebRTC 與 DTLS-SRTP 背景

**WebRTC** 是一個標準化的 JavaScript API，用於在瀏覽器中啟用安全的即時通訊應用。Google 於 2010 年提出，2011 年以開源專案發布，但直到約十年後才正式標準化。其核心加密元件是 **DTLS-SRTP**——結合 DTLS 握手與 SRTP 安全通道的混合協議。

WebRTC 的信令（signaling）部分大致未被規範，僅指定了名為 **SDP**（Session Description Protocol）的文字訊息格式，用於協商會話參數。

### 原始設計 vs 實際部署

WebRTC 原本設計為點對點協議，認證流程如下：

1. 雙方各自產生**自簽憑證**（self-signed certificate）
2. 計算憑證的**指紋**（fingerprint），透過信令通道交換
3. 使用自簽憑證進行 DTLS 握手，建立共享金鑰
4. 以共享金鑰建立 SRTP 通道傳輸媒體

然而，現今大多數應用並非點對點運作。為支援百人以上的群組通話，廠商使用**媒體伺服器**（media server）進行媒體中繼。這意味著 DTLS-SRTP 連線實際上是用戶與媒體伺服器之間的連線，**並非端對端加密**。例外如 Jitsi 等極少數應用。

### 測試方法與挑戰

研究團隊建構了一個黑盒測試框架，檢查以下基本安全屬性：

- **DTLS 標準合規性**：支援的密碼套件、演算法、SRTP profile
- **雙向認證**：伺服器是否要求客戶端憑證（DTLS 預設為單向認證）
- **空憑證接受**：伺服器是否接受空的客戶端憑證
- **指紋驗證**：握手中的憑證是否與信令階段交換的指紋進行比對
- **PKI 憑證繞過**：能否用 PKI 憑證替代自簽憑證繞過驗證

測試面臨獨特挑戰：媒體伺服器地址僅在信令完成後才知曉，且僅接受信令中指定的特定客戶端；信令協議為各廠商私有；涉及 ICE 等多種子協議；存在多條 DTLS-SRTP 連線（音訊/視訊分開）、負載均衡等複雜狀況。

### 發現的安全漏洞

在 53 個 DTLS 實例中：

| 漏洞類型 | 受影響實例數 |
|---------|-----------|
| 未要求客戶端憑證 | 7 |
| 接受空憑證 | 4 |
| 未驗證指紋 | 17 |
| 使用 512-bit 靜態 RSA 憑證（Zoho）| 1 |
| 使用非標準額外認證機制 | 3 |

### 可利用性驗證

研究團隊進一步驗證漏洞的可利用性，透過中間人攻擊阻斷受害者連線並冒充身份：

- **已確認可利用**（成功恢復通話媒體）：**WebEx、Steam、RingCentral、Vonage**（4 個應用，10 個實例）
- **完成 DTLS 握手但未完全驗證**：**Discord、Zoho、BigBlueButton**——研究者認為可透過額外技巧利用
- **廠商確認高嚴重性**：**Zoom**——研究者完成 DTLS 握手並進行實驗，Zoom 自行確認為高嚴重性問題

### 漏洞揭露的困難

多數廠商已將漏洞回報外包給漏洞獎勵平台（bug bounty platforms），這些平台常以「中間人漏洞不在範圍內」為由駁回報告。研究者不得不繞過平台，直接聯繫廠商。

### 關鍵觀察

- **瀏覽器實作相對安全**：主流瀏覽器未發現重大問題，因為瀏覽器社群深度參與了 WebRTC 的設計與標準化
- **應用開發者理解不足**：RTC 應用社群對 WebRTC 的理解遠不如瀏覽器社群
- **靈活性是安全的敵人**：WebRTC 包含大量現成元件（off-the-shelf components），雖加速開發但增加了系統複雜度與攻擊面
- **端對端加密（S-Frame）**：Q&A 中提及 S-Frame（IETF 標準，在 SRTP 內部額外加密媒體幀），但研究者表示難以逆向工程判斷各應用是否使用，WebEx 即使可能使用了 S-Frame 仍被成功攻擊

## 與 TWCA 臺灣網路認證的關聯

1. **WebRTC 認證模型的啟示**：WebRTC 使用自簽憑證搭配指紋交換進行認證，但多數廠商未正確驗證指紋。TWCA 作為 PKI 憑證機構，若客戶的即時通訊服務採用 WebRTC 架構，應關注其 DTLS 層的認證是否確實執行，而非僅依賴 TLS/PKI 層的安全性。

2. **非端對端加密的風險**：研究揭示大多數 WebRTC 應用透過媒體伺服器中繼，並非端對端加密。TWCA 在評估或推廣安全通訊解決方案時，應注意區分「傳輸加密」與「端對端加密」的差異，協助客戶理解真正的安全邊界。

3. **安全測試的重要性**：這些都是非常基本的安全檢查（是否要求憑證、是否驗證指紋），卻在主流商業應用中大量失敗。這提醒 TWCA 在自身服務及客戶整合中，應定期進行類似的基本合規性掃描，確保 TLS/DTLS 設定正確。
