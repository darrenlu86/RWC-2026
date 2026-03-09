# Real-Time FHE Applications and the Era of Encrypted AI

## Session

- **Session:** Invited Talk
- **Conference:** RWC 2026

## Speaker(s)

- **Jung Hee Cheon** — Professor, Mathematical Sciences, Seoul National University; CEO, CryptoLab Inc.

## Abstract

自 2011 年首次實作以來，全同態加密（FHE）經歷了快速成長，即使不使用客製化硬體，處理速度每年提升約 7 倍。許多應用場景已達到即時可行的階段。本演講介紹 FHE 運算與 bootstrapping 在 CPU/GPU 平台上的最新效能基準，以及加密統計與機器學習等應用。特別介紹近期提出的同態矩陣運算及其在加密 RAG 與搜尋中的即時應用。最後探討加密大型語言模型（Encrypted LLM）的實作趨勢與實務應用。

## Summary

### Part 1：Real-Time FHE

#### FHE 簡介

- **資料保護的三個面向**：靜態資料（Symmetric Crypto）、傳輸中資料（Public Key Crypto）、使用中資料（Homomorphic Encryption）
- 1978 年首次提出概念，被譽為「密碼學的聖杯」（Holy Grail of Cryptography）
- 2009 年 Gentry 提出首個完全同態加密方案
- 2021–2026 年進行 ISO 標準化
- 基於 Lattice 問題，具備**抗量子**安全性

#### FHE 方案的四代演進

| 世代 | 特點 | 代表方案 |
|------|------|----------|
| 1st | 首個可行的 FHE 構造 | Gen09, DGHV10 |
| 2nd | 更高效的 depth-linear 構造 | BGV11, LTV12, BFV12, BLLN13 |
| 3rd | 更慢的噪音成長 + 快速 bootstrapping | GSW13, DM14 (FHEW), CGGI16 (TFHE) |
| 4th | 支援近似計算與捨入數 | CKKS |

#### CKKS 效能突破

- Bootstrapping 時間：1,800 秒（2011）→ **9.62 毫秒**（2026, RTX 5090）
- 13 年間加速超過 **220 億倍**（約每年 7 倍）
- 一次 bootstrapping 可獲得 **15 層**乘法深度（19-bit 精度）
- 意味著 1 秒內可評估深度 1,500 的電路

#### 4.5 代 CKKS：矩陣乘法加速

關鍵突破：**Plaintext-Ciphertext Matrix Multiplication**——不需要使用昂貴的 FHE 乘法或旋轉（rotation），矩陣乘法開銷從 **1,000x 降至 ~2x**。

效能數據（512×512 × 512×4096 矩陣）：
- CPU 單線程：884.3 ms
- GPU (RTX 4090)：**1.52 ms**

#### 加密向量相似度搜尋

傳統做法（AES-256）：先解密再明文搜尋，解密速度 200 MB/s
4.5 代 CKKS：直接在加密向量上搜尋，運算速度 **2 GB/s**（比 AES 解密後搜尋更快）

#### HEaaN2 Library 效能（2026 年 3 月）

| 運算 | Open Source (CPU) | HEaaN2 (CPU) | HEaaN2 (GPU, RTX 5090) |
|------|-------------------|---------------|------------------------|
| Addition | 3.85 ms | 0.09 ms | 0.01 ms |
| Multiplication | 121.4 ms | 10.69 ms | 0.11 ms |
| Bootstrap | 11,341 ms | 665 ms | **9.62 ms** |

### Part 2：Real-Time and Real-World FHE Applications

#### 已部署的應用場景

| 應用 | 客戶/場景 | 保護對象 | 延遲 |
|------|----------|----------|------|
| 加密人臉辨識 | Fintech 刷臉支付 | 人臉模板 | 0.37s（100 萬模板，CPU） |
| 加密 RAG | 韓國國防部 | 軍事機密文件的向量與文字 | 0.2s（1 萬文件，CPU） |
| 加密語義搜尋 | 雲端供應商 E2EE 圖片搜尋 | 使用者儲存的圖片 | ~30ms（140 萬圖片，L40S GPU） |
| 加密物件偵測 | 軍方 | 模型權重（即使模型被竊取也安全） | 即時推論 |
| 加密資料分析 | 韓國統計局 | 跨機構機密資料 | — |

