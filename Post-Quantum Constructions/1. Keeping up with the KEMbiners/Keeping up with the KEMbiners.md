# Keeping up with the KEMbiners

## Session

- **Session:** Post-Quantum Constructions
- **Chair:** Douglas Stebila
- **Conference:** RWC 2026

## Speaker(s)

- **Deirdre Connolly and Kathrin Hövelmanns** — Eindhoven University of Technology

## Authors

- Kathrin Hövelmanns (Eindhoven University of Technology)
- Deirdre Connolly (Selkie Cryptography)
- Andreas Hülsing (Federal Office for Information Security Germany)
- Stavros Kousidis (SandboxAQ)
- Matthias Meijers

## Abstract

探討後量子遷移中 KEM 組合器（KEM combiner）的通用構造。證明所有基於 Fujisaki-Okamoto 變體的 KEM（包括 ML-KEM、HQC、FrodoKEM、Classic McEliece、NTRU 等）均滿足密文第二原像抗性（C2PRI），進而提出比 X-Wing (QSF) 更高效的新組合器 QSI（Quantum Superiority Interceptor），僅需將兩個 KEM 的共享秘密與標籤串接後輸入 KDF，即可在標準模型下證明 IND-CCA 安全性。

## Summary

### 背景：Harvest Now, Decrypt Later 威脅

後量子遷移的核心動機在於「先收割、後解密」（Harvest Now, Decrypt Later）的威脅模型：攻擊者記錄當前傳輸中的加密資料，待量子電腦或密碼分析突破後進行解密。金鑰交換（key exchange）是此場景中的主要故障點，因此後量子遷移的第一階段著重於對稱金鑰的建立方式。

### KEM 基礎與 IND-CCA 安全性

**金鑰封裝機制（KEM）** 是一種公鑰方法，用於建立對稱金鑰。Bob 產生公鑰對，Alice 呼叫 Encaps 演算法產生金鑰與密文，Bob 使用 Decaps 從密文中還原金鑰。KEM 廣泛用於 TLS 等需要即時建立對稱金鑰的協議中。

安全性要求分為兩層：
1. **IND-CPA**：攻擊者看到密文後無法區分對應金鑰與隨機值
2. **IND-CCA**：即使攻擊者擁有解封裝預言機存取權，金鑰仍不可區分於隨機值——即解封裝操作不應成為探測秘密金鑰材料的途徑

### Fujisaki-Okamoto 變體的多樣性

後量子 KEM 幾乎都使用 Fujisaki-Okamoto (FO) 轉換，將弱安全加密方案轉為具 IND-CCA 安全性的 KEM。但實務上各 KEM 在 FO 細節上有差異：
- 有些將密文雜湊進最終會話金鑰，有些不會
- 有些附加第二段密文作為校驗和（金鑰確認）
- 有些使用 salt 以緩解多目標攻擊
- 有些在推導最終金鑰時包含公鑰材料以建立多使用者安全性
- 有些進行迭代雜湊或預雜湊

### 混合 KEM 組合器的演進

**GHP 組合器（Kitchen Sink Combiner）**：最保守的方法，將兩個 KEM 的共享秘密與密文全部納入 KDF 計算。安全性僅要求至少一個組件 KEM 維持 IND-CCA。但由於後量子 KEM 密文很大（如 ML-KEM 768 的密文至少 1.5 KB），需要大量雜湊運算，在 TLS 1.3 等需頻繁臨時金鑰交換的場景中效率不佳。

**QSF（Quantum Superiority Fighter / X-Wing）**：X-Wing 論文提出的通用構造，使用一個後量子 KEM 加上直接依賴橢圓曲線群的 Strong Diffie-Hellman 假設。關鍵創新是引入「密文第二原像抗性」（Ciphertext Second Pre-image Resistance, C2PRI）概念——給定共享秘密、密文與金鑰對，攻擊者無法找到另一個密文解封裝為相同共享秘密。由於 C2PRI 成立，可以省略大型後量子 KEM 密文的雜湊，顯著提升效率。

### 本研究的核心貢獻：QSI 組合器

**QSI（Quantum Superiority Interceptor）**：本研究提出的新組合器，進一步簡化結構——僅將兩個 KEM 的共享秘密加上標籤輸入 KDF，完全省略所有密文的雜湊。

安全性要求：
- 至少一個組件 KEM 維持 IND-CCA 安全
- 另一個組件 KEM 維持 C2PRI
- 反之亦然（雙向保護）

證明在**標準模型**下完成（使用 split-key PRF 假設），不需依賴隨機預言機模型或量子隨機預言機模型。

### C2PRI 的廣泛適用性

研究證明 C2PRI 適用於所有已知的 FO 轉換變體，包括：
- **顯式拒絕（explicit rejection）** 與 **隱式拒絕（implicit rejection，ML-KEM 使用）**
- **含 salt 變體** 與 **含金鑰確認變體**
- 所有具體實例：ML-KEM、HQC、FrodoKEM、Classic McEliece、NTRU 等

### PQ-PQ 混合組合

QSI 的另一個吸引力在於支持高效的 **PQ-PQ 混合**。當需要結合兩個基於不同困難假設的後量子 KEM（例如結構化格、編碼基、非結構化格）時，QSI 提供最高效的方式：僅串接共享秘密、加入標籤、通過 KDF。

### 多目標安全性與綁定性質

QSI 的多目標安全性（multi-user, multi-challenge security）直接繼承自其組件 KEM 的性質，不需要額外的組合器層級分析，這是相較 QSF 的另一個優勢。

## 與 TWCA 臺灣網路認證的關聯

1. **TLS 後量子遷移策略**：TWCA 作為 SSL/TLS 憑證簽發機構，需密切關注 TLS 1.3 中的混合金鑰交換標準化進程。QSI 組合器比 GHP 更高效、比 QSF 更通用，其在 IETF CFRG 的標準化工作直接影響 TWCA 未來支援的後量子 TLS 密碼套件選擇。

2. **KEM 演算法多樣性支援**：QSI 對所有 FO 變體 KEM 的 C2PRI 證明，意味著 TWCA 在評估後量子方案時不必侷限於單一演算法（如 ML-KEM），可以靈活選擇 HQC、Classic McEliece 等替代方案，並以 QSI 進行高效混合組合。

3. **PQ-PQ 混合的保守策略**：對於高安全等級的 PKI 應用，TWCA 可考慮採用 PQ-PQ 混合方案（例如同時使用格基與編碼基 KEM），QSI 提供了最高效的實現路徑，特別適用於 TWCA 需要長期保護的根憑證和中介憑證場景。

4. **標準模型安全證明的優勢**：QSI 在標準模型下的安全證明（不依賴隨機預言機）為 TWCA 的合規審查提供更強的理論保障，符合金融監管機構對密碼學方案嚴謹性的要求。
