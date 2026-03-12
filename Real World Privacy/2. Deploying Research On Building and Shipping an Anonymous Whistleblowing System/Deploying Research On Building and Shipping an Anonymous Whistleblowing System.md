# Deploying Research: On Building and Shipping an Anonymous Whistleblowing System

## Session

- **Session:** Real World Privacy
- **Chair:** Henry Corrigan-Gibbs
- **Conference:** RWC 2026

## Speaker(s)

- **Daniel Hugenroth** — University of Cambridge

## Authors

- Daniel Hugenroth (University of Cambridge)
- Luke Hoyland (The Guardian)
- Sam Cutler (Independent)
- Alastair Beresford

## Abstract

本演講介紹 Coverdrop，一個嵌入於新聞應用程式中的匿名吹哨系統，透過持續發送固定大小的加密掩護訊息來隱藏真實通訊的元資料（metadata），解決記者與吹哨者之間「首次接觸問題」。系統已於 2025 年 6 月在 The Guardian 的 Android 與 iOS 應用中正式上線，歷經三年從學術原型到產品部署的旅程。

## Summary

### 首次接觸問題與 Coverdrop 設計理念

吹哨者與記者之間的首次接觸面臨兩難：一方面需要低延遲、易用的通訊來建立信任；另一方面必須盡可能安全，不留下任何元資料痕跡。因為元資料一旦洩露就無法挽回——安全性可以事後降級，但無法事後追溯升級。

2018 年研究團隊舉辦工作坊，邀請記者與新聞編輯室工程師，深入了解這個問題空間後，設計出 Coverdrop 系統。

### 系統運作機制

Coverdrop 嵌入在一般新聞應用程式（The Guardian App）中，所有安裝該應用的手機都會在背景持續發送加密且填充至固定大小的掩護訊息（cover messages）。對於網路監聽者而言，所有流量看起來完全相同。

當吹哨者要發送真實訊息時：
- 等待下一次預定發送掩護訊息的時間點
- 將掩護訊息替換為真實訊息
- 由於加密且填充至相同大小，外觀與掩護訊息無異

訊息到達新聞組織後，專用的 cover node 負責丟棄掩護訊息，真實訊息被收集到「dead drops」中，記者從中下載並查找發給自己的訊息。

嵌入新聞應用的設計巧妙地解決了匿名通訊系統常見的「臨界質量」問題——從一開始就有大量使用者基礎。

### 從學術論文到產品部署

2022 年在 PETS 會議發表後，The Guardian 主動聯繫合作。原本預期數月的工作最終耗時三年，於 2025 年 6 月在 Android 和 iOS 上線。期間獲得大量專家回饋，並由 Open Tech Fund 贊助進行完整安全審計。

部署過程中的架構遠比論文複雜：
- 多個 cover node 需要故障轉移機制
- 需要獨立的身份服務（identity service）來處理金鑰輪換
- 需要處理本地服務可能斷網的情況
- 需要特殊程序來輪換長期金鑰
- 需要完善的日誌系統

### 信任邊界與密碼學選擇

核心設計原則：最小化信任邊界——非本地環境的元件僅信任其可用性（availability），不信任其機密性。

密碼學策略採用「最無聊的密碼學」：
- 協議僅依賴 libsodium 的基本原語
- 前向保密（forward secrecy）透過金鑰輪換實現
- Rust 語言的型別系統用於確保 API 安全使用
- 效能充裕允許選擇更簡單（而非更高效）的演算法與協議

### 金鑰階層與信任建立

由於不信任任何 CA，系統建立自己的金鑰階層：
1. 長期組織金鑰（long-term organization key）
2. 可較快輪換的中間金鑰（intermediate keys）
3. Cover node 與記者的身份金鑰（identity keys）
4. 實際加密訊息的訊息金鑰（messaging keys），快速輪換

信任根的建立透過帶外（out-of-band）通道：The Guardian 在實體報紙第二頁印刷公鑰摘要（digest），每個報攤都成為一個小型憑證授權中心的鏡像。

### 安全元件與密碼延展（Key Stretching）

使用者以三個單詞的口令保護本地加密儲存（vault），看似熵值不足，但透過手機的安全元件（Secure Element）進行密碼延展。

由於一般應用無法存取安全元件的認證嘗試計數器和逾時功能（這些保留給作業系統），團隊設計了「Rainbow Sloth」方案：
- 將口令雜湊為公鑰
- 利用安全元件內的私鑰進行 Diffie-Hellman 金鑰交換（該私鑰永不離開安全元件）
- 利用 DH 操作的可預測、穩定耗時作為延展基礎

### 可否認性（Plausible Deniability）

首次安裝應用時，即建立空的加密儲存區。即使裝置被沒收，攻擊者無法判斷該功能是否曾被使用。輸入錯誤口令時，應用顯示「可能輸入錯誤，或從未使用過此功能」，維持可否認性。

### 與 SecureDrop 的互補關係

Coverdrop 專注於初始接觸，僅支援小型文字訊息（不支援檔案傳輸）。對於需要傳送檔案的情境，記者可引導吹哨者使用 SecureDrop 或實體交接。在許多環境中，僅使用 Tor 本身就可能引起懷疑，而 Coverdrop 嵌入在廣泛使用的新聞應用中，大幅降低被識別的風險。

### 未來方向

- 基於 MLS 實作群組訊息，讓多位記者可協同處理同一來源
- 推動形式化驗證（formal verification）
- 後量子密碼學（post-quantum）遷移準備
- 改進發送佇列的隱私性
- 利用安全元件作為裝置端可信第三方，將可否認儲存升級為多快照抗性儲存

## 與 TWCA 臺灣網路認證的關聯

1. **自建 PKI 信任模型的實踐案例**：Coverdrop 不信任任何外部 CA，自建金鑰階層並透過實體報紙作為帶外信任錨點。這對 TWCA 思考高安全場景下的信任模型設計具有參考價值——在某些極端威脅模型下，傳統 PKI 信任鏈可能不足以滿足需求。

2. **金鑰輪換與前向保密的工程實踐**：系統的多層金鑰階層與輪換機制（組織金鑰 → 中間金鑰 → 身份金鑰 → 訊息金鑰）是 TWCA 在設計憑證生命週期管理和金鑰更新策略時的實用參考。

3. **安全元件應用於密碼延展**：利用手機 Secure Element 的 DH 操作進行金鑰延展的 Rainbow Sloth 方案，展示了在受限環境中善用硬體安全模組的創新思路，與 TWCA 推動行動裝置身分驗證（如 FIDO2/通行金鑰）的方向相關。

4. **元資料保護的重要性**：Coverdrop 強調即使通訊內容加密，元資料洩露仍可造成嚴重後果。TWCA 在提供 SSL/TLS 憑證服務時，可關注如何協助客戶降低連線元資料的暴露風險。