#### 加密人臉辨識架構

Camera → Query Vector → 加密模組 → Server（加密相似度搜尋 + 加密人臉資料庫）→ HSM 解密 → 結果

#### 加密 RAG

文件加密後儲存，LLM 查詢時在加密向量上執行相似度搜尋，僅搜尋結果（token）解密後送入 LLM。資料庫全程保持加密。另有美國虛擬心理諮詢聊天機器人的應用案例。

#### 加密資料分析與統計

政府部門（A, B）、公共機構（C）、地方政府（D, E）、私人機構（F）將資料加密後上傳至 Encrypted Cloud 的 Data Hub，可在不解密的情況下進行跨機構資料分析。韓國統計局已支持開發。

#### CryptoLab 產品

- **HEaaN2 Library**：C++ CKKS 同態加密函式庫，支援 GPU（CUDA），bootstrapping 9.62ms
- **enVector**：加密向量搜尋產品，基於 4.5 代 CKKS，Docker 部署或 SaaS

### Part 3：Encrypted LLM

#### 加密 LLM 實作

使用 HEaaN2 + 4.5 代 CKKS + 矩陣乘法，在 **Llama-3 8B** 模型上實現端對端加密 LLM 推論。

**關鍵挑戰與解法**：
- 先前 FHE LLM 僅限小模型或極短輸入（8–16 tokens）
- 大模型中的 **outlier 問題**：使用 token prepending 和 rotation 緩解
- **Two-stage prefill**：加密重要 token，其餘用明文處理，速度僅慢不到 2 倍
- 矩陣運算在最低層級（level 1–2）執行以降低成本
- 每層需 7 次 bootstrapping，32 層完成 prefill

**效能數據**：
- Prefill time：**16 秒**（8x RTX 5090 GPU）
- 支援更多 unencrypted tokens 時，TTFT 約 25 秒
- 新的 bootstrapping（<10ms）尚未應用，預期可進一步改善

**應用場景**：Private AI counseling、Secure enterprise intelligence、Confidential collaborative R&D

### 結論

- FHE 已達到**即時處理**的能力：1,500 深度電路可在 1 秒內用 GPU 評估
- 即時且真實世界的 FHE 應用正在擴散（人臉辨識、RAG、語義搜尋、物件偵測、統計分析）
- 加密 LLM 已成為現實

## 與 TWCA 臺灣網路認證的關聯

本場演講與 TWCA 的業務有多層關聯：

1. **加密資料處理的新範式**：TWCA 作為憑證中心持有大量敏感資料（憑證申請資料、私鑰操作記錄等）。FHE 提供了一種在不解密的情況下處理這些資料的可能性——例如在加密狀態下進行憑證狀態查詢、CRL/OCSP 查詢等，從根本上消除資料處理過程中的暴露風險。

2. **加密向量搜尋與 AI 服務**：TWCA 若未來提供 AI 輔助的資安服務（如加密日誌分析、異常偵測），FHE 的加密 RAG 和加密語義搜尋技術可確保客戶資料在分析過程中保持加密，解決雲端 AI 服務的信任問題。

3. **跨機構加密資料分析**：韓國統計局的案例與 TWCA 的情境相似——TWCA 可能需要與政府機關、金融機構進行跨機構的憑證使用統計或資安事件關聯分析。FHE 使多方可在不揭露各自原始資料的情況下進行聯合分析。

4. **Lattice-based 與 PQC 的共通基礎**：FHE 與 ML-KEM 等後量子密碼學方案共享 Lattice 問題作為安全基礎。TWCA 在 PQC 遷移過程中累積的 Lattice 密碼學知識，未來可延伸應用到 FHE 領域。
