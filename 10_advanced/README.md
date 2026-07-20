## Part 10　進階資料結構與演算法

### 這一 Part 在做什麼

這一 Part 收錄在特定限制下能顯著提升效率的工具。Bit Manipulation 使用位元表示或轉換狀態；Fenwick Tree 與 Segment Tree 處理動態區間查詢；String Matching 利用字串結構避免重複比較；Line Sweep 與 Coordinate Compression 則把事件與巨大座標轉成可處理的順序。

### 學完後應該能做到

- 用 Bit Mask 表示集合或有限狀態
- 處理更新與 Prefix / Range Query
- 理解字串匹配中的前綴資訊與失配跳轉
- 用事件排序處理區間與幾何問題
- 將稀疏大座標壓縮成相對順序

### 建議閱讀方式

- 這些主題不必一次讀完，可依題目需求查閱。
- 閱讀前先確認基礎 Tree、Prefix Sum、Binary Search 與排序觀念。

### 閱讀時可以反覆問自己

- 這個主題要解決哪一類問題？
- 它成立需要哪些前提？
- 目前維護的 State 或 Invariant 是什麼？
- 如果條件改變，原方法是否仍然成立？
- 時間與空間成本主要來自哪裡？
