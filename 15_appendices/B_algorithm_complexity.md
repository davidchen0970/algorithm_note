## 附錄 B　常見演算法複雜度表

### 適用範圍

本附錄提供常見演算法的快速查閱。複雜度必須搭配成立條件閱讀，例如 Binary Search 需要單調性，普通 BFS 的最短路需要每條 Edge 成本相同。

### B.1 搜尋

<table>
<tr><th>演算法</th><th>時間</th><th>空間</th><th>前置條件</th></tr>
<tr><td>Linear Search</td><td>O(n)</td><td>O(1)</td><td>無</td></tr>
<tr><td>Binary Search</td><td>O(log n)</td><td>O(1)</td><td>已排序或判斷具有單調性</td></tr>
<tr><td>Hash Lookup</td><td>平均 O(1)</td><td>O(n)</td><td>可 Hash 的 Key</td></tr>
</table>

### B.2 排序

<table>
<tr><th>演算法</th><th>最佳</th><th>平均</th><th>最差</th><th>額外空間</th></tr>
<tr><td>Bubble Sort，含提前停止</td><td>O(n)</td><td>O(n²)</td><td>O(n²)</td><td>O(1)</td></tr>
<tr><td>Selection Sort</td><td>O(n²)</td><td>O(n²)</td><td>O(n²)</td><td>O(1)</td></tr>
<tr><td>Insertion Sort</td><td>O(n)</td><td>O(n²)</td><td>O(n²)</td><td>O(1)</td></tr>
<tr><td>Merge Sort</td><td>O(n log n)</td><td>O(n log n)</td><td>O(n log n)</td><td>O(n)</td></tr>
<tr><td>Heap Sort</td><td>O(n log n)</td><td>O(n log n)</td><td>O(n log n)</td><td>O(1)</td></tr>
<tr><td>Counting Sort</td><td>O(n + K)</td><td>O(n + K)</td><td>O(n + K)</td><td>O(n + K)</td></tr>
</table>

K 是 Key Range 大小。Quick Sort 與標準函式的具體保證應依實作與規格查閱。

### B.3 Tree Traversal

<table>
<tr><th>工作</th><th>時間</th><th>額外空間</th></tr>
<tr><td>DFS Traversal</td><td>O(n)</td><td>O(h)</td></tr>
<tr><td>BFS Traversal</td><td>O(n)</td><td>O(w)</td></tr>
</table>

h 是 Tree 高度，w 是最大寬度。

### B.4 Graph Traversal

<table>
<tr><th>表示方式</th><th>DFS / BFS 時間</th><th>額外空間</th></tr>
<tr><td>Adjacency List</td><td>O(V + E)</td><td>O(V)</td></tr>
<tr><td>Adjacency Matrix</td><td>O(V²)</td><td>O(V)</td></tr>
</table>

### B.5 Shortest Path

<table>
<tr><th>演算法</th><th>常見時間</th><th>主要條件</th></tr>
<tr><td>BFS</td><td>O(V + E)</td><td>Unweighted 或相同 Edge Cost</td></tr>
<tr><td>0-1 BFS</td><td>O(V + E)</td><td>Weight 只有 0 或 1</td></tr>
<tr><td>Dijkstra with Binary Heap</td><td>O((V + E) log V)</td><td>Weight 非負</td></tr>
<tr><td>Bellman-Ford</td><td>O(VE)</td><td>可處理負 Edge，並偵測可達負環</td></tr>
<tr><td>Floyd-Warshall</td><td>O(V³)</td><td>All-Pairs，空間 O(V²)</td></tr>
</table>

### B.6 Minimum Spanning Tree

<table>
<tr><th>演算法</th><th>常見時間</th><th>主要結構</th></tr>
<tr><td>Kruskal</td><td>O(E log E)</td><td>Edge Sort + DSU</td></tr>
<tr><td>Prim with Binary Heap</td><td>O((V + E) log V)</td><td>Adjacency List + Heap</td></tr>
</table>

適用於 Weighted Undirected Graph。

### B.7 String Matching

<table>
<tr><th>方法</th><th>時間</th><th>注意事項</th></tr>
<tr><td>Naive Matching</td><td>O(nm)</td><td>逐位置比較 Pattern</td></tr>
<tr><td>KMP</td><td>O(n + m)</td><td>需建立 Prefix Function</td></tr>
<tr><td>Rabin-Karp</td><td>平均接近 O(n + m)</td><td>Hash Collision 需驗證</td></tr>
</table>

### B.8 Dynamic Programming

DP 沒有單一固定複雜度。一般估算方式：

```text
時間 = State 數量 × 每個 State 的 Transition 數量
空間 = 保存的 State 數量 + 遞迴 Stack
```

<table>
<tr><th>類型</th><th>常見 State 數</th><th>常見時間</th></tr>
<tr><td>一維 DP</td><td>O(n)</td><td>O(n) 或 O(nk)</td></tr>
<tr><td>二維 Grid DP</td><td>O(rows × cols)</td><td>依每格 Transition 數</td></tr>
<tr><td>Interval DP</td><td>O(n²)</td><td>常見 O(n³)</td></tr>
<tr><td>Bitmask DP</td><td>O(2^n × n)</td><td>常見 O(2^n × n²)</td></tr>
</table>

### B.9 Range Query

<table>
<tr><th>方法</th><th>建立</th><th>Query</th><th>Update</th></tr>
<tr><td>Prefix Sum</td><td>O(n)</td><td>O(1)</td><td>一般 O(n)</td></tr>
<tr><td>Difference Array</td><td>O(n)</td><td>重建後讀取</td><td>Range Update O(1)</td></tr>
<tr><td>Fenwick Tree</td><td>O(n) 或 O(n log n)</td><td>O(log n)</td><td>O(log n)</td></tr>
<tr><td>Segment Tree</td><td>O(n)</td><td>O(log n)</td><td>O(log n)</td></tr>
<tr><td>Sparse Table</td><td>O(n log n)</td><td>靜態 RMQ 常見 O(1)</td><td>不適合頻繁更新</td></tr>
</table>

### 使用檢查表

- 複雜度中的 n、V、E、K、m 分別代表什麼？
- 前置條件是否成立？
- 成本是 Worst、Average 還是 Amortized？
- 是否漏算排序、建圖或預處理？
- 是否漏算輸出本身的大小？
- 遞迴 Stack、Heap、DP Table 是否列入空間？
