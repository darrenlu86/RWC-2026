# A Practical Wrapper Protocol for Metadata-Hiding in Messaging

## Session

- **Session:** Beyond Secure Messaging
- **Chair:** Cas Cremers
- **Conference:** RWC 2026

## Speaker(s)

- **Lea Thiemt** — FAU Erlangen-Nürnberg

## Authors

- Lea Thiemt (FAU Erlangen-Nürnberg)
- Paul Rösler (FAU Erlangen-Nürnberg)
- Alexander Bienstock (J.P. Morgan AI Research and J.P. Morgan AlgoCRYPT CoE)
- Rolfe Schmidt (Signal Messenger)
- Yevgeniy Dodis (New York University)

## Abstract

This talk is based on the paper "Generic Anonymity Wrapper for Messaging Protocols" which appears at CCS'25. This RWC talk focuses on presenting the most impactful and practical findings from the paper.

End-to-end encryption in modern messengers ensures the confidentiality of user messages so that network observers or servers cannot learn the content of messages. Even if devices are ever temporarily compromised and encryption keys are leaked, e.g., by trojans or at airport security, the protocols guarantee that past and future communication remains confidential. While confidentiality of protocols has been studied intensively, comparatively little attention was given to the anonymity of protocols. That is, in protocols like Double Ratchet or MLS, not only the ciphertext is transmitted, but also attached metadata, like sender and receiver ID. A server which observes all incoming and outgoing traffic can analyze this metadata to reveal social networks and can ultimately even learn the identity of communicating parties. This is particularly threatening for vulnerable user groups: For instance, journalists who communicate with activists rely on guaranteed anonymity. In fact, the former NSA director, Michael Hayden, made the significance of metadata undoubtedly clear when he said, "We kill people based on metadata."

The only widely used messenger, to the best of our knowledge, which currently implements measures to hide metadata is Signal. More concretely, Signal's Sealed Sender protocol functions as a wrapper protocol around ciphertexts and metadata to provide sender anonymity. While this is a commendable development, Sealed Sender comes with drawbacks. First, the protocol relies on the receiver's static long term keys. Considering that messaging sessions can last for months or years, it is likely that, at some point, the receiver keys become compromised. This immediately allows de-anonymization of all previous and future communication. Second, Sealed Sender is inefficient in group chats: Without Sealed Sender, the sender creates a constant-size ciphertext which all group members can decrypt. With Sealed Sender, the sender re-encrypts this ciphertext for each recipient, which means that the ciphertext size increases linearly in the number of group members.

In this talk, I present our practical anonymity wrapper protocol which fixes both these drawbacks of Sealed Sender and can be used to hide metadata of existing (group) messaging protocols. The key idea is that the communicating parties use the shared key material of the underlying messaging protocol to derive wrapper keys. In group communication, the resulting ciphertext size is constant. Moreover, our protocol provides strong anonymity guarantees such that, even if encryption secrets are ever compromised, past and future communication remains anonymous. We implement this approach and compare it to Signal's Sealed Sender: The performance evaluation shows that the wire size of small 1:1 messages goes down from 441 bytes to 114 bytes. For a group of 100 members, it reduces the wire size of outgoing group messages from 7240 bytes to 155 bytes. We see similar improvements in computation time for encryption and decryption, but these improvements come with substantial storage costs for receivers. Yet, by using a Bloom Filter to compress the receiver state, we are able to make this approach practical: Our resulting protocol is efficient and has a storage overhead of only a few hundred bytes for the sender and a few kilobytes for the receiver. Since this significantly improves on the currently deployed Sealed Sender protocol, Signal considers employing this solution.

## Summary

### 要解決的問題

現有的端對端加密通訊（如 Signal）雖然加密了訊息內容，但密文層的 metadata（如寄件人/收件人 ID、序號等 header 資訊）仍然暴露給伺服器和網路上的攻擊者。正如前 NSA 局長 Michael Hayden 所說：「我們根據 metadata 殺人。」

### Signal 現有方案（Sealed Sender）的缺點

- 依賴接收端的**長期公鑰**，一旦金鑰外洩，所有過去與未來的通訊都可被去匿名化
- 群組訊息的密文大小隨成員數**線性增長**，難以擴展到大群組
- 每則訊息都需要**昂貴的公鑰密碼學運算**

### 提出的方案：Anonymity Wrapper

利用通訊雙方已有的共享金鑰材料，衍生出**對稱式 wrapper key** 加上隨機 label 來包裹 metadata：

- 金鑰外洩下仍有**強匿名性保證**（forward & backward anonymity）
- 只用**對稱密碼學**，速度大幅提升
- 群組密文大小為**常數**，不隨成員數增長

### 效能比較

| 指標 | Sealed Sender | Anonymity Wrapper |
|------|--------------|-------------------|
| 1:1 訊息大小 | 441 bytes | 114 bytes |
| 100 人群組訊息大小 | 7,240 bytes | 155 bytes |
| 加密時間（大群組） | 6,052 μs | 21 μs |

### Receiver State 問題與解法

接收端需要預計算查詢表來識別收到的密文屬於哪位發送者，初始方案導致每位聯絡人需 ~600KB state。透過 **Counting Bloom Filter** 壓縮至 ~38KB/聯絡人（1,000 聯絡人約 3.8MB），無 false negative，false positive 極少且成本低。

### 結論

- **Signal 正在考慮部署此協議**
- 論文：eprint.iacr.org/2025/1619

## 與 TWCA 臺灣網路認證的關聯

本場演講聚焦於利用對稱密碼學隱藏通訊 metadata，屬於端對端加密協議的內部機制設計，與憑證機構（CA）的 PKI、數位簽章、SSL/TLS 等業務無直接關係。
