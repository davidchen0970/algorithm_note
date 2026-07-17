## 第 29 章　Depth-First Search

### 適用範圍

本章說明 Depth-First Search，簡稱 DFS。DFS 是一種走訪 Graph、Tree 或 Grid 的方法。它會先沿著一條路持續往下走，直到不能再走，再回到上一個分岔點，改走其他尚未探索的路。

第一次接觸 DFS 時，常見困難不是不會寫遞迴，而是不清楚：

- DFS 現在走到哪一個節點？
- 為什麼需要 `visited`？
- 走到沒有鄰居時，程式如何回到上一層？
- Recursive DFS 與 Iterative DFS 有什麼差異？
- Graph DFS、Tree DFS 與 Grid DFS 為什麼看起來不一樣？
- Connected Components 與 Cycle Detection 如何從 DFS 延伸？

這些問題通常不是語法問題，而是尚未把走訪過程整理成「目前位置」、「下一步候選」與「哪些位置已經處理過」。

本章會建立一套固定流程：

- 先將題目整理成節點與連線。
- 使用小型 Graph 手動走一次 DFS。
- 確認是否可能繞回已走過的節點。
- 使用 `visited` 保存已走訪狀態。
- 分別理解 Recursive DFS 與 Iterative DFS。
- 再延伸到 Connected Components、Cycle Detection 與 Grid DFS。

```mermaid
flowchart TD
    A[從起點開始] --> B[標記目前節點已走訪]
    B --> C[查看相鄰節點]
    C --> D{"有尚未走訪的鄰居嗎"}
    D -->|有| E[前往該鄰居]
    E --> B
    D -->|沒有| F[回到上一個分岔點]
    F --> G{"還有未探索分支嗎"}
    G -->|有| C
    G -->|沒有| H[走訪完成]
```

### 適用讀者

- 已理解 Array、Stack 與基本 Recursion，但第一次接觸 Graph Traversal 的讀者。
- 能看懂 DFS 程式，卻無法手動追蹤走訪順序的讀者。
- 不清楚 `visited` 為什麼必要的讀者。
- 容易混淆 DFS、Backtracking 與 BFS 的讀者。
- 想理解 Connected Components、Cycle Detection 與 Flood Fill 的讀者。

### 快速導覽

