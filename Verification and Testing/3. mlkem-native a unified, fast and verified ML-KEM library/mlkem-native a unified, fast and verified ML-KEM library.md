# mlkem-native: A Unified, Fast and Verified ML-KEM Library

## Session

- **Session:** Verification and Testing
- **Conference:** RWC 2026

## Speaker(s)

- **Hanno Becker** — Amazon Web Services

## Authors

- Hanno Becker (Amazon Web Services)
- Matthias J. Kannwischer (Chelpis)
- Dusan Kostic
- John Harrison
- Rod Chapman

## Abstract

mlkem-native 是一個 C90 + assembly 的 ML-KEM 實作，在 Post-Quantum Cryptography Alliance（Linux Foundation）治理下開發，結合高效能、形式化驗證與可維護性。

## Summary

### 為什麼是 C90

- **可移植性**：C90 可整合到幾乎所有函式庫和平台
- **記憶體安全**：透過形式化驗證彌補 C 語言缺乏的記憶體安全
- **案例研究**：ML-KEM 作為驗證方法的示範，未來可推廣到其他 PQC 演算法

### 模組化架構

- **Front-end**：所有高層邏輯（ML-KEM + Shake/SHA-3），純 C 實作，只需維護一份
- **Back-end**：效能關鍵函式（NTT、Keccak 等），提供 C 參考實作 + 各平台高度最佳化的組合語言（AArch64、AVX2、RISC-V、ARM MVE）
- **統一 API**：前後端之間的抽象層，方便插入新平台的最佳化程式碼

### 效能最佳化

使用 **Slothy** 組合語言超級最佳化器：維護乾淨可讀的組合語言原始碼，由 Slothy 自動生成高效版本。好處是不需要信任 Slothy（可能有 bug），因為驗證的是最終輸出的 object code。

### Constant-time 保護

- 組合語言層面：遵循規則（無秘密相依的記憶體存取、分支、變動時間指令）
- C 層面：使用 **value barriers**（inline assembly 或 portable 實作）阻止編譯器對機密值進行推理和最佳化
- 多種編譯器 + 多種 flags 組合測試

### 形式化驗證

**Assembly 端**：
- 使用 **HOL-Light** 互動式定理證明器 + **s2n-bignum** 驗證基礎設施
- 驗證**最終的 object file**（post-hoc verification），不需信任產生組合語言的工具鏈
- 目前涵蓋 memory safety，functional correctness 進行中

**C 端**：
- 使用 **CBMC**（C Bounded Model Checker）證明 memory safety 與 type safety
- Type safety 尤其重要：整數溢位是 ML-KEM 實作中最可能的隱藏 bug 來源（模組算術的 bounds tracking 錯誤）
- 透過 in-source annotations（contracts + loop invariants）實現，兼具可讀性與形式化文件功能

**CI 整合**：
- 所有 CBMC 和 HOL-Light 證明在 CI 中持續運行
- 提供 **Nix Flake** 確保完全可重現的驗證環境

### 風險透明化

發布 **Soundness.md** 文件，列出形式化驗證故事中所有已知風險與緩解措施（借鑑 Amazon 的形式化驗證最佳實務）。例如：CBMC 在 64-bit 系統上驗證，保證不直接轉移到 32-bit 實作。

## 與 TWCA 臺灣網路認證的關聯

本場演講與 TWCA 的 PQC 遷移策略直接相關：

1. **PQC 函式庫選擇**：mlkem-native 是目前形式化驗證最完整的 ML-KEM 實作之一，TWCA 在評估 PQC 函式庫時可優先考慮。其 C90 的可移植性也有利於整合到 TWCA 的 HSM 或伺服器環境。

2. **驗證方法論參考**：Soundness.md 的風險透明化做法值得 TWCA 在評估任何密碼學實作時借鑑——要求供應商明確列出驗證範圍與已知限制。

3. **Constant-time 驗證**：TWCA 的密碼學實作（特別是私鑰運算）需要 constant-time 保證。本講展示的 value barriers + 多編譯器測試 + 形式化驗證的組合策略，可作為 TWCA 內部安全評估的檢查清單。
