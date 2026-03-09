# Migrating a Silicon Root of Trust to Post-Quantum Crypto

## Session

- **Session:** Hardware
- **Conference:** RWC 2026

## Speaker(s)

- **Jade Philipoom** — zeroRISC
- **Hoang Nguyen Hien Pham** — PhD Student, Max Planck Institute for Security and Privacy (MPI-SP)

## Authors

Amin Abdulrahman, Evan Apinis, Andrew 'bunnie' Huang, Matthias J. Kannwischer, Ruben Niederhagen, Felix Oberhansl, Hoang Nguyen Hien Pham, Jade Philipoom, Dominic Rizzo, Robert Schilling, Peter Schwabe, Tobias Stelzer, Augustine Tang, Andreas Zankl

**機構：** Fraunhofer AISEC, zeroRISC, CHELPIS, Academia Sinica, Rivos, MPI-SP

## Abstract

本演講講述學術界與產業界聯手，將後量子密碼學（PQC）導入開源矽晶設計 OpenTitan 的多年合作歷程。從 SPHINCS+ 安全啟動開始，逐步發展出針對 ML-KEM 和 ML-DSA 的硬軟體協同設計，在嵌入式環境中實現高效的後量子密碼運算，並分享合作過程中的經驗教訓。

## Summary

### 背景：從經典密碼到後量子密碼的航程

演講以奇幻冒險地圖為主題，將 PQC 遷移比喻為一場跨越「經典密碼學之海」通往「後量子國度」的航程。這是一個跨越多年、多國、多學科，橫跨學術界與產業界的大型合作項目。

### Part 1：SPHINCS+ 安全啟動

#### 起源故事

2022 年 CHES 會議上，有人建議 Peter Schwabe 看看開源矽晶設計。Jade 在 OpenTitan 硬體上成功驗證了第一個後量子簽章後，立即傳訊給 OpenTitan 專案創辦人 Dom Rizzo。Dom 的回應是：「我們六個月後 ROM 凍結，你覺得我們能有後量子安全嗎？」

#### 成果

六個月後，SPHINCS+（即 SLH-DSA）安全啟動成功實現：

- 比未修改的參考實作**快 3 倍**
- shake-128s-simple：**< 13 ms** @100MHz
- sha3-128s-simple：**~ 9.3 ms** @100MHz

#### 與經典演算法的比較

- 比 RSA-3072 **慢 6 倍**
- 比 ECDSA-P256 **慢 3 倍**
- 透過 OTP（One-Time Programmable）記憶體設定啟用

### Part 2：ML-KEM 與 ML-DSA 的硬軟體協同設計

#### Amin 的碩士論文：Dilithium on OpenTitan（2023）

提出針對 OpenTitan 大數加速器（OTBN）的向量指令集：

| 指令 | 功能 |
|------|------|
| `bn.addv(m)` | 向量（模）加法 |
| `bn.subv(m)` | 向量（模）減法 |
| `bn.mulv(m)` | 向量（模）乘法 → 需要硬體乘法器 |
| `bn.shv` | 向量位移 |
| `bn.trn` | 向量轉置（原名 `bn.trans`） |

另外提出 SHA3/SHAKE 與 OTBN 的硬體連接、軟體實作 Round-3 Dilithium，以及將 IMEM 從 8 擴展到 32KB、DMEM 從 4 擴展到 128KB。

#### IEEE S&P 2025 論文成果

團隊（Andreas, Peter, Felix, Tobias, Hien, Jade, Amin）共同完成：

- 向量指令的硬體實作
- 從 Round-3 Dilithium 更新為 **ML-DSA 和 ML-KEM**
- 新的 SHA3/SHAKE 硬體連接

**目標頻率效能**：

| 運算 | 延遲 |
|------|------|
| ML-KEM-768 decapsulation | **0.8 ms** @100MHz |
| ML-DSA-65 signing (median) | **7.0 ms** @100MHz |

- 比無向量指令的 OTBN 快 **6–9 倍**
- 大多數運算**比非 PQC 替代方案更快**
- 電路面積增加：OTBN < 17%，OpenTitan Top Earlgrey < 3%（不含記憶體）

#### 進一步優化（CHES 2026）

Ruben 和 Hien 加入後，將模乘法移回軟體端：

- `bn.mulv(m)` → `bn.mulv{.lo,.hi}{.even,.odd}` → 三個版本
- Montgomery 模乘法：32-bit 從 12 降至 **7 cycles**，16-bit 從 12 降至 **5 cycles**
- 非模乘法：32-bit 從 4 降至 **2 cycles**，16-bit 從 4 降至 **1 cycle**
- 代價：延遲降低但程式碼變大
- 改良向量加法器設計（針對 ASIC/FPGA 平台優化）
- 在多種製程節點上進行 ASIC 合成驗證（Cadence 和 OpenROAD）

