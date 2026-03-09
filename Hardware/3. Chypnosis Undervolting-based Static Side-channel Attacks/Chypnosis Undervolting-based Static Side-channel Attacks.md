# Chypnosis: Undervolting-based Static Side-Channel Attacks

## Session

- **Session:** Hardware
- **Conference:** RWC 2026

## Speaker(s)

- **Shahin Tajik** — Worcester Polytechnic Institute (WPI)

## Authors

Kyle Mitard, Saleh Khalaj Monfared, Fatemeh Khojasteh Dana, Robert Dumitru, Yuval Yarom, Shahin Tajik

**機構：** Worcester Polytechnic Institute (WPI), Ruhr University Bochum (RUB), University of Adelaide

**發表：** To appear at IEEE S&P 2026

## Abstract

Chypnosis（chip + hypnosis）是一種基於降壓（undervolting）的靜態旁通道攻擊。透過快速降低晶片供電電壓至「休眠區間」（brownout region），使晶片時脈停止但資料保留，從而繞過安全感測器的回應機制，再利用雷射邏輯狀態成像（LLSI）或阻抗分析（IA）等靜態探測技術逐位元提取密鑰。攻擊已在 AMD/Xilinx 和 Microchip FPGA 上驗證，並成功攻破 OpenTitan Earl Grey 的 domain-oriented masking AES 實作。

## Summary

### 物理旁通道攻擊回顧

傳統物理旁通道攻擊（功率分析、電磁分析、光子發射分析等）的核心假設：**資訊洩漏發生在資料/狀態轉換時**（電晶體從 0→1 或 1→0 切換）。過去 25 年以來，已發展出大量基於此假設的攻擊與防禦（如 masking、hiding）。

### 靜態反向散射旁通道攻擊的興起

近年興起的**靜態反向散射旁通道攻擊**（Static Backscattered Side-Channel Attacks）根本性地改變了遊戲規則：

- 利用失效分析（FA）、功率/訊號完整性（PI/SI）、電磁相容性（EMC）領域的既有工具
- **主動刺激**裝置（射頻/微波訊號、雷射、電子束），測量**調變後的回波**
- 優勢：可達到更高的**訊雜比**（SNR），因為攻擊者控制刺激強度
- 關鍵能力：可探測**靜態訊號**——不需要狀態轉換即可提取資訊
- 後果：依賴隨機化的防禦措施（如 masking）**失效**

#### 範例 1：雷射邏輯狀態成像（LLSI）

IEEE S&P 2021 提出。雷射從晶片背面掃描，同時調變電壓線。處於 on-state 的電晶體會調變反射雷射光，產生與 off-state 不同的反射訊號。可同時讀取所有 flip-flop 的狀態值，無論使用多少 masking shares 都能同時提取。

#### 範例 2：阻抗分析（IA）

ACM CCS 2023 提出。晶片的電源傳輸網路阻抗具有**資料相依性**——不同儲存資料改變電晶體開關狀態，進而改變電路阻抗。透過電源線傳送微波訊號並測量反射（相位與振幅調變），可在不同頻率探測不同位置的 flip-flop，實現非侵入式攻擊。

### 現實世界的挑戰

靜態攻擊在學術論文中效果顯著，但在真實安全晶片上面臨兩大挑戰：

**挑戰 1：時脈控制**
- 大多數安全晶片使用**內部時脈**（ring oscillator），攻擊者無法從外部停止
- 晶片內建**時脈異常偵測**（如 PLL-based clock sensor）

**挑戰 2：安全感測器**
- **電壓感測器**（ADC-based voltage tampering detection）
- 範例：AMD/Xilinx SecMon soft IP、Microchip Anti-Tamper hard IP
- 偵測到異常後會觸發警報並清除機密資料

### Chypnosis 攻擊原理

#### 發現：Brownout Region（休眠區間）

晶片在正常電壓（如 1V）與崩潰電壓之間存在一個**休眠區間**：

- 電壓過低 → 晶片崩潰，資料遺失
- 電壓略低 → 晶片正常運作
- **休眠區間**（約 0.55V）→ 晶片**沒有能量進行切換**，時脈停止，但**資料保留**

