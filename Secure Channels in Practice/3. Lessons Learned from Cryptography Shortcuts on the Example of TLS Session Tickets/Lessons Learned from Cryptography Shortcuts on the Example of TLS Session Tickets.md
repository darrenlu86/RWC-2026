# Lessons Learned from Cryptography Shortcuts on the Example of TLS Session Tickets

## Session

- **Session:** Secure Channels in Practice
- **Chair:** Marie-Sarah Lacharité
- **Conference:** RWC 2026

## Speaker(s)

- **Sven Hebrok** — Paderborn University

## Authors

- Sven Hebrok (Paderborn University)
- Tim Leonhard Storm (achelos GmbH)
- Felix Matthias Cramer (Ruhr University Bochum)
- Maximilian Radoy (Technology Innovation Institute)
- Simon Nachtigall
- Marcel Maehren
- Nurullah Erinola
- Juraj Somorovsky
- Robert Merget
- Jörg Schwenk

## Abstract

本研究系統性分析 TLS Session Ticket 機制中的安全陷阱，發現多項已知但反覆出現的攻擊：包括約 2,000 台伺服器使用全零加密金鑰（含 AWS）、CDN 環境中的跨域 session 重定向攻擊、以及利用 session resumption 繞過客戶端認證的攻擊。研究進一步追溯問題根源至 OpenSSL API 設計的複雜性與歷史演變，提出加密函式庫應強制安全預設行為的建議。

## Summary

### TLS Session Ticket 機制回顧

TLS 使用混合加密：先透過 Diffie-Hellman 金鑰交換建立對稱金鑰，再驗證伺服器（可選驗證客戶端），最後切換至對稱加密傳輸應用資料。然而，每次新連線都需重新執行公鑰加密操作，成本高昂。

**Session Resumption**（會話恢復）機制解決此問題：利用前一次會話已建立的金鑰材料，衍生新金鑰，跳過認證與金鑰交換步驟。實務上透過 **Session Ticket** 實作——伺服器將對稱金鑰加密後放入 ticket 發送給客戶端，加密所用的金鑰稱為 **Session Ticket Encryption Key (STEK)**。此金鑰應僅伺服器知曉，因為它保護了金鑰材料。

### 攻擊一：弱金鑰 / 全零金鑰

若攻擊者能解密 ticket，後果嚴重：

- **TLS 1.2**：可**被動解密所有流量**，包括第一次會話（即使 ticket 未被恢復）
- **TLS 1.3**：僅影響金鑰交換前的資料，影響較小
- **TLS 1.2 與 1.3**：主動攻擊者均可**冒充伺服器**

研究者掃描開源實作後未發現問題，但在真實世界中發現約 **2,000 台伺服器使用全零金鑰**。其中最大宗屬於 **AWS**，AWS 回應這並非 TLS 實作的 bug，而是管理 STEK 的實作在記憶體未正確初始化時使用了零值。此漏洞已修復。

### 攻擊二：CDN 環境中的跨域 Session 重定向

正常 TLS 中，若流量被重定向到另一台伺服器，客戶端會因憑證不符而拒絕連線。但在 session resumption 中**不驗證憑證**——理論上其他伺服器不持有 STEK 所以無法處理 ticket，會回退到完整握手。

然而在 **CDN**（內容遞送網路）環境中，多台伺服器共享 STEK。此攻擊在 2015 年就已被發現（Mozilla 的 Git 伺服器與 bug tracker 共享 session cache，攻擊者可將不可信內容注入可信空間）。

研究者在 2025 年重新分析，發現**多個 CDN 仍受此攻擊影響**。攻擊場景：付費取得 CDN 的專用 IP，使該 IP 永遠指向攻擊者的 evil.com。當客戶端的流量被重定向到此 IP 時，CDN 可能仍接受來自其他客戶的 ticket，客戶端因 session resumption 跳過憑證驗證而信任回應內容。另外在 Cloudflare 發現了一個不相關但影響相同的漏洞。

### 攻擊三：繞過客戶端認證

Session resumption 跳過認證——包括**客戶端認證**。攻擊流程：

1. 伺服器同時託管公開網域（public.com）與私有網域（private.com，需客戶端憑證認證）
2. 攻擊者先連線到 public.com，取得 session ticket
3. 攻擊者持 ticket 連線到 private.com，伺服器認為「有 ticket 代表之前已認證過」
4. 成功繞過客戶端認證存取私有資源

此攻擊早在 **1999 年**就被 OpenSSL 發現並修補——引入 **session ID context** 機制，要求 ticket 發行時設定上下文，恢復時比對上下文不同則拒絕。

