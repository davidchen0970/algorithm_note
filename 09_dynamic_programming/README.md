## Part 9　Dynamic Programming

### 這一 Part 在做什麼

這一 Part 處理可拆成子問題、而且中間答案值得保存的問題。Dynamic Programming 的核心不是建立 `dp` Array，而是定義 State、推導 Transition、設定 Base Case、安排計算順序，並確認 State 保存的資訊足以支援未來。後續題型會依 State 維度與問題結構逐步擴充。

### 學完後應該能做到

- 用完整句子定義 State
- 從最後一步或合法選擇推導 Transition
- 區分 Top-down Memoization 與 Bottom-up Tabulation
- 處理一維、Grid、Knapsack、Subsequence、Interval、Tree 與 State Compression DP
- 分析 State 數量、Transition 成本與 Reconstruction

### 建議閱讀方式

- 第 37 章先建立共同分析流程。
- 第 38 至第 44 章依題型逐步擴充。
- 先完成正確的完整 State，再考慮空間壓縮。

### 閱讀時可以反覆問自己

- 這個主題要解決哪一類問題？
- 它成立需要哪些前提？
- 目前維護的 State 或 Invariant 是什麼？
- 如果條件改變，原方法是否仍然成立？
- 時間與空間成本主要來自哪裡？
