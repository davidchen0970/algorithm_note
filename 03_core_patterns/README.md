## Part 3　核心演算法模式

### 這一 Part 在做什麼

這一 Part 將常見題目整理成可重複使用的掃描與搜尋模式。它不是模板集合，而是訓練你辨認候選空間、維護必要狀態，以及證明某些候選可以被安全略過。Enumeration 是基準，Two Pointers、Sliding Window、Prefix Sum 與 Binary Search 則分別減少不同形式的重複工作。

### 學完後應該能做到

- 從暴力列舉辨認效能瓶頸
- 使用雙指標維護位置關係或排除候選
- 使用 Sliding Window 維護連續區間狀態
- 使用 Prefix Sum 重用區間累積結果
- 利用單調性進行 Binary Search

### 建議閱讀方式

- 先理解 Enumeration 作為比較基準。
- 再依序閱讀 Two Pointers、Sliding Window、Prefix Sum 與 Binary Search。
- 每個模式都要能說明成立條件，不只記住迴圈外形。

### 閱讀時可以反覆問自己

- 這個主題要解決哪一類問題？
- 它成立需要哪些前提？
- 目前維護的 State 或 Invariant 是什麼？
- 如果條件改變，原方法是否仍然成立？
- 時間與空間成本主要來自哪裡？
