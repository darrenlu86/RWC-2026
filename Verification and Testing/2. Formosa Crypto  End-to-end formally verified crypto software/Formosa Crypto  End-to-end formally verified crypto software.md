# Formosa Crypto: End-to-end Formally Verified Crypto Software

## Session

- **Session:** Verification and Testing
- **Conference:** RWC 2026

## Speaker(s)

- **Santiago Arranz-Olmos** — Max Planck Institute for Security and Privacy
- **Ignacio Cuevas** — Max Planck Institute for Security and Privacy

## Authors

- José Bacelar Almeida, Gustavo Xavier Delerue Marinho Alves, Santiago Arranz-Olmos, Manuel Barbosa, Francisca Barros, Gilles Barthe, Lionel Blatter, Chitchanok Chuengsatiansup, Ignacio Cuevas, François Dupressoir, Luís Esquível, Andreas Hülsing, Benjamin Grégoire, Ruben Gonzalez, Jan Jancar, Vincent Hwang, Vincent Laporte, Jean-Christophe Léchenet, Ting-han Lim, Cameron Low, Tiago Oliveira, Hugo Pacheco, Swarn Priya, Miguel Quaresma, Rolfe Schmidt, Peter Schwabe, Antoine Séré, Basavesh Ammanaghatta Shivakumar, Pierre-Yves Strub, Lucas Tabary-Maujean, Yuval Yarom, Zhiyuan Zhang, Jieyu Zheng

## Abstract

Formosa 專案提供端對端形式化驗證的密碼學軟體方法論與工具鏈，從密碼學規格到組合語言層級的安全保證。Signal 已將其 ML-KEM 實作整合至生產環境。

## Summary

### Formosa 是什麼

**Formosa**（Formally Verified Open Source Amazing Crypto）是一個跨學術界與業界的專案，目標是為密碼學軟體提供**端對端的形式化安全保證**——從抽象密碼學方案，經過實作，一路到最終執行的組合語言。

附帶一提，Formosa 也是臺灣的舊稱（福爾摩沙）。

### 三個抽象層級

1. **密碼學方案（Scheme）**：抽象數學物件，關注正確性（解密後得到原文）與安全性（IND-CCA）
2. **實作（Implementation）**：Rust/C 等語言的程式碼，需符合規格並避免記憶體錯誤、未定義行為
3. **執行碼（Executable）**：最終運行的組合語言，需確保效率並避免旁路攻擊

### 工具鏈

- **Jasmin**：專為密碼學設計的程式語言，編譯到組合語言
- **EasyCrypt**：形式化安全證明工具，用於證明方案的 IND-CCA 安全性與功能正確性
- **Jasmin 編譯器**（已驗證）：保證 Jasmin 程式碼的性質延伸到生成的組合語言
- 安全性與 constant-time 的自動檢查器（基於型別系統）
- Zeroization：編譯器可自動清除暫存記憶體，不留下機密痕跡

### Signal 的採用

Signal 已在生產環境中採用 Formosa：

- **伺服器端 Noise Protocol**：遷移到使用 Formosa 的 ML-KEM 實作
- **Contact Discovery Service**：大幅改寫，使用 Jasmin 重新實作基於 Oblivious RAM 的模組（pathORAM、oblivious hash table、oblivious sort 等）

### 發現 C 編譯器引入的旁路漏洞

在 Signal 的 Contact Discovery Service 中，原始 C 實作在源碼層面是 constant-time 的，但 **C 編譯器的最佳化**引入了 data-dependent branch，破壞了 constant-time 保證。這種漏洞極難預防，因為編譯器持續演進，可能隨時重新引入。Jasmin 語言的設計就是為了從根本上避免這類問題。

### ML-KEM 驗證狀態

- 密碼學方案：EasyCrypt 中完整形式化，已證明 IND-CCA 安全
- 功能正確性：完成（除了新版 SHA-3 實作尚在驗證中）
- 安全性（memory safe、type safe）：以 EasyCrypt 證明
- Constant-time + Spectre P1 保護：型別系統驗證（assembly 層的保存證明進行中）
- 編譯器正確性：已在 Coq 中證明
- EasyCrypt 模型提取：仍為 trusted（但設計上使提取近乎 trivial）

## 與 TWCA 臺灣網路認證的關聯

本場演講與 TWCA 的業務有多層關聯：

1. **密碼學實作的最高標準**：TWCA 的核心業務依賴密碼學實作的正確性（TLS、數位簽章）。Formosa 展示的端對端驗證方法論——從數學證明到組合語言——代表了密碼學軟體品質保證的最高標準，可作為 TWCA 評估供應商或自有實作的參考基準。

2. **編譯器引入旁路漏洞的警示**：C 編譯器最佳化破壞 constant-time 的案例，直接影響所有用 C 實作密碼學的軟體，包括 TWCA 可能使用的 HSM 韌體或伺服器端密碼學模組。

3. **PQC 遷移的驗證標準**：Signal 選擇經 Formosa 驗證的 ML-KEM 實作，為 PQC 遷移設立了驗證標準。TWCA 在規劃 PQC 遷移時，可參考此標準來選擇或驗證 ML-KEM 實作。