**結果**：cycle count 提升最高 **17%**，程式碼增加 **16%**，面積相近且最大頻率改善。

#### zeroRISC 精煉團隊（Dom, Kat, Evan, Jade）

產業端的優化工作：

- **Stack 優化 ML-DSA**：121kB → 11kB（**91% 減少**），ML-DSA-87 簽章代價為 80%–140% 速度降低
- **更快的 rejection sampling**：ML-KEM ~13% 加速，ML-DSA ~52% 加速
- **重新設計 SHA3/SHAKE 介面**：全部運算 ~6% 加速
- 整合 Design Verification 工具

#### 超越大數運算：ACC（非對稱密碼學協處理器）

OTBN + ML-DSA + ML-KEM = **ACC（Asymmetric Cryptography Coprocessor）**

最終效能：

| 運算 | 延遲 |
|------|------|
| ML-KEM-768 decapsulation | **0.7 ms** @100MHz |
| ML-DSA-65 signing (median) | **6.5 ms** @100MHz |

記憶體：32KB IMEM + 32KB DMEM

#### 彈性整合選項

| 架構 | 特點 |
|------|------|
| Ibex + ACC | ML-KEM/ML-DSA 在 ACC 上高速執行 |
| Ibex only | 使用 mlkem-native/mldsa-native，面積小但慢 ~10x（ML-KEM/ML-DSA）、~20x（RSA） |

Matthias Kannwischer（同場 Verification and Testing session 的 mlkem-native 講者）負責 Ibex-only 方案。

### Part 3：進行中的工作與經驗教訓

#### 進行中的工作

- 準備產業與研究用途的 **tapeout**
- 為 **UOV/Mayo、FN-DSA** 設計指令集擴展（ISE）
- **Jasmin** 對 ACC 的支援（形式驗證）
- **SLOTHY** 自動洩漏緩解
- **旁通道攻擊**評估與防護
- ML-KEM 與 ML-DSA 的 **masking**（Hien 的實習計畫）——目前在文獻回顧階段，探索全軟體 masking（probing model）

#### 經驗教訓

1. **開源消除摩擦**——開放原始碼讓跨機構合作變得可能
2. **文件拯救生命**——官方文件常常只寫 "TODO"，實際上大量依賴直接聯繫原作者（Santiago）
3. **研究點燃火花，產業規模化**——學術研究提供原型，產業將其推向量產
4. **面對面會議至關重要**——每次實體聚會後都能觀察到多個成果產出，合作者從 2022 年的 3 人成長到 RWC 2026 期間的 23 人

### 合作時間線

- **2022**：CHES 會議，Jade、Peter、Santiago 三人開始
- **2023**：Amin 碩士論文完成 Dilithium on OpenTitan
- **2024–2025**：IEEE S&P 2025 論文，七人團隊
- **2025–2026**：CHES 2026 優化論文 + zeroRISC 產品化 + ACC 設計
- **RWC 2026**：第三次實體高峰會，23 位合作者

## 與 TWCA 臺灣網路認證的關聯

1. **PQC 遷移的嵌入式實務參考**：TWCA 的 HSM 與伺服器設備未來需要支援 PQC 演算法。本研究展示了在資源受限的嵌入式環境（OpenTitan）中實現 ML-KEM 和 ML-DSA 的完整路徑——從純軟體到硬軟體協同設計——提供了效能基準與設計取捨的第一手數據。

2. **SPHINCS+ / SLH-DSA 在安全啟動的可行性**：TWCA 若評估在憑證簽發基礎設施中加入 hash-based 簽章作為備援方案，本研究證實 SPHINCS+ 在 100MHz 嵌入式處理器上可達 9.3ms 驗證，雖比 RSA/ECDSA 慢但已在實用範圍內。

3. **臺灣機構的直接參與**：合作者包含 CHELPIS（臺灣區塊鏈安全公司）和中央研究院（Academia Sinica），代表臺灣密碼學社群已深度參與國際 PQC 硬體研究。TWCA 可考慮與這些在地機構建立合作關係。

4. **開源矽晶的信任基礎**：OpenTitan 作為開源 Silicon Root of Trust，其透明性與可審計性對 TWCA 評估信任基礎設施具有參考價值——相較於封閉設計的 HSM 晶片，開源設計允許獨立安全審計。
