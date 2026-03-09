# Signal Lost (Integrity): The Signal App is More than the Sum of its Protocols

## Session

- **Session:** Beyond Secure Messaging
- **Chair:** Cas Cremers
- **Conference:** RWC 2026

## Speaker(s)

- **Noemi Terzo** — ETH Zurich
- **Kien Tuong Truong** — ETH Zurich

## Authors

- Kien Tuong Truong (ETH Zurich)
- Noemi Terzo (ETH Zurich)
- Peter Schwabe (Max-Planck Institute for Security and Privacy)
- Kenneth Paterson (ETH Zurich)

## Abstract

We present an attack against the integrity of conversations in Signal: we show that a malicious server can inject messages into a conversation between two honest users without them being aware of it. The attack does not require any key compromises. While the attack causes the honest users to receive a notification that their safety numbers have changed, those safety numbers remain consistent, so the attack cannot be detected by comparing them out-of-band. This attack naturally gives rise to a number of questions. How was this vulnerability introduced? How can such a vulnerability still be present after the extensive security analysis to which Signal's protocols have been subjected? What wider lessons can be drawn in order to prevent similar issues arising in the future? We answer these questions in detail in our talk.

## Summary

### 核心發現

研究者發現 Signal 存在**對話完整性攻擊**：惡意伺服器可以在兩位誠實使用者之間的對話中**注入偽造訊息**，且不需要任何金鑰外洩。演說中共揭露了兩個攻擊。

### 背景：Signal 的雙重身份系統

自 2024 年起，Signal 引入 username 功能，每位使用者有兩個身份識別碼：

- **ACI（Account Identifier）**：穩定的帳號識別碼，綁定公鑰，可透過 safety number 驗證
- **PNI（Phone Number Identifier）**：電話號碼識別碼，綁定另一組公鑰

當 Alice 透過電話號碼聯繫 Bob 時，會先建立 ACI-PNI session，Bob 再用 PNI 私鑰簽署 ACI 公鑰發給 Alice，將身份合併升級為 ACI-ACI session。研究者稱之為 **PNI-to-ACI protocol**——一個未被文件記載、也未被單獨分析過的跨協議互動流程。

### 攻擊一：透過 PNI-to-ACI Protocol 注入訊息（Desktop + Android）

1. 惡意伺服器 Mallory 為 Bob 的 PNI 生成一組假的金鑰對
2. Mallory 用假金鑰向 Alice 發送 pre-key message，冒充 Bob
3. Alice 會看到「safety number 已變更」的通知
4. 但當 Alice 與 Bob 比對 safety number 時，**雙方數字一致**，看似正常
5. Alice 收到 Mallory 注入的訊息，以為來自 Bob
6. 攻擊可重複執行，且**後續注入不會再觸發 safety number 變更通知**（因為重用同一 Mallory session）
7. 攻擊可雙向進行
8. **修復**：Signal 當天修復 Desktop，兩天後修復 Android。修補方式為不再接受來自 PNI 的訊息

### 攻擊二：完全不可偵測的訊息注入（僅 Android）

結合兩個問題：

1. **libsignal 中 Sealed Sender 的發送者認證缺陷**：Sealed Sender 用伺服器簽發的 certificate 將公鑰綁定到地址，但接收端不會驗證該公鑰是否為已知的發送者公鑰。因此惡意伺服器可以生成自己的金鑰對、自行簽發 certificate，以任何使用者身份發送 Sealed Sender 訊息。

2. **Android 版本的訊息類型驗證缺陷**：Signal 有三種訊息類型——whisper（Double Ratchet 加密）、sender key（群組加密）、plain text（未認證，正常只用於傳送解密錯誤通知）。正常情況下 plain text 中的文字訊息會被拒絕，但 Android 版本僅檢查外層 envelope 的 type 欄位。當訊息被 Sealed Sender 包裹後，所有訊息的 envelope type 都變成 `unidentified_sender`，導致 plain text 內的文字訊息繞過驗證被接受。

**結果**：惡意伺服器可以用偽造的 Sealed Sender 包裹 plain text 訊息，以任何使用者身份發送文字訊息到 Android 用戶端，**完全不觸發 safety number 變更通知，攻擊不可偵測**。

- 僅影響 Android 版本
- 揭露後約一週修復，修補方式為正確驗證 Sealed Sender 包裹的 plain text 訊息

### 漏洞揭露時程

- 2025 年 9 月負責任揭露給 Signal
- Signal 回應迅速：Desktop 當天修復，Android 兩天後修復攻擊一、一週後修復攻擊二
- 第三方使用 libsignal 的專案（如 Sicness CLI、Whisperfish）也需個別通知修補

### 根本教訓：見樹不見林

Signal 的各個密碼學協議（X3DH、Double Ratchet、Sesame、Sealed Sender）都經過多年密集的學術安全分析，但都是**單獨分析**。協議之間的互動——特別是 PNI-to-ACI 這類未被文件化的跨協議流程——從未被整體檢視。

研究者對業界與學界提出的建議：

- **系統層級的密碼學安全分析仍是盲區**，現有形式化工具尚不足以捕捉完整系統的複雜度（例如訊息類型的區分、協議間的互動）
- **開源程式碼有助於安全分析**，封閉原始碼的通訊軟體將更難發現類似漏洞
- **文件化所有密碼學相關流程**，包括看似瑣碎但涉及密碼學的功能（如 PNI-to-ACI protocol）
- **將密碼學邏輯統一在單一原生函式庫（libsignal）**，避免散佈在平台相依的程式碼中——本次攻擊二就是因為 Android 的驗證邏輯不在 libsignal 中，導致只有 Android 受影響
- **複雜度來自隱私增強功能**，Signal 在技術上領先，但「with great features come great responsibilities」

### 現實影響

如果 Bob 是政治人物、Alice 是高風險環境中的記者，一則偽造的訊息可能誘導 Alice 前往危險地點，甚至危及生命安全。

## 與 TWCA 臺灣網路認證的關聯

本場演講揭露的是 Signal 協議層級的完整性漏洞，涉及跨協議互動與訊息類型驗證缺陷，屬於端對端加密應用的內部安全問題，與憑證機構（CA）的 PKI、數位簽章等業務無直接關係。
