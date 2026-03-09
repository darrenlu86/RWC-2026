# Improving the Trustworthiness of Javascript on the Web

## Session

- **Session:** Beyond Secure Messaging
- **Chair:** Cas Cremers
- **Conference:** RWC 2026

## Speaker(s)

- **Michael Rosenberg**
- **Giulio Berra** — Freedom of the Press Foundation
- **Ezzudin Alkotob** — Meta
- **Dennis Jackson** — Cloudflare

## Authors

- Ezzudin Alkotob (Meta)
- Giulio Berra (Freedom of the Press Foundation)
- Benjamin Beurdouche (Mozilla)
- Richard Hansen (Meta)
- Daniel Huigens (Proton AG)
- Dennis Jackson (Cloudflare)
- Cory Francis Myers (Freedom of the Press Foundation)
- Michael Rosenberg (Cloudflare)

## Abstract

The web is the universal application platform, but there are some applications you cannot safely run in a browser today. Consider an end-to-end encrypted messenger whose encryption code executes entirely client-side in Javascript. There is nothing preventing the server from sending a specific user some malicious Javascript code to exfiltrate all their messages. Further, this attack would be effectively undetectable by the user, and leave no trace.

Smartphone apps don't have this problem. This is because app stores perform lots of security-specific operations: they scan for known malware, they verify developers' code signing, they show developers all apps deployed under their developer ID, and they distribute the same binary to every end user. It would be nice if we could achieve app store-like guarantees for web applications, without the centralizing effects of a private app store.

In this talk, we discuss efforts among browser vendors, cloud providers, and encrypted communication developers to bring app store-like security guarantees to the entire web. We will present an overview of already deployed systems—CodeVerify by Meta and WEBCAT by the Freedom of the Press Foundation—as well as a developing standard called Web Application Integrity, Consistency, and Transparency (WAICT), which aims to unify these approaches, embed them natively in browsers, and scale these guarantees across the whole web.

## Summary

### 核心問題

當你用瀏覽器開啟端對端加密的通訊服務（如 Signal Web），伺服器本身就提供了 JavaScript 程式碼。如果伺服器被入侵或惡意行為，它可以送出竄改的 JS，直接竊取明文訊息——而使用者完全無法察覺。

這不只影響 E2EE 通訊，也影響瀏覽器中的密碼管理器、TEE 上的 LLM 前端、線上投票等所有依賴瀏覽器執行安全邏輯的場景。這個問題已知超過十年。

### 對比：為什麼原生 App 可以信任？

原生 App（如 Signal iOS）透過 App Store 具備三個保障：

- **Integrity（完整性）**：二進位檔有開發者和商店的簽章
- **Consistency（一致性）**：全球使用者拿到相同版本
- **Transparency（透明性）**：開發者可看到已發布的應用

關鍵問題：如何在開放網頁上達到 App Store 等級的保障，又不引入中心化控制？

### 三個方案

#### 1. CodeVerify（Meta，已部署）

- Meta 發布 Web App 時，產生包含所有 JS/CSS 雜湊的 **manifest**（Merkle Tree 結構）
- Root hash 發送到 **Cloudflare** 存放作為獨立第三方驗證
- 開源的瀏覽器擴充套件即時 hash 頁面上每個腳本，比對 manifest，再跟 Cloudflare 交叉驗證
- 額外檢查 CSP headers（禁止 eval、inline script）及 cache 完整性
- 挑戰：Facebook.com 有數十萬個 JS 資源，用主 manifest + 長尾 manifest 分層處理

#### 2. WebCat（Freedom of the Press Foundation，alpha 階段）

- 為**任何網站**提供程式碼簽章，專為 Tor Browser 等高安全需求場景設計
- 建立分散式共識系統，網站管理員可註冊並記錄信任政策（基於 SIGSUM 或 Sigstore）
- 每日快照簽章後發送到瀏覽器擴充套件的 preload list（每筆僅 64 bytes）
- **Fail-close 設計**：完整性驗證失敗時直接阻止載入，不只是警告
- **隱私保護**：導航時不需聯絡第三方，所有資料從同一 origin 拉取
- 驗證範圍更廣：不只 JS，還包含 HTML、CSS、圖片、翻譯檔、CSP headers 等

#### 3. WACT — Web Application Integrity, Consistency, and Transparency（標準化提案，早期階段）

- 試圖統一 CodeVerify 和 WebCat 的方法，直接整合進瀏覽器
- 網站透過 **HTTP header**（類似 HSTS）主動 opt-in
- 使用 **Merkle Patricia Tree**（字典序排列），讓稽核者可用 O(log n) 查詢特定 domain 的 manifest，大幅降低頻寬成本
- 設計**緊急退出機制（break glass）**：網站出問題時可 opt-out，但 opt-out 本身也會被記錄到 transparency log
- 已有 Firefox Nightly 實作，以及 Cloudflare Orange Meets（E2EE 視訊）整合
- 規格草案公開在 GitHub，可到 **wact.dev** 試用 demo

## 與 TWCA 臺灣網路認證的關聯

本場演講的核心——為網頁應用建立**完整性、一致性與透明性**保障——與 TWCA 作為憑證機構的業務高度相關：

1. **Transparency Log 營運者角色**：WACT 標準需要獨立的第三方營運 transparency log（類似 Certificate Transparency 的角色）。TWCA 已有營運 CT log 的經驗與基礎設施，可直接延伸為 WACT transparency log 的營運方，為台灣及亞太區的網頁應用提供可稽核的 manifest 紀錄服務。

2. **程式碼簽章服務**：CodeVerify 與 WebCat 都依賴開發者對網頁資源的簽章。TWCA 現有的 code signing 憑證服務可擴展至網頁應用場景，為開發者簽發用於 WACT 的簽章憑證。

3. **信任錨點（Trust Anchor）**：WACT 的設計需要瀏覽器信任特定的 transparency service。TWCA 作為已被主流瀏覽器信任的根憑證機構，具備成為 WACT 信任錨點的先天優勢。

4. **台灣政府數位服務應用**：台灣政府的線上報稅、健保、戶政等網頁服務處理高度敏感資料，WACT 機制可確保這些服務的前端程式碼未被竄改。TWCA 可作為這些政府網頁應用的 transparency service 提供者，強化公民對政府數位服務的信任。
