## Part 2　基礎資料結構

### 這一 Part 在做什麼

這一 Part 說明資料在程式中如何被保存、存取、插入、刪除與走訪。資料結構會直接影響演算法能否有效率地取得所需資訊。學習時不要只記 API，應同時理解記憶體模型、成本、生命週期，以及每種結構適合回答的問題。

### 學完後應該能做到

- 比較 Array、Linked List、Stack、Queue 與 Hash-based Structure 的成本
- 根據主要動作選擇資料結構
- 理解 Index、Pointer、Iterator 與 Ownership 對正確性的影響
- 辨認 LIFO、FIFO、Key-Value 與連續儲存模型

### 建議閱讀方式

- Array 與 String 建立線性資料基礎。
- Linked List 補充 Pointer 與鏈結更新。
- Stack、Queue、Hash Table 再依存取模式延伸。

### 閱讀時可以反覆問自己

- 這個主題要解決哪一類問題？
- 它成立需要哪些前提？
- 目前維護的 State 或 Invariant 是什麼？
- 如果條件改變，原方法是否仍然成立？
- 時間與空間成本主要來自哪裡？