但研究者在 2025 年分析四個開源伺服器實作，**全部四個都存在漏洞**，需要特定的測試向量（變更 SNI 與 Host header 的組合）。特別值得注意的是 **TLS 1.3 有時反而比 1.2 更脆弱**。在 Cloudflare 也發現了可實際利用的漏洞——攻擊者可在 Cloudflare 註冊網域，取得 ticket 後在其他客戶的網域上恢復 session，繞過客戶端認證。

### OpenSSL API 的陷阱

研究者嘗試自行使用 OpenSSL 實作伺服器，發現多個 API 陷阱：

1. **Key Callback**：若因程式 bug 回傳全零金鑰，OpenSSL 不會拒絕，照常使用
2. **Client Verify Callback**：用於驗證客戶端憑證的回呼函式，在 **session resumption 時不會被呼叫**——OpenSSL 認為憑證已在之前驗證過。開發者若不知道 session resumption 的存在，測試時一切正常但實際不安全
3. **SNI Callback 的版本變更**：
   - 2018 年前：在 SNI callback 中設定 session ID context 即可
   - TLS 1.3 引入後：必須使用新的 callback，舊的 SNI callback 中設定的 session ID context **被忽略**（因為時機太晚）
   - 2018 年前撰寫的遺留軟體若未更新，仍然脆弱
   - 關鍵差異：1999 年的修補是**功能性破壞**（不改程式碼就無法編譯），2018 年的變更是**靜默的安全性破壞**（程式碼正常編譯但不再安全）

### 根本性的協議設計問題

- **不可審計的金鑰**：Session ticket 對客戶端完全不透明——客戶端不知道收到的是什麼，只是原樣回傳。STEK 的格式與內容完全由伺服器自行決定，客戶端無法審計金鑰品質。相比之下，TLS 中的其他金鑰要麼是非對稱的（可檢查公鑰），要麼由雙方隨機性共同衍生
- **真實世界的複雜度**：標準假設「一個客戶端對一個伺服器」，但實際部署中有 CDN、中間箱（middlebox）、多域名共用伺服器等，標準對這些場景缺乏明確指引
- **SNI 與 Host Header 的不一致**：TLS 層的 SNI 與 HTTP 層的 Host header 都指定目標主機名，當兩者不一致時該如何處理，標準並未清楚規範

### 建議與教訓

1. **TLS 並非即插即用**：錯誤配置或誤用函式庫仍會導致漏洞
2. **加密函式庫應強制安全行為**：
   - 驗證金鑰材料（如拒絕全零金鑰，除非開發者明確多次確認）
   - 為 session ID context 產生安全預設值
   - 若出錯應使用協議的回退機制（如回退到完整握手）
3. **API 變更應破壞性地強制更新**：靜默的安全性變更比功能性破壞更危險
4. **協議設計應考慮部署環境**：包括 CDN、負載均衡、多域名等實際場景
5. **使金鑰可審計**：單方決定且不可審計的金鑰是安全隱患

## 與 TWCA 臺灣網路認證的關聯

1. **TLS Session Ticket 管理的直接影響**：TWCA 營運的各項網路服務（OCSP 回應伺服器、CRL 發佈點、憑證管理平台等）均使用 TLS。本研究揭示的 STEK 管理問題——特別是全零金鑰、跨域 ticket 共享——是 TWCA 伺服器運維團隊應立即檢查的項目。應確認 STEK 是否定期輪換、是否有適當的隨機性驗證。

2. **CDN 部署的安全考量**：若 TWCA 使用 CDN 加速其服務，本研究的跨域 session 重定向攻擊直接相關。應確認 CDN 配置中 session ticket 的隔離機制，避免攻擊者透過同一 CDN 上的其他域名取得可跨域恢復的 ticket。

3. **客戶端憑證認證的安全性**：TWCA 的多項服務（如 RA 系統、憑證申請平台）可能使用 TLS 客戶端憑證進行身分驗證。本研究證明 session resumption 可繞過客戶端認證，TWCA 應檢查其伺服器是否正確實作了 session ID context，特別是使用 OpenSSL 且支援 TLS 1.3 的環境，必須使用 2018 年後引入的新 callback。

4. **對加密函式庫選擇的啟示**：研究揭示 OpenSSL API 的複雜性與歷史包袱可能導致安全問題。TWCA 在選擇與更新加密函式庫時，應關注 API 的安全性預設行為，並在 TLS 版本升級時重新審視 session resumption 相關的設定是否仍然安全。
