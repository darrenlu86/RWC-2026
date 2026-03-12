# Defending the Open Internet: TLS, Passkeys, and the Privacy Stakes of Digital Identity

## Session

- **Session:** Invited Talk
- **Chair:** 
- **Conference:** RWC 2026

## Speaker(s)

- **Christopher Harrell** — Yubico

## Authors

- Christopher Harrell (Yubico)

## Abstract

從 TLS 加密到 FIDO Passkeys 再到數位身分，講者以二十年實戰經驗回顧密碼學如何從標準走向大規模部署，並指出當前數位身分系統在隱私、後量子安全與主權需求之間的嚴峻挑戰，呼籲密碼學社群加速研發可部署的零知識證明與後量子憑證方案，避免預設監控式架構成為全球數位身分的基礎。

## Summary

### 從 TLS 加密的成功經驗談起

講者 Christopher Harrell 來自 Yubico，以二十年參與密碼學部署的經驗開場。他回顧了 TLS 加密的發展歷程：SSL 技術早已存在，但真正推動大規模部署的是 Snowden 事件帶來的槓桿效應（lever）。加密的成功不只靠數學，而是**標準、協定、實作、UX、政策、認證**等多重因素同時到位的結果。這段歷程長達三十年，至今仍在持續改進。

### FIDO / Passkeys 的發展歷程

FIDO 的起點是 2009 年 Operation Aurora 事件——國家級駭客攻擊 Google 等企業。Google 內部一位曾生活在威權政權下的領導者決定大力投資解決此問題，最終催生了 FIDO U2F 標準。

**關鍵成果**：Google 在 85,000 名員工部署安全金鑰後，實現**零次成功釣魚攻擊**。目前全球已有超過**十億帳戶**使用 Passkeys。

講者分享了一個重要洞察：金融服務公司的產品經理對 FIDO 最感興趣的不是安全性，而是**減少登入摩擦**——未登入的使用者是不快樂的使用者。這說明技術的採用動力往往不在安全本身。

### Passkeys 的技術演進

- **硬體綁定 vs 同步 Passkeys**：同步憑證（synced credentials）的出現是關鍵轉折，同一標準能同時服務高安全需求（硬體綁定）和一般使用者（同步），這在標準化歷史上非常罕見
- **PRF 與 HMAC Secret 擴展**：用於加密功能，已有產品建構於此之上
- **ARKG（Asynchronous Remote Key Generation）**：來自密碼學領域，用於 YubiKey 5.8 Preview 的數位簽署方案，支援驗證者不可連結（verifier unlinkable）的數位憑證展示
- **Dub Wallet（即將更名 Cirros ID）**：開源的 Web 原生身分錢包，由 Yubico 創辦人成立的 Cirrus Foundation 主導

### 數位身分的緊迫危機

講者指出數位身分正面臨立法驅動的快速部署壓力：

**瑞典 BankID 的案例**：基於銀行聯盟運營的數位身分系統，幾乎涵蓋所有生活服務（稅務、學校、交通、健身房、房屋買賣、開設公司）。極為便利但**不具備隱私保護特性**——這就是當前已部署系統的現狀。

**全球現況**：數十個國家已部署或正在試點數位身分系統。美國有約 19-20 個州推動年齡驗證法規，但沒有全國性數位身分方案，形成矛盾。東南亞的方案多為中央化 QR Code 查詢，完全可追蹤。

### 零知識證明與後量子的核心矛盾

講者列舉了多種基於古典密碼學（ECDSA、BLS、Schnorr）的盲化方案，可實現零知識選擇性揭露（selective disclosure）。這些方案大多具備**永恆隱私**（everlasting privacy）——即使擁有無限運算能力，隱私保證仍然成立。

**核心問題**：所有這些方案在**後量子電腦出現時將喪失不可偽造性**（forgeability）。

- 後量子方案如 ML-DSA 使用 SHAKE 雜湊，對零知識證明極不友好
- 目前**不存在同時具備後量子安全、永恆隱私、可實際部署的憑證方案**
- 各國要求 2027-2030 年部署後量子方案，但技術尚未就緒

當外交官們在會議室討論需求優先序時，**隱私總是第一個被犧牲的**，其次是後量子，再來是主權。

### 主權與 FIDO 的矛盾

部分國家因 FIDO 的 Hybrid 協定需要經過可能不在本國的網路中繼（network relay）而拒絕部署。而 Hybrid 協定正是 Digital Credentials API 的基礎，這成為數位身分推進的障礙。

### 行動呼籲

1. **加入 IETF ZIP（Zero-knowledge Identity Proofs）郵件列表**：新成立的討論群組，歡迎數學、協定、UX 各領域專家參與
2. **將零知識方案帶到 Dub Wallet / Cirros ID**：已有 Longfellow 的獨立實作即將合併，對 Vega（Crescent 的後續方案）也有高度興趣
3. **協助研發可在硬體中實現的後量子憑證方案**：不只是後量子展示（presentation），更需要後量子憑證（credential）本身

## 與 TWCA 臺灣網路認證的關聯

本場演講對 TWCA 有多層面的重要啟示：

1. **數位身分與 PKI 信任架構的交會**：TWCA 作為 PKI/SSL 憑證與電子簽章的核心機構，數位身分的全球趨勢將直接影響 TWCA 的業務方向。講者描述的歐盟 eIDAS 2.0 框架要求錢包供應商（Wallet Provider）發行具備特定安全屬性的錢包實例，這與 TWCA 的憑證發行角色高度相關。

2. **Passkeys 與身分驗證的融合**：FIDO Passkeys 正在從認證（authentication）擴展到身分（identity）領域。TWCA 的身分驗證服務應評估整合 Passkeys 的可行性，特別是 PRF 擴展和 ARKG 簽署方案，這些技術能在不犧牲隱私的前提下實現選擇性揭露。

3. **後量子遷移的急迫性**：講者明確指出當前所有基於橢圓曲線的零知識方案都無法抵禦後量子攻擊。TWCA 在規劃後量子遷移路徑時，不僅要考慮 TLS 和簽章演算法的更新，更需關注未來數位身分憑證的後量子安全性。

4. **隱私保護的設計原則**：講者強調「不建構預設監控機制」的重要性。TWCA 在設計或參與數位身分方案時，應優先採用具備選擇性揭露和不可連結性的技術，而非傳統的全量揭露模式，以符合國際隱私標準的演進方向。

5. **臺灣數位身分的定位**：講者提及東南亞多數數位身分方案缺乏隱私保護，TWCA 有機會在臺灣推動更符合隱私原則的數位身分基礎設施，借鏡歐盟經驗，避免部署「永久可追蹤」的架構。
