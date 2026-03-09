# Levchin Prize Invited Talk: Automated Protocol Analysis using Tamarin

## Session

- **Session:** Invited Talk — Levchin Prize Awardee
- **Conference:** RWC 2026

## Speaker(s)

- **Cas Cremers** — CISPA Helmholtz Center for Information Security

## Authors / Development Core

- David Basin (ETH Zurich)
- Cas Cremers (CISPA)
- Jannik Dreier (LORIA, France)
- Ralf Sasse (ETH Zurich)

## Abstract

Levchin Prize 得主 Cas Cremers 受邀演講，介紹 Tamarin Prover——一個用於自動化安全協議形式化驗證的開源工具。演講涵蓋 Tamarin 的核心技術、工作流程、實際應用案例，以及形式化分析在未來安全領域的角色。

## Summary

### Tamarin Prover 是什麼

Cas Cremers 用一句話總結：「Tamarin 是一個**偽裝成定理證明器的約束求解器**（a constraint solver impersonating as a theorem prover）。」

Tamarin 是一個開源的安全協議形式化驗證工具（tamarin-prover.com），主要運作在**設計層級**（design level），針對**安全性質**（security properties）進行分析。專案始於 2008 年，開發核心團隊分布在 CISPA（德國，Cremers、Robert Künnemann）、ETH Zurich（瑞士，David Basin、Ralf Sasse）與 LORIA（法國，Jannik Dreier）。Logo 來自南美洲的皇帝絹毛猴（Emperor Tamarin），因其標誌性的鬍子而被選為吉祥物。

### 安全協議分析為何困難

現代安全協議（如 TLS、Signal、WireGuard、Wi-Fi）面對的威脅模型中，攻擊者可以竊聽、注入、加密、解密、簽署、雜湊，甚至入侵部分參與方。需要證明即使在網路中存在任意數量的不受信任實例，誠實的通訊雙方仍能安全通訊。正是因為 session 數量無上界，使得這個問題在本質上是**不可判定的（undecidable）**。

### 分析層次

Tamarin 區分三個層次的協議分析：

1. **設計層（Design）**：如 IETF RFC 或 ISO 標準的抽象訊息流
2. **語言實作層（Language-specific Implementation）**：如 Rust、Go、C 的具體實作
3. **執行檔層（Executable）**：最終的二進位程式

每個層次都可以檢驗**功能正確性**（如「具有帳號的使用者可以登入」）與**安全性質**（如「網路攻擊者無法取得金鑰」）。Tamarin 主要針對設計層的安全性質，近年也開始透過 Igloo 框架連接到語言實作層，甚至從二進位執行檔提取模型。

### 工作流程

1. 從協議規格（如 TLS 1.3 RFC）中提取各角色的**狀態機**
2. 將狀態轉換編碼為 Tamarin 的 **multiset rewriting rules**（規則定義觸發條件與狀態轉移）
3. 定義威脅模型：加入攻擊者規則（如攻擊者可取得長期金鑰或 session key 的 oracle rules）
4. 以一階邏輯公式描述安全性質（如 perfect forward secrecy：「攻擊者若知道 session key，必然是因為事先洩露了 I 或 R 的長期金鑰」）
5. Tamarin 內部將系統規則與性質否定轉化為**約束求解問題**，交由專用約束求解器處理，有三種結果：
   - **找到解 → 攻擊**：以 dependency graph（部分有序事件集合）呈現具體反例，顯示事件之間的依賴關係
   - **證明無解 → 安全性證明**：確認系統中不存在反例
   - **耗盡時間/記憶體**：此時可進入**互動模式（Interactive Mode）**，GUI 呈現已建構的部分證明樹，使用者可手動引導證明方向，或撰寫 **invariants** 放回檔案重新執行自動驗證

### 技術基礎與演進

- **操作語義**：基於等式理論的 multiset rewriting（CSF 2012、CAV 2012）
- **性質規約**：帶有時間點量化的一階邏輯片段，可直接對應 game-based 安全定義與 freshness conditions
- 持續 15 年以上的演算法改進（S&P 2014、CCS 2015、POST 2017、CSF 2018、JCS 2022、CSF 2023、CCS 2025），逐步擴展到 XOR、subterms、observational equivalence 等
- **密碼學原語的精確建模**（CCS 2019、CSF 2019、USENIX 2023、CCS 2024）：不再假設「完美密碼學」，而是針對每種密碼原語建立更貼近密碼學安全定義的模型，因此能發現更多微妙攻擊
- 符號分析的研究反向影響了密碼學定義本身，例如 buffing signatures 和 KEM binding properties 的工作

### 符號方法 vs 計算方法

Cremers 指出一個常見誤解：符號方法抽象掉了位元大小與機率，看似不如計算方法精確。但在實務上，計算方法受限於可擴展性，通常只能驗證系統的一小部分；而符號方法雖然抽象，卻能分析完整系統的所有互動，反而在實務中更能捕捉到跨協議攻擊等問題。

### Tamarin 能發現的攻擊類型

這些並非硬編碼的攻擊模式，而是透過系統化建模自然發現的：

- 大規模攻擊場景（如 TLS 1.3 Rev 10 的 18 條訊息流）
- 跨協議攻擊（Cross-protocol attacks）
- 非預期狀態機轉換
- 降級攻擊（Downgrade attacks）
- Nonce 重用攻擊（如 WiFi AES-GCM）
- 無效曲線點攻擊（Invalid Curve Points）
- 小階群攻擊（Small order points）
- DSKS 攻擊（Duplicate Signature Key Selection）
- 長度延伸攻擊（Length extension attacks）
- 惡意生成金鑰攻擊
- 後量子安全協議與混合方案分析

### 成功案例

- **EMV（Chip and Pin）**：用 Tamarin 發現 Visa 非接觸式支付協議漏洞
- **SPDM（PCI Express）**：驗證硬體安全協議
- **iMessage PQ3**：Apple 發布前委託兩項獨立分析——Douglas Stebila 的計算安全分析，以及使用 Tamarin 的形式化分析，以提高設計信心
- 越來越多業界公司主動使用 Tamarin 來**強化**協議設計，而非僅被動等待攻擊被發現

### 未來展望

Cremers 指出四個趨勢正在改變攻擊面，並引用了同場會議最後一個 session（Signal 攻擊）作為實例：

1. **記憶體安全語言普及**（Rust 等）：減少記憶體相關漏洞
2. **密碼學標準化改善**：減少密碼學實作錯誤
3. **LLM 驅動的程式分析**：將使更多人有能力發現跨協議互動中的攻擊
4. **攻擊面轉向程式邏輯**：上述三點使得協議邏輯與系統互動成為主要攻擊面

結論：像 Tamarin 這樣的形式化分析工具，對於強化協議安全或發現攻擊將越來越關鍵。

### 工具現況

- **Version 1.12** 於 RWC 2026 會議期間發布
- 開源、文件完善、可立即使用
- 活躍的使用者社群與教學材料（含免費 PDF 書籍）
- 提供編輯器外掛（如 VS Code）
- 正在籌備 **Tamarin Foundation**

## 與 TWCA 臺灣網路認證的關聯

Tamarin 是驗證安全協議設計正確性的形式化工具，其分析對象涵蓋 TLS、EMV 等 TWCA 業務所依賴的基礎協議。然而 Tamarin 本身是學術研究工具，與 TWCA 的 CA 營運、憑證簽發等日常業務無直接關係。TWCA 若需對其採用的協議進行形式化安全驗證，Tamarin 可作為評估工具之一。
