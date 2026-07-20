## Part 7　Graph

### 這一 Part 在做什麼

這一 Part 處理節點之間的一般關係。Graph 可以表示道路、依賴、網路、狀態轉換與連通關係。解題第一步通常是建模：什麼是 Vertex、什麼是 Edge、方向與權重代表什麼。之後才選擇 DFS、BFS、Topological Sort、DSU、Shortest Path 或 MST。

### 學完後應該能做到

- 從題意定義 Vertex、Edge、方向與權重
- 使用 Adjacency List 或 Matrix 表示 Graph
- 以 DFS、BFS 處理走訪、連通與最短邊數
- 辨認 DAG、連通元件、最短路徑與生成樹問題
- 根據邊權與問題目標選擇演算法

### 建議閱讀方式

- 先讀 Graph 基礎，再學 DFS 與 BFS。
- Topological Sort、DSU、Shortest Path、MST 建議在基本走訪穩定後閱讀。

### 閱讀時可以反覆問自己

- 這個主題要解決哪一類問題？
- 它成立需要哪些前提？
- 目前維護的 State 或 Invariant 是什麼？
- 如果條件改變，原方法是否仍然成立？
- 時間與空間成本主要來自哪裡？