- [29.1 DFS 前到底要分析什麼](#291-dfs-前到底要分析什麼)：先整理節點、連線與輸出目標。
- [29.2 先不要寫程式：手動走一次 DFS](#292-先不要寫程式手動走一次-dfs)：建立走到底再回頭的直覺。
- [29.3 Visited State](#293-visited-state)：避免重複走訪與無限循環。
- [29.4 Recursive DFS](#294-recursive-dfs)：使用 Call Stack 保存回程位置。
- [29.5 Iterative DFS](#295-iterative-dfs)：使用明確的 Stack 控制走訪。
- [29.6 Recursive 與 Iterative DFS 的差異](#296-recursive-與-iterative-dfs-的差異)：比較狀態保存方式。
- [29.7 Connected Components](#297-connected-components)：計算互不相連的區塊。
- [29.8 Undirected Graph Cycle Detection](#298-undirected-graph-cycle-detection)：使用 Parent 判斷環。
- [29.9 Directed Graph Cycle Detection](#299-directed-graph-cycle-detection)：區分走訪中與已完成狀態。
- [29.10 Grid DFS 與 Flood Fill](#2910-grid-dfs-與-flood-fill)：將二維格子視為 Graph。
- [29.11 Entry 與 Exit Time](#2911-entry-與-exit-time)：理解進入與離開節點的時機。
- [29.12 常見問題與判讀](#2912-常見問題與判讀)：整理常見錯誤。
- [29.13 本章檢查表](#2913-本章檢查表)：確認是否掌握核心概念。
- [29.14 本章重點](#2914-本章重點)：回顧本章核心。

### 29.1 DFS 前到底要分析什麼

假設有以下無向 Graph：

```text
0 連到 1、2
1 連到 0、3
2 連到 0、4
3 連到 1
4 連到 2
```

題目要求：從節點 0 開始走訪所有可到達節點。

第一步不是直接寫遞迴，而是先整理問題：

<table>
<tr><th>分析項目</th><th>本題內容</th></tr>
<tr><td>節點</td><td>0、1、2、3、4</td></tr>
<tr><td>連線方向</td><td>無向</td></tr>
<tr><td>起點</td><td>0</td></tr>
<tr><td>輸出</td><td>所有從 0 可到達的節點</td></tr>
<tr><td>是否可能繞回原節點</td><td>會，例如 0 到 1 後可再走回 0</td></tr>
<tr><td>需要保存的狀態</td><td>每個節點是否已走訪</td></tr>
<tr><td>鄰居順序是否固定</td><td>依 Adjacency List 的儲存順序而定</td></tr>
</table>

這張表會直接影響程式：

- 因為是無向 Graph，每條 Edge 會在兩個方向出現。
- 因為可能從 0 走到 1，再從 1 走回 0，所以需要 `visited`。
- 因為 DFS 的合法順序可能不只一種，不能只背一組輸出順序。

### 29.2 先不要寫程式：手動走一次 DFS

先畫出 Graph：

```mermaid
graph TD
    N0[0] --- N1[1]
    N0 --- N2[2]
    N1 --- N3[3]
    N2 --- N4[4]
```

假設每個節點都依數字由小到大查看鄰居。

從 0 開始：

1. 到達 0，標記 0 已走訪。
2. 0 的第一個未走訪鄰居是 1，所以前往 1。
3. 到達 1，標記 1 已走訪。
4. 1 的鄰居 0 已走訪，下一個未走訪鄰居是 3。
5. 到達 3，標記 3 已走訪。
6. 3 沒有其他未走訪鄰居，回到 1。
7. 1 沒有其他未走訪鄰居，回到 0。
8. 0 的下一個未走訪鄰居是 2。
9. 到達 2，再前往 4。

走訪順序是：

```text
0, 1, 3, 2, 4
```

#### 逐輪執行

<table>
<tr><th>步驟</th><th>目前節點</th><th>動作</th><th>visited</th></tr>
<tr><td>1</td><td>0</td><td>標記 0，準備查看鄰居</td><td>{0}</td></tr>
<tr><td>2</td><td>1</td><td>從 0 前往 1</td><td>{0, 1}</td></tr>
<tr><td>3</td><td>3</td><td>從 1 前往 3</td><td>{0, 1, 3}</td></tr>
<tr><td>4</td><td>3</td><td>沒有新鄰居，回到 1</td><td>{0, 1, 3}</td></tr>
<tr><td>5</td><td>1</td><td>沒有新鄰居，回到 0</td><td>{0, 1, 3}</td></tr>
<tr><td>6</td><td>2</td><td>從 0 前往 2</td><td>{0, 1, 2, 3}</td></tr>
<tr><td>7</td><td>4</td><td>從 2 前往 4</td><td>{0, 1, 2, 3, 4}</td></tr>
</table>

DFS 的「深度優先」是指先把目前分支往下走到底，再回頭處理旁邊分支。

### 29.3 Visited State

`visited` 用來記錄哪些節點已經走訪過。

假設 Graph 中有：

```text
0 --- 1
```

從 0 走到 1 後，1 的鄰居又包含 0。如果沒有 `visited`，流程可能變成：

```text
0 -> 1 -> 0 -> 1 -> 0 -> ...
```

因此在進入節點時，通常要先標記：

```cpp
visited[node] = true;
```

再走訪鄰居：

```cpp
for (int neighbor : graph[node])
{
    if (!visited[neighbor])
    {
        dfs(neighbor);
    }
}
```

#### 何時標記 visited

Recursive DFS 常在一進入函式時標記。

Iterative DFS 可以在放入 Stack 時標記，也可以在取出 Stack 時標記。若等到取出時才標記，同一節點可能被不同鄰居重複放入 Stack。

為了減少重複加入，本章的 Iterative DFS 會在放入 Stack 時標記。

### 29.4 Recursive DFS

Recursive DFS 使用 Call Stack 保存「走完目前節點後要回到哪裡」。

#### C++ 實作

```cpp
#include <iostream>
#include <vector>

void dfsRecursive(
    const std::vector<std::vector<int>>& graph,
    int node,
    std::vector<bool>& visited)
{
    visited[node] = true;
    std::cout << node << ' ';

    for (int neighbor : graph[node])
    {
        if (!visited[neighbor])
        {
            dfsRecursive(graph, neighbor, visited);
        }
    }
}
```

呼叫方式：

```cpp
std::vector<bool> visited(graph.size(), false);
dfsRecursive(graph, 0, visited);
```

#### 遞迴正在保存什麼

當流程是：

```text
0 -> 1 -> 3
```

Call Stack 會保存：

```text
dfs(0)：仍要繼續查看 0 的其他鄰居
dfs(1)：仍要繼續查看 1 的其他鄰居
dfs(3)：目前正在處理
```

3 處理完成後，函式返回 1；1 處理完成後，再返回 0。

```mermaid
flowchart TD
    A[dfs 0] --> B[dfs 1]
    B --> C[dfs 3]
    C --> D[3 完成，回到 1]
    D --> E[1 完成，回到 0]
    E --> F[dfs 2]
    F --> G[dfs 4]
```

### 29.5 Iterative DFS

Iterative DFS 不依賴函式遞迴，而是自己建立 Stack。

Stack 保存接下來要處理的節點。

#### C++ 實作

```cpp
#include <iostream>
#include <stack>
#include <vector>

void dfsIterative(
    const std::vector<std::vector<int>>& graph,
    int start)
{
    std::vector<bool> visited(graph.size(), false);
    std::stack<int> pending;

    pending.push(start);
    visited[start] = true;

    while (!pending.empty())
    {
        int node = pending.top();
        pending.pop();

        std::cout << node << ' ';

        // 反向加入，讓較小的鄰居較早被處理。
        for (int i = static_cast<int>(graph[node].size()) - 1;
             i >= 0;
             --i)
        {
            int neighbor = graph[node][i];

            if (!visited[neighbor])
            {
                visited[neighbor] = true;
                pending.push(neighbor);
            }
        }
    }
}
```

#### 為什麼反向加入鄰居

Stack 是 Last In, First Out。

如果希望較小的鄰居先被取出，就要先把較大的鄰居放入 Stack，再放較小的鄰居。

若不在乎特定走訪順序，也可以直接依原順序加入。DFS 的正確性通常不要求唯一順序。

### 29.6 Recursive 與 Iterative DFS 的差異

<table>
<tr><th>項目</th><th>Recursive DFS</th><th>Iterative DFS</th></tr>
<tr><td>回程資訊</td><td>Call Stack</td><td>自行建立的 Stack</td></tr>
<tr><td>程式長度</td><td>通常較短</td><td>通常較長</td></tr>
<tr><td>深度過大風險</td><td>可能 Stack Overflow</td><td>可使用 Heap Memory 上的容器</td></tr>
<tr><td>走訪順序控制</td><td>由鄰居遞迴順序決定</td><td>由放入 Stack 的順序決定</td></tr>
<tr><td>適合情況</td><td>深度可控、結構清楚</td><td>深度很大或需明確控制 Stack</td></tr>
</table>

兩種版本的核心相同：

- 記錄已走訪節點。
- 沿著一條分支持續往下。
- 沒有新鄰居時回到之前的分岔點。

### 29.7 Connected Components

Connected Component 是無向 Graph 中彼此可互相到達的一組節點。

例如：

```mermaid
graph LR
    A[0] --- B[1]
    B --- C[2]
    D[3] --- E[4]
    F[5]
```

這個 Graph 有三個 Connected Components：

```text
{0, 1, 2}
{3, 4}
{5}
```

#### 解題想法

從未走訪節點啟動一次 DFS，該次 DFS 會走完整個 Component。

所以：

```text
啟動 DFS 的次數 = Connected Components 數量
```

#### C++ 實作

```cpp
void markComponent(
    const std::vector<std::vector<int>>& graph,
    int node,
    std::vector<bool>& visited)
{
    visited[node] = true;

    for (int neighbor : graph[node])
    {
        if (!visited[neighbor])
        {
            markComponent(graph, neighbor, visited);
        }
    }
}

int countComponents(const std::vector<std::vector<int>>& graph)
{
    std::vector<bool> visited(graph.size(), false);
    int components = 0;

    for (int node = 0; node < static_cast<int>(graph.size()); ++node)
    {
        if (!visited[node])
        {
            ++components;
            markComponent(graph, node, visited);
        }
    }

    return components;
}
```

不能只從節點 0 做一次 DFS，因為 Graph 可能不連通。

### 29.8 Undirected Graph Cycle Detection

在無向 Graph 中，鄰居包含上一個節點。若只看到「鄰居已走訪」就判定有 Cycle，會把返回 Parent 的 Edge 誤認為 Cycle。

因此 DFS 需要額外保存 `parent`。

判斷方式：

- 鄰居未走訪，繼續 DFS。
- 鄰居已走訪，而且鄰居不是 Parent，表示發現 Cycle。

```cpp
bool hasCycleUndirected(
    const std::vector<std::vector<int>>& graph,
    int node,
    int parent,
    std::vector<bool>& visited)
{
    visited[node] = true;

    for (int neighbor : graph[node])
    {
        if (!visited[neighbor])
        {
            if (hasCycleUndirected(graph, neighbor, node, visited))
            {
                return true;
            }
        }
        else if (neighbor != parent)
        {
            return true;
        }
    }

    return false;
}
```

若 Graph 不保證連通，要從每個尚未走訪節點啟動檢查。

### 29.9 Directed Graph Cycle Detection

Directed Graph 不能只用單一 `visited` 判斷 Cycle。

原因是走到一個「以前處理完成」的節點，不一定表示形成 Cycle。只有回到目前這條 DFS 路徑中的節點，才表示產生 Directed Cycle。

可以使用三種狀態：

<table>
<tr><th>狀態</th><th>意思</th></tr>
<tr><td>0</td><td>尚未走訪</td></tr>
<tr><td>1</td><td>正在目前 DFS 路徑中</td></tr>
<tr><td>2</td><td>該節點與後續分支已處理完成</td></tr>
</table>

```mermaid
flowchart TD
    A[進入節點] --> B[狀態設為 1]
    B --> C[檢查鄰居]
    C --> D{"鄰居狀態"}
    D -->|0| E[遞迴走訪]
    D -->|1| F[發現 Directed Cycle]
    D -->|2| G[該分支已完成，不是目前路徑]
    E --> C
    G --> C
    C --> H[所有鄰居完成]
    H --> I[狀態設為 2]
```

#### C++ 實作

```cpp
bool hasCycleDirected(
    const std::vector<std::vector<int>>& graph,
    int node,
    std::vector<int>& state)
{
    state[node] = 1;

    for (int neighbor : graph[node])
    {
        if (state[neighbor] == 1)
        {
            return true;
        }

        if (state[neighbor] == 0 &&
            hasCycleDirected(graph, neighbor, state))
        {
            return true;
        }
    }

    state[node] = 2;
    return false;
}
```

### 29.10 Grid DFS 與 Flood Fill

二維 Grid 也可以視為 Graph：

- 每一格是一個節點。
- 上、下、左、右相鄰格是 Edge。

例如計算島嶼數量時：

- `'1'` 表示陸地。
- `'0'` 表示水。
- 從一格尚未走訪的陸地啟動 DFS，可以標記整座島。

#### 四個方向

```cpp
const int directions[4][2] = {
    {-1, 0},
    {1, 0},
    {0, -1},
    {0, 1}
};
```

#### Grid DFS

```cpp
void floodFill(
    const std::vector<std::vector<char>>& grid,
    int row,
    int col,
    std::vector<std::vector<bool>>& visited)
{
    int rows = static_cast<int>(grid.size());
    int cols = static_cast<int>(grid[0].size());

    if (row < 0 || row >= rows ||
        col < 0 || col >= cols ||
        grid[row][col] == '0' ||
        visited[row][col])
    {
        return;
    }

    visited[row][col] = true;

    floodFill(grid, row - 1, col, visited);
    floodFill(grid, row + 1, col, visited);
    floodFill(grid, row, col - 1, visited);
    floodFill(grid, row, col + 1, visited);
}
```

#### Grid DFS 的固定檢查順序

每次進入一格時，先確認：

1. Row 是否越界。
2. Column 是否越界。
3. 是否為可以走的格子。
4. 是否已走訪。

需先檢查邊界，再讀取 `grid[row][col]`，避免越界。

### 29.11 Entry 與 Exit Time

Recursive DFS 對每個節點有兩個重要時機：

- Entry：剛進入節點時。
- Exit：所有鄰居都處理完成，即將返回時。

```cpp
void dfs(int node)
{
    visited[node] = true;
    // Entry

    for (int neighbor : graph[node])
    {
        if (!visited[neighbor])
        {
            dfs(neighbor);
        }
    }

    // Exit
}
```

Entry 適合處理「第一次看到節點」的工作。

Exit 適合處理「所有子分支都完成後」的工作，例如：

- Tree 的 Postorder 計算。
- Directed Graph 的完成順序。
- Topological Sort 的一種 DFS 寫法。

不要只記錄程式位置，而要確認題目需要在進入時處理，還是在所有子問題完成後處理。

### 29.12 常見問題與判讀

<table>
<tr><th>現象</th><th>可能原因</th><th>第一輪檢查</th></tr>
<tr><td>DFS 無限循環</td><td>Graph 有環但未使用 `visited`</td><td>進入節點時先標記</td></tr>
<tr><td>同一節點輸出多次</td><td>太晚標記 `visited`</td><td>確認是在加入待處理集合時或進入節點時標記</td></tr>
<tr><td>只走到部分節點</td><td>Graph 不連通，但只從單一起點 DFS</td><td>外層掃描所有節點</td></tr>
<tr><td>Iterative 與 Recursive 順序不同</td><td>Stack 放入鄰居的順序不同</td><td>若要相同順序，反向 push 鄰居</td></tr>
<tr><td>無向 Graph 誤判 Cycle</td><td>把返回 Parent 的 Edge 當成環</td><td>額外保存 `parent`</td></tr>
<tr><td>有向 Graph 漏判或誤判 Cycle</td><td>只使用單一 Boolean `visited`</td><td>使用未走訪、走訪中、已完成三種狀態</td></tr>
<tr><td>Grid 讀取越界</td><td>先讀格子再檢查 Row、Column</td><td>先做邊界判斷</td></tr>
<tr><td>輸入很深時崩潰</td><td>Recursive DFS 深度過大</td><td>改用 Iterative DFS</td></tr>
</table>

### 29.13 本章檢查表

- 我能說明 DFS 為什麼叫 Depth-First Search。
- 我能用小型 Graph 手動追蹤 DFS。
- 我知道 Graph 中通常需要 `visited`。
- 我能說明 Recursive DFS 如何利用 Call Stack 回到上一層。
- 我能使用明確 Stack 寫出 Iterative DFS。
- 我知道 DFS 走訪順序可能不唯一。
- 我能計算無向 Graph 的 Connected Components。
- 我能使用 Parent 判斷無向 Graph Cycle。
- 我能用三種 State 判斷有向 Graph Cycle。
- 我能將 Grid 視為節點與相鄰關係。
- 我會先檢查 Grid 邊界，再讀取格子內容。
- 我能區分 DFS 的 Entry 與 Exit 時機。
- 我知道時間複雜度要包含所有 Vertex 與 Edge。

### 29.14 本章重點

- DFS 會先沿著一條分支持續往下，再回頭探索其他分支。
- Graph DFS 通常需要 `visited`，避免重複走訪與無限循環。
- Recursive DFS 使用 Call Stack 保存回程資訊。
- Iterative DFS 使用明確的 Stack 保存待處理節點。
- 鄰居處理順序會影響 DFS 輸出順序，但合法 DFS 順序可能不只一種。
- 從每個未走訪節點啟動 DFS，可以計算 Connected Components。
- 無向 Graph 判斷 Cycle 時，需要排除返回 Parent 的 Edge。
- 有向 Graph 判斷 Cycle 時，需要區分走訪中與已完成節點。
- Grid DFS 會把每一格視為節點，把相鄰方向視為 Edge。
- Entry 表示第一次進入節點，Exit 表示所有後續分支都已完成。
- 使用 Adjacency List 時，DFS 的時間複雜度是 O(V + E)。
- DFS 的額外空間與 `visited`、Stack 或最大遞迴深度有關。
