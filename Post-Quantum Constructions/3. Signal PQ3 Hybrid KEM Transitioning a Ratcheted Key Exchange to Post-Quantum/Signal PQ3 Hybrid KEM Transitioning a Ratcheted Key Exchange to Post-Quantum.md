# Signal PQ3 Hybrid KEM: Transitioning a Ratcheted Key Exchange to Post-Quantum

## Session

- **Session:** Post-Quantum Constructions
- **Chair:** Douglas Stebila
- **Conference:** RWC 2026

## Speaker(s)

- **Rune Fiedler** — CISPA Helmholtz Center for Information Security

## Authors

- Rune Fiedler (CISPA Helmholtz Center for Information Security)
- Robin Leander Schröder (Signal Messenger)
- Rolfe Schmidt
- Karthikeyan Bhargavan
- Cas Cremers

## Abstract

提出 XHMQV 協議作為 Signal 初始交握的改進方案，結合 HMQV 風格的金鑰推導公式與半靜態金鑰支援，將 Diffie-Hellman 組合數從 4 個增加到 6 個，但群指數運算僅需 5 次（低於 X3DH 的 8 次），同時獲得更強的最大曝露安全性（maximal exposure security）與等效的可否認性（deniability），並可在後量子認證遷移時一併部署。

## Summary

### Signal 協議的初始交握

Signal 協議被 Signal Messenger、WhatsApp、Google Messages、Facebook Messenger 等廣泛使用。初始交握的流程為：
1. **Bob 上傳預金鑰包（prekey bundle）** 至金鑰伺服器
2. Bob 可離線
3. **Alice 查詢**金鑰伺服器取得 Bob 的公鑰材料
4. Alice 計算共享會話金鑰
5. Alice 將自己的金鑰材料傳送給 Bob，Bob 同樣計算會話金鑰

此機制提供**非同步性**（Bob 不需在線）與**雙向認證**。交握完成後，雙方使用 Double Ratchet 協議持續輪換金鑰。

### X3DH 基線協議

**X3DH（Extended Triple Diffie-Hellman）** 是目前的基線協議。Alice 和 Bob 各持有：
- **長期金鑰**：永久保留
- **臨時金鑰**：僅使用一次
- Bob 額外有**半靜態金鑰**：約每月輪換一次

Bob 的長期金鑰同時用作簽署金鑰，簽署半靜態金鑰。Bob 的預金鑰包包含三個公鑰加上簽章。

Alice 計算四個 Diffie-Hellman 組合：
1. Alice 長期 × Bob 半靜態
2. Alice 臨時 × Bob 長期
3. Alice 臨時 × Bob 半靜態
4. Alice 臨時 × Bob 臨時

四個共享秘密經 KDF 推導會話金鑰。

**臨時金鑰耗盡問題**：若 Bob 上傳 100 個預金鑰包，第 101 人連線時無臨時金鑰可用，退回「精簡模式」（少一個 DH 組合）。

### 最大曝露安全性的弱點

考慮 Alice 臨時金鑰被洩露的場景：攻擊者可計算四個 DH 組合中的三個，僅剩長期-半靜態組合提供保護。若攻擊者進一步洩露 Bob 的半靜態金鑰，則所有四個組合均被攻破。

**解決方案**：增加一個**長期-長期 DH 組合**，使攻擊者需同時洩露雙方長期金鑰才能攻破。但這意味著需要 5 個 DH 組合（比 4 個更多），似乎降低效率。

### HMQV 的啟發

Hugo Krawczyk 於 Crypto 2005 提出的 **HMQV 協議**提供了效率解法。核心思想是不直接串接 DH 共享秘密，而是使用公式：

將臨時金鑰輸入雜湊函數產生隨機化因子 D 和 E，再將其與對應長期金鑰組合。如此所有四個可能的 DH 組合都被包含，但僅需**兩次群指數運算**。

HMQV 的缺點：不支援臨時金鑰耗盡的備援機制。

### XHMQV：本研究的核心提案

**XHMQV** 在 HMQV 基礎上增加 Bob 的半靜態金鑰，結合另一個隨機化因子。改進後：
- 包含所有**六個可能的 DH 組合**（長期-長期、長期-半靜態、長期-臨時、臨時-長期、臨時-半靜態、臨時-臨時）
- 在金鑰推導中加入上下文資訊以防止非金鑰共享攻擊

**效能比較**：

| 協議 | 群指數運算 | 安全組合數 | 金鑰重用建模 |
|------|-----------|-----------|-------------|
| X3DH | 8 次 | 1/4 存活即安全 | 否 |
| XHMQV | 5 次 | 1/6 存活即安全 | 是 |
| MU-DH | 5 次 | 1/4 存活即安全 | 否 |

XHMQV 是唯一準確建模金鑰重用問題（Bob 的長期金鑰同時用於 DH 和簽章）的安全分析。

### 可否認性分析

**半誠實對手模型**（所有參與者遵循協議）：X3DH 和 XHMQV 均達成可否認性。這是 Signal 所尋求的類型。

**惡意對手模型**（Bob 可使用他人公鑰或故意不知自己的秘密金鑰）：X3DH 和 XHMQV 均無法達成。但目前沒有已知協議能在此模型下達成可否認性。

### 後量子遷移路徑

XHMQV 提供兩條後量子遷移路徑：
1. **在 XHMQV 上疊加 KEM**（類似 PQXDH 在 X3DH 上疊加 KEM）
2. **與純後量子方案混合**（如 Ring-XKEM，由 Shuichi Katsumata 提出）

**部署窗口**：當 Signal 從經典認證遷移至後量子認證時（此為不可避免的破壞性變更），正是同時將經典部分替換為 XHMQV 的最佳時機，獲得效能提升（比 X3DH 快 2 倍，精簡模式快 70%）和更強安全性，而不增加額外的破壞性變更。

## 與 TWCA 臺灣網路認證的關聯

1. **金鑰交換協議的效率與安全性權衡**：XHMQV 展示了如何透過精巧的代數結構（隨機化因子組合），在減少運算量的同時增強安全性。TWCA 在設計後量子安全的認證協議或金鑰交換機制時，可借鑑此「以數學結構換取效率」的方法論。

2. **金鑰重用的安全建模**：XHMQV 是唯一正確建模金鑰重用（同一金鑰同時用於 DH 和簽章）的分析。TWCA 的 PKI 實務中，金鑰對可能被用於多種目的（簽章、加密、認證），此研究提醒需在安全分析中準確反映這種重用。

3. **後量子遷移的漸進策略**：XHMQV 提出的「利用不可避免的破壞性變更窗口同時部署多項改進」策略，對 TWCA 規劃後量子遷移路線圖有參考價值。TWCA 可在必須升級後量子憑證時，同時引入其他安全性與效率改進。

4. **非同步認證的應用**：XHMQV 的非同步特性（一方可離線）對 TWCA 的某些服務場景有啟發，例如行動裝置上的憑證驗證或離線簽章場景。
