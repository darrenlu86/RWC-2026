# Finding Bugs and Features Using Cryptographically-Informed Functional Testing

## Session

- **Session:** Verification and Testing
- **Conference:** RWC 2026

## Speaker(s)

- **Fernando Virdia** — University of Surrey

## Authors

- Giacomo Fenzi (EPFL)
- Jan Gilcher (ETH Zurich)
- Fernando Virdia (University of Surrey)

## Abstract

基於密碼學安全性質設計功能測試，對 NIST PQC 競賽的 KEM 與簽章方案實作進行大規模測試，發現軟體 bug 與非預期特性。

## Summary

### 核心想法

傳統測試依賴已知答案測試（Known Answer Tests, KATs），但 KATs 的本質是與現有參考實作比對。如果你是設計者，沒有參考實作可比，該怎麼測？

答案：利用密碼學安全性質本身來設計測試。例如 hash function 具有 collision resistance，可以據此設計功能性測試來壓力測試實作。

### 方法論

以 KEM 為例（IND-CCA2 安全性要求密文不可竄改）：

1. 生成有效金鑰對並 encapsulate
2. **逐位元修改密文**
3. 用修改後的密文 decapsulate
4. 驗證 decapsulate 出的金鑰是否與原始不同（不可竄改性）

實作上使用 **AFL++ fuzzer** 搭配自定義 mutator，結合 **liboqs** 測試多個版本的 NIST PQC 提交。Fuzzer 的好處是能優雅處理 crash 而不中斷整個測試迴圈。

### 發現的 Bug 與 Feature

- **密文對齊 bug**：Kyber（後來的 ML-KEM）某些版本中，修改密文末尾的 padding bits 仍能正確 decapsulate（位元組對齊問題）
- **早期候選方案 crash**：無效密文直接導致程式崩潰——這等於提供了密文有效性 oracle
- **可避免的簽章可竄改性**：某些簽章方案非 strongly unforgeable，因非唯一編碼或 C API 與簽章語法的落差
- **Fujisaki-Okamoto 變換的非直覺特性**：使用 implicit rejection 的 FO-KEM，修改 secret key 的 gamma 分量後，對有效密文仍能正確 decapsulate（因為正確路徑不依賴 gamma）。這不是安全漏洞，但違反直覺。

### 實驗規模

在兩台多核機器上平行測試多個版本的 liboqs，包含所有 KEM 和簽章方案。大多數測試在半小時內完成，少數大 domain 的方案需要更長時間。

### 結論

這是一個**低成本、高報酬**的測試方法——利用安全性質寫 fuzzer harness，適用於任何密碼學實作。講者鼓勵所有人在自己的實作中嘗試。

## 與 TWCA 臺灣網路認證的關聯

本場演講提出的密碼學導向功能測試方法，對 TWCA 有直接的實務參考價值：

1. **密碼學實作驗證**：TWCA 的產品涉及多種密碼學實作（TLS、數位簽章、雜湊等），本方法可作為內部品質保證的補充——不只跑 KATs，還能基於安全性質設計 fuzzer 來壓力測試實作。

2. **PQC 遷移準備**：隨著後量子密碼學標準化，TWCA 未來需要整合 ML-KEM 等新方案。本研究揭示的 liboqs 實作 bug 是直接的前車之鑑，測試方法也可直接套用。
