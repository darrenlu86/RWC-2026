# Compositional Formal Verification of SNARKs with ArkLib

## Session

- **Session:** Verification and Testing
- **Conference:** RWC 2026

## Speaker(s)

- **Quang Dao** — Carnegie Mellon University

## Authors

- Quang Dao (Carnegie Mellon University)
- Alexander Hicks (Ethereum Foundation)
- Devon Tuma (University of Minnesota & Succinct)
- Julian Sutherland (Nethermind)
- Katerina Hristova (Imperial College London)
- Frantisek Silvasi (Independent)
- Ilia Vlasov
- Chung Thai Nguyen

## Abstract

ArkLib 是一個在 Lean 定理證明器中形式化指定與驗證 SNARK 協議安全性的函式庫，以 Interactive Oracle Reductions (IORs) 為核心框架，實現模組化的可組合驗證。

## Summary

### 背景：SNARK 的重要性與風險

**SNARK**（Succinct Non-interactive Arguments of Knowledge）是簡短且可高效驗證的證明，讓不受信任的一方能證明某個計算被正確執行。應用包括：

- 身份驗證
- 區塊鏈擴展（scaling）
- 正確 AI 推論的證明

目前 SNARK 在以太坊生態系中保護超過 **10 億美元**的資產。未來以太坊計劃將 SNARK 整合到共識客戶端中，意味著每個以太坊節點都將運行 SNARK 驗證器——SNARK 正在達到如 TLS、MLS 等協議的關鍵基礎設施等級。

### 現有 SNARK 的問題

1. **Bug 極為常見且後果災難性**：可導致區塊鏈上不可偵測的無限印鈔
2. **實作與規格脫節**：大多數實作沒有基於任何規格，程式碼本身就是「規格」；即使基於學術論文，也在不斷最佳化後快速偏離
3. **學術文獻分散且不完整**：教科書簡化處理，不涵蓋實作驗證所需的所有細節，且更新緩慢

### ArkLib 的設計理念

**三大設計目標**：

1. **規格與實作緊密耦合**：SNARK 演進快速（每隔幾個月就有新最佳化論文），規格必須持續更新以匹配實作——這與 TLS 等長期穩定的協議不同
2. **統一且模組化的定義與定理**：單體安全證明極其脆弱且難以維護。需要從基本建構塊出發，透過既有的可組合模式逐步建構
3. **高保證基礎**：選擇具有最小且簡單核心的定理證明器，不過度依賴外部求解器

### 為何選擇 Lean

- **語言設計現代化**，開發活躍
- 擁有 **MathLib**（最大的形式化數學庫），SNARK 所需的數學基礎皆可取用
- **AI 輔助證明**的最佳平台

### SNARK 的模組化結構

現代 SNARK 遵循一個共通的構造模式：

1. **Non-interactive proof** → 內部核心是 **Interactive Argument**（Prover 與 Verifier 互動多輪，Verifier 只發送隨機挑戰）
2. **Fiat-Shamir Transform**：將互動協議轉為非互動，透過 hash Prover 的訊息來生成挑戰
3. **Interactive Oracle Proofs (IOPs)**：更抽象的資訊理論安全協議，Verifier 不完整讀取 Prover 的訊息，而是透過 query 介面查詢特定位置或多項式求值
4. **Interactive Oracle Reductions (IORs)**：IOP 可進一步分解為更小的 IOR，每個 IOR 將複雜 relation 約化為較簡單的 relation

### IOR 的關鍵性質

- **可組合性**：可迭代地約化複雜 relation 直到極簡
- **Virtualization**：子協議可能有不同的 statement/witness 格式，透過虛擬化對齊介面
- **安全性質**：Completeness、Knowledge Soundness、Zero Knowledge

### SumCheck 協議的 Lean 形式化

以 SumCheck 協議為例展示 ArkLib 的 Lean 實作：

- **Protocol Format**：定義為輪次列表，每輪有方向（Prover/Verifier）和訊息型態
- **Prover 規格**：Co-inductive 定義，逐層剝離協議結構
- **Verifier 規格**：接收所有訊息，執行檢查，返回下一個 target 或失敗
- **Soundness 定理**：形式化定義並證明 SumCheck 的 soundness error 為 d/|F|（d 為度數，|F| 為域大小）

整個規格在適當的 Lean notation 下極為簡潔。

### AI 加速形式化驗證

前沿 LLM（Claude、GPT、Gemini）以及專門的證明器在 Lean 上的能力已令人驚豔：

- AI 證明速度已超過人類，metaprogramming 能力更強
- 每個 PR review 都整合 AI，已捕捉到多個 **misformalization**（形式化錯誤）
- 未完成的定理提交給 AI 服務 API 自動填充證明——目前約有 10 個 AI 生成的證明待合併
- AI 生成的證明有時也能發現 misformalization（因為無法為錯誤陳述找到證明）

### 專案現狀與未來

- 正在形式化多個 IOR 協議（包括 SumCheck、FRI 等）
- 建設背景理論：coding theory、Fiat-Shamir transform
- 開發基礎子函式庫：oracle computations、program logic、computable polynomials
- 目標：與驗證約束（verified constraints）和驗證實作（verified implementations）整合，橋接協議層與其他堆疊層

## 與 TWCA 臺灣網路認證的關聯

本場演講與 TWCA 的業務有以下間接但有意義的關聯：

1. **密碼學協議形式化驗證的趨勢**：SNARK 社群正在以 Lean 定理證明器為密碼學協議建立可機械驗證的安全證明。這代表密碼學領域的驗證標準正在提升。TWCA 依賴的 TLS、數位簽章等協議，未來也可能被要求具備類似等級的形式化安全保證。ArkLib 的可組合驗證框架展示了此方向的可行路徑。

2. **AI 輔助形式化驗證的實用性**：ArkLib 團隊展示 AI 已能有效加速 Lean 中的形式化證明工作，並能捕捉 misformalization。這暗示未來密碼學實作的形式化驗證成本可能大幅降低，TWCA 可關注此趨勢，評估是否在自有密碼學模組中引入類似的驗證流程。

3. **Fiat-Shamir Transform 的正確性**：Q&A 中提到 Fiat-Shamir 實例化錯誤極為常見（instance 未被 hash 等問題）。TWCA 的任何使用 Fiat-Shamir 相關技術的協議（如某些數位簽章方案內部），都應注意此類微妙錯誤。