這是一個**設計特性**——為了防止電壓波動導致資料遺失，記憶體單元被設計為在低電壓下仍能保持資料。

**驗證方式**：
- 光子發射分析：正常電壓下可見 ring oscillator 發光，降至休眠電壓後 oscillator 消失但仍有漏電流發射
- 雷射電壓成像：正常電壓下可見 clock buffer 活動，休眠電壓下 clock buffer 消失但資料完整

#### 繞過感測器：快速降壓

關鍵發現：**如果降壓速度夠快**，感測器會偵測到異常，但**沒有足夠時間回應**。

在 AMD/Xilinx Kintex-7 和 Microchip PolarFire FPGA 上驗證：
- 慢速降壓 → 感測器偵測到並觸發警報 ✓
- **快速降壓** → 感測器偵測到，但**無回應** ✗

原因：晶片在感測器完成回應（如清除金鑰）之前就已經耗盡能量。特別是使用**同步重置**（synchronous reset/clear）的設計，需要等到下一個時脈上升沿才能執行，但此時時脈已經停止。

### 攻擊 OpenTitan Earl Grey

將 OpenTitan Earl Grey 實作在 Xilinx Kintex-7 FPGA 上，連接 XADC 電壓感測器。OpenTitan 內部有完整的 alert handler → AES domain-oriented masking 金鑰清除流程。

**攻擊結果**：
- Chypnosis 快速降壓後，晶片能量耗盡
- 警報訊號**未能傳播**到 AES 加密引擎
- AES 金鑰暫存器**保留完整**
- 使用 LLSI 和 IA 逐位元提取金鑰（每個位元需數分鐘，但時脈已停止，有無限時間）

### 防禦措施

提出的防禦機制（已開源：`github.com/0xADE1A1DE/Borrowed-Time`）：

**偵測**：使用**非同步組合邏輯時脈感測器**——不需要時脈信號就能偵測異常

**回應**：使用**非同步重置**（asynchronous reset clearing）——不依賴時脈邊沿即可清除 flip-flop

**防止清除洩漏**：
- 清除操作本身也是計算（會產生動態洩漏）
- 使用**互補 flip-flop**：每個位元都有對應的互補位元，清零時 0→1 和 1→0 數量相等
- 使用**預存隨機數**進行隨機化覆寫（而非簡單歸零），防止清除過程的洩漏

### 廠商回應

- 向 AMD 和 Microchip 進行了負責任揭露
- 廠商要求 **90 天禁令期**後發布安全公告
- AMD 的安全公告中列出了**大量受影響的晶片型號**——雖然研究者僅測試了一款，但多個產品家族均受影響
- Microchip 發布 PSIRT-118 公告

## 與 TWCA 臺灣網路認證的關聯

1. **安全晶片評估的新威脅模型**：TWCA 在採購或評估 HSM、安全元件時，Chypnosis 揭示了一個被忽視的攻擊面——降壓休眠攻擊可繞過傳統安全感測器。TWCA 應要求供應商說明其安全晶片是否具備非同步重置機制來防護此類攻擊。

2. **Masking 防護的侷限性**：TWCA 的密碼運算設備若依賴 masking 作為旁通道防護手段，本研究證明靜態探測可同時讀取所有 shares，使 masking 完全失效。需要結合實體防護（如主動遮蔽層）而非僅依賴演算法層面的防禦。

3. **FPGA-based 安全方案的風險**：若 TWCA 使用 FPGA 實作安全功能（如加密加速器），AMD/Xilinx 和 Microchip 兩大 FPGA 廠商的產品均被確認受此攻擊影響。應確認部署的 FPGA 型號是否在受影響清單中。

4. **OpenTitan 的安全啟示**：前一場演講（Hardware #2）介紹了 OpenTitan 的 PQC 遷移，本場則展示 OpenTitan Earl Grey 的 AES masking 可被 Chypnosis 攻破。TWCA 若評估 OpenTitan 相關方案，需注意此已知漏洞及其修補狀態。
