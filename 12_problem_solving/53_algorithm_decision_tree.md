## 第 53 章　演算法選擇決策樹

### 適用範圍

本章將常見題型整理成決策流程。決策樹用來提出候選方法，不取代 Precondition、正確性證明與複雜度分析。

### 53.1 主決策樹

```mermaid
flowchart TD
    A[定義輸入、輸出、限制] --> B{答案是否為 Graph 關係}
    B -->|是| G{最短路、連通、相依或 MST}
    G --> G1[依 Edge 方向與 Weight 選方法]
    B -->|否| C{是否為連續區間}
    C -->|是| W[Window、Prefix、Deque、Range Query]
    C -->|否| D{是否可排序或已有單調性}
    D -->|是| S[Binary Search、Two Pointers、Greedy]
    D -->|否| E{是否有重複 State}
    E -->|是| P[Memoization / DP]
    E -->|否| F[Enumeration、Backtracking、Hash 或其他模型]
```

每個葉節點都只是下一步調查方向。

### 53.2 排序與單調性

```mermaid
flowchart TD
    A[資料有序或可排序] --> B{找邊界位置嗎}
    B -->|是| C[Lower / Upper Bound]
    B -->|否| D{兩端移動可排除候選嗎}
    D -->|是| E[Two Pointers]
    D -->|否| F{局部選擇可證明安全嗎}
    F -->|是| G[Greedy]
    F -->|否| H[排序後掃描、DP 或搜尋]
```

排序前要檢查原 Index、穩定性與連續性需求。

### 53.3 連續區間

- 固定長度且 State 可增量更新：固定 Sliding Window。
- 可變長度且 Validity 單調：Variable Sliding Window。
- 大量靜態 Sum Query：Prefix Sum。
- Sum 等於 k 且可含負數：Prefix Sum + Hash。
- Window Maximum：Monotonic Deque。
- 動態 Range：Fenwick / Segment Tree。

```mermaid
flowchart TD
    A[連續區間] --> B{固定長度}
    B -->|是| C[Fixed Window]
    B -->|否| D{Expand Shrink 單調嗎}
    D -->|是| E[Variable Window]
    D -->|否| F{可由 Prefix 關係描述嗎}
    F -->|是| G[Prefix + Hash / Binary Search]
    F -->|否| H[DP、Deque、Tree]
```

### 53.4 Graph 分支

```mermaid
flowchart TD
    A[Graph 問題] --> B{主要目標}
    B -->|Reachability/Component| C[BFS / DFS]
    B -->|相依順序| D[Topological Sort]
    B -->|單源最短路| E[依 Weight 選 BFS、0-1 BFS、Dijkstra、Bellman-Ford]
    B -->|連接全部 Node 最低成本| F[MST]
    B -->|動態合併查詢| G[DSU]
```

Directed、Undirected 與 Weight 是必要前置資訊。

### 53.5 DP 分支

若選擇序列會產生重複 State：

```mermaid
flowchart TD
    A[重複子問題] --> B{State 由哪些欄位唯一決定}
    B --> C[定義 Transition]
    C --> D[定義 Base Case]
    D --> E{Dependency 是否無環}
    E -->|是| F[Memoization 或 Bottom-up]
    E -->|否| G[重新定義 State 或改用 Graph 方法]
```

常見分類：

- Prefix / Sequence DP。
- Grid DP。
- Knapsack。
- Subsequence DP。
- Interval DP。
- Tree DP。

### 53.6 Greedy 或完整搜尋

Greedy 需要 Exchange、Stay-ahead、Cut Property 等證明。若無法證明，先使用：

- Enumeration。
- Backtracking。
- DP。
- Branch and Bound。

```mermaid
flowchart TD
    A[提出局部最佳選擇] --> B{可證明存在最佳解包含它嗎}
    B -->|是| C[Greedy]
    B -->|否| D[找反例]
    D --> E[DP 或完整搜尋]
```

### 53.7 動態更新

| Update / Query | 常見工具 |
|---|---|
| 無 Update，多次 Prefix Sum | Prefix Sum |
| 批次 Range Add，最後一次輸出 | Difference Array |
| Point Update、Prefix Sum | Fenwick Tree |
| 一般 Range Query / Update | Segment Tree |
| 動態 Connected Component 合併 | DSU |

### 53.8 複雜度過濾

先由限制估算可接受規模：

```text
n 約 10^5：通常不能 O(n²)
n 約 20：2^n 可能可行
n 約 10：n! 仍需評估
V、E 大：優先 Adjacency List
```

這些只是量級判讀，實際限制還受常數、記憶體、語言與時間限制影響。

### 53.9 決策結果驗證

選出候選方法後，仍需回答：

1. Precondition 是否成立？
2. State 或 Invariant 是什麼？
3. 每次移動或選擇排除哪些候選？
4. 是否一定終止？
5. 時間與空間是否符合限制？
6. 反例與邊界案例是否通過？

### 53.10 常見誤判

| 誤判 | 檢查方向 |
|---|---|
| 有兩個 Pointer 就叫 Sliding Window | 是否維護連續 Window State |
| 有「最短」就用 BFS | Edge Cost 是否相同 |
| 有「相依」就一定能排序 | Graph 是否有 Cycle |
| 有最佳化就用 Greedy | 是否有正確性證明 |
| 有區間就用 Prefix Sum | Operation 是否可由 Prefix 抵消 |
| 有重複工作就一定是 DP | State 是否有限且 Transition 明確 |

### 53.11 本章檢查表

- 我先定義問題，再走決策樹。
- 我知道決策樹只提出候選方法。
- 我會確認排序、單調性、連續性與 Weight。
- 我能區分 Graph、DP、Greedy、Window 與搜尋問題。
- 我會用輸入限制過濾不可行複雜度。
- 我能為最後選擇的方法補上證明與測試。

### 53.12 本章重點

- 決策樹用於縮小方法範圍，不取代正確性推導。
- 排序與單調性常導向 Binary Search、Two Pointers 或 Greedy。
- 連續區間需區分 Window、Prefix 與動態 Range Query。
- Graph 方法由目標、方向與 Weight 決定。
- 重複 State 可考慮 DP，但要先定義 State 與 Transition。
- Greedy 沒有證明時，應回到搜尋、DP 或反例分析。
