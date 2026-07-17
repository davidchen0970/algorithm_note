## 第 27 章　Heap 與 Priority Queue

### 適用範圍

本章說明 Heap 與 Priority Queue，以及它們適合處理哪些「反覆取得目前最小值或最大值」的問題。

第一次接觸 Heap 時，常見困難不是不會呼叫 `std::priority_queue`，而是不清楚：

- Heap 是不是一棵完整排序的 Tree？
- 為什麼只保證 Root 是最小值或最大值？
- Tree 為什麼可以放在 Array 裡？
- 插入後為什麼要 Heapify Up？
- 移除頂端後為什麼要 Heapify Down？
- `priority_queue` 預設到底是 Min Heap 還是 Max Heap？
- Top K 為什麼有時使用 Min Heap，有時使用 Max Heap？

這些問題通常不是語法問題，而是尚未分清楚「Heap 保存的性質」與「排序陣列保存的性質」。

本章會建立一套固定流程：

- 先確認題目是否反覆需要目前最小值或最大值。
- 使用小型資料手動畫出 Heap。
- 理解 Complete Binary Tree 與 Array Index 的關係。
- 分別追蹤插入、Heapify Up、移除頂端與 Heapify Down。
- 再使用 C++ `std::priority_queue`。
- 最後延伸到 Top K、K-way Merge、Median Maintenance 與 Lazy Deletion。

```mermaid
flowchart TD
    A[拿到題目] --> B{"是否反覆需要目前最小值或最大值"}
    B -->|否| C[先分析其他資料結構]
    B -->|是| D{"資料是否會持續加入或移除"}
    D -->|是| E[考慮 Heap 或 Priority Queue]
    D -->|否| F{"只查詢一次嗎"}
    F -->|是| G[一次線性掃描可能已足夠]
    F -->|否| E
```

### 適用讀者

- 已理解 Array 與 Binary Tree，但第一次接觸 Heap 的讀者。
- 會使用 `priority_queue`，但不清楚內部維護方式的讀者。
- 容易混淆 Min Heap、Max Heap 與完整排序的讀者。
- 不熟悉 Parent、Left Child、Right Child Index 公式的讀者。
- 想理解 Top K、K-way Merge 與 Median Maintenance 的讀者。

### 快速導覽

- [27.1 Heap 前到底要分析什麼](#271-heap-前到底要分析什麼)：先確認題目是否需要動態極值。
- [27.2 Heap 不是完整排序](#272-heap-不是完整排序)：理解 Heap Property。
- [27.3 Min Heap 與 Max Heap](#273-min-heap-與-max-heap)：區分頂端元素的意義。
- [27.4 使用 Array 表示 Heap](#274-使用-array-表示-heap)：建立 Tree 與 Index 的關係。
- [27.5 Insert 與 Heapify Up](#275-insert-與-heapify-up)：插入後向上修復。
- [27.6 Pop 與 Heapify Down](#276-pop-與-heapify-down)：移除頂端後向下修復。
- [27.7 Build Heap](#277-build-heap)：從既有 Array 建立 Heap。
- [27.8 C++ Priority Queue](#278-c-priority-queue)：使用標準容器。
- [27.9 Top K](#279-top-k)：只保留目前最重要的 k 個元素。
- [27.10 K-way Merge](#2710-k-way-merge)：合併多個已排序序列。
- [27.11 Median Maintenance](#2711-median-maintenance)：用兩個 Heap 維護中位數。
- [27.12 Lazy Deletion](#2712-lazy-deletion)：延後處理不在頂端的刪除。
- [27.13 常見問題與判讀](#2713-常見問題與判讀)：整理常見錯誤。
- [27.14 本章檢查表](#2714-本章檢查表)：確認是否掌握核心概念。
- [27.15 本章重點](#2715-本章重點)：回顧本章核心。

### 27.1 Heap 前到底要分析什麼

假設題目如下：

資料會持續加入，系統需要隨時取出目前最小值。

很多人看到「最小值」就直接想到排序。但第一步應先整理問題：

<table>
<tr><th>分析項目</th><th>本題內容</th></tr>
<tr><td>輸入</td><td>持續加入的整數</td></tr>
<tr><td>查詢</td><td>取得目前最小值</td></tr>
<tr><td>是否會移除最小值</td><td>依題目而定</td></tr>
<tr><td>資料是否持續變化</td><td>是</td></tr>
<tr><td>是否需要完整排序</td><td>不需要</td></tr>
<tr><td>真正需要維護的資訊</td><td>目前最小值，以及移除後的下一個最小值</td></tr>
</table>

這張表會直接影響方法：

- 若只查詢一次最小值，一次 O(n) 掃描通常足夠。
- 若資料不再變化且需要完整順序，可以排序。
- 若資料持續加入，並反覆取得或移除最小值，Min Heap 通常較合適。

Heap 的用途不是讓所有元素保持排序，而是讓最高優先權元素可以快速出現在頂端。

### 27.2 Heap 不是完整排序

假設有以下 Min Heap：

```text
[1, 3, 2, 8, 5, 7]
```

它的 Tree 可以表示成：

```mermaid
graph TD
    A[1] --> B[3]
    A --> C[2]
    B --> D[8]
    B --> E[5]
    C --> F[7]
```

Min Heap 只要求：

```text
每個 Parent 小於等於自己的 Children。
```

它不要求同一層由小到大，也不要求左子樹全部小於右子樹。

例如 3 位於 2 的左邊，但 3 大於 2，仍然是合法 Min Heap。因為 3 與 2 不是 Parent 與 Child 的關係。

#### Heap 與排序陣列的差異

<table>
<tr><th>資料結構</th><th>保證的順序</th><th>最小值位置</th><th>任意元素查詢</th></tr>
<tr><td>排序 Array</td><td>所有元素依序排列</td><td>固定在一端</td><td>可使用 Binary Search</td></tr>
<tr><td>Min Heap</td><td>Parent 不大於 Children</td><td>Root</td><td>一般仍需 O(n)</td></tr>
<tr><td>Max Heap</td><td>Parent 不小於 Children</td><td>Root 是最大值</td><td>一般仍需 O(n)</td></tr>
</table>

因此不能對 Heap 直接使用 Binary Search。Heap 並沒有完整的左右單調順序。

### 27.3 Min Heap 與 Max Heap

#### Min Heap

Min Heap 的 Root 是目前最小值。

```text
Parent <= Children
```

適合：

- 反覆取出最小工作成本。
- Dijkstra 中取得目前距離最小的節點。
- K-way Merge 中取得目前最小候選。

#### Max Heap

Max Heap 的 Root 是目前最大值。

```text
Parent >= Children
```

適合：

- 反覆取出最大值。
- 維護目前最高優先權工作。
- 找第 k 大元素時保存候選。

```mermaid
flowchart LR
    A[Priority Queue] --> B[Min Heap]
    A --> C[Max Heap]
    B --> D[Root 是最小值]
    C --> E[Root 是最大值]
```

Min 或 Max 描述的是頂端優先權，不代表整個 Array 是遞增或遞減。

### 27.4 使用 Array 表示 Heap

Heap 通常是一棵 Complete Binary Tree。

Complete Binary Tree 的每一層會由左到右填滿，因此可以直接依層級放入 Array，不需要為每個節點保存左右指標。

對 0-based Index：

```text
Parent(i)      = (i - 1) / 2
LeftChild(i)   = 2 * i + 1
RightChild(i)  = 2 * i + 2
```

例如：

```text
Index: 0  1  2  3  4  5
Value: 1  3  2  8  5  7
```

<table>
<tr><th>Index</th><th>Value</th><th>Parent</th><th>Left Child</th><th>Right Child</th></tr>
<tr><td>0</td><td>1</td><td>無</td><td>Index 1，值 3</td><td>Index 2，值 2</td></tr>
<tr><td>1</td><td>3</td><td>Index 0，值 1</td><td>Index 3，值 8</td><td>Index 4，值 5</td></tr>
<tr><td>2</td><td>2</td><td>Index 0，值 1</td><td>Index 5，值 7</td><td>無</td></tr>
</table>

計算 Child Index 後，仍要確認它小於 Heap 大小，否則該 Child 不存在。

### 27.5 Insert 與 Heapify Up

插入新元素時，先把元素放在 Array 尾端。這樣可以保持 Complete Binary Tree 的形狀。

但新元素可能破壞 Heap Property，因此要向上比較 Parent，必要時交換。這個過程稱為 Heapify Up，也常稱為 Sift Up。

#### 手動追蹤

原本 Min Heap：

```text
[2, 5, 4, 9, 7]
```

插入 1：

```text
[2, 5, 4, 9, 7, 1]
```

1 的 Parent 是 4。因為 1 < 4，交換：

```text
[2, 5, 1, 9, 7, 4]
```

接著 1 的 Parent 是 2。因為 1 < 2，再交換：

```text
[1, 5, 2, 9, 7, 4]
```

```mermaid
flowchart TD
    A[將新元素放到 Array 尾端] --> B{"新元素是否小於 Parent"}
    B -->|是| C[與 Parent 交換]
    C --> B
    B -->|否| D[Heapify Up 完成]
```

#### C++ Min Heap Insert

```cpp
void push(std::vector<int>& heap, int value)
{
    heap.push_back(value);
    int index = static_cast<int>(heap.size()) - 1;

    while (index > 0)
    {
        int parent = (index - 1) / 2;

        if (heap[parent] <= heap[index])
        {
            break;
        }

        std::swap(heap[parent], heap[index]);
        index = parent;
    }
}
```

Heap 高度是 O(log n)，所以插入時間為 O(log n)。

### 27.6 Pop 與 Heapify Down

移除 Min Heap 頂端時，不能直接刪除 Root 後留下空洞，否則會破壞 Complete Binary Tree 的形狀。

常見流程：

1. 保存 Root 值。
2. 將最後一個元素移到 Root。
3. 刪除 Array 最後一格。
4. 從 Root 向下修復 Heap Property。

#### 手動追蹤

原本：

```text
[1, 3, 2, 8, 5, 7]
```

移除 1，將最後的 7 移到 Root：

```text
[7, 3, 2, 8, 5]
```

7 的 Children 是 3 與 2。Min Heap 要和較小的 Child 交換，因此選 2：

```text
[2, 3, 7, 8, 5]
```

Heap Property 已恢復。

```mermaid
flowchart TD
    A[最後元素移到 Root] --> B[比較左右 Children]
    B --> C[找出較小的 Child]
    C --> D{"Parent 是否大於較小 Child"}
    D -->|是| E[交換並繼續向下]
    E --> B
    D -->|否| F[Heapify Down 完成]
```

#### C++ Min Heap Pop

```cpp
int popMin(std::vector<int>& heap)
{
    int result = heap.front();
    heap.front() = heap.back();
    heap.pop_back();

    int index = 0;
    int n = static_cast<int>(heap.size());

    while (true)
    {
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        int smallest = index;

        if (left < n && heap[left] < heap[smallest])
        {
            smallest = left;
        }

        if (right < n && heap[right] < heap[smallest])
        {
            smallest = right;
        }

        if (smallest == index)
        {
            break;
        }

        std::swap(heap[index], heap[smallest]);
        index = smallest;
    }

    return result;
}
```

呼叫前要確認 Heap 不為空。移除時間為 O(log n)。

### 27.7 Build Heap

若已有一個未排序 Array，可以逐一插入 Heap。n 次 O(log n) 插入得到 O(n log n)。

另一種方式是從最後一個非 Leaf 節點開始，逐一執行 Heapify Down。

0-based Index 中，最後一個非 Leaf 節點是：

```text
n / 2 - 1
```

```cpp
void heapifyDown(std::vector<int>& heap, int index)
{
    int n = static_cast<int>(heap.size());

    while (true)
    {
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        int smallest = index;

        if (left < n && heap[left] < heap[smallest])
        {
            smallest = left;
        }

        if (right < n && heap[right] < heap[smallest])
        {
            smallest = right;
        }

        if (smallest == index)
        {
            break;
        }

        std::swap(heap[index], heap[smallest]);
        index = smallest;
    }
}

void buildMinHeap(std::vector<int>& heap)
{
    for (int i = static_cast<int>(heap.size()) / 2 - 1; i >= 0; --i)
    {
        heapifyDown(heap, i);
    }
}
```

Bottom-up Build Heap 的時間複雜度是 O(n)，不是 O(n log n)。原因是大部分節點位於底層，需要向下移動的距離很短。

### 27.8 C++ Priority Queue

C++ `std::priority_queue` 預設是 Max Heap。

#### Max Heap

```cpp
#include <queue>
#include <vector>

std::priority_queue<int> maxHeap;

maxHeap.push(3);
maxHeap.push(1);
maxHeap.push(5);

int largest = maxHeap.top(); // 5
maxHeap.pop();
```

#### Min Heap

```cpp
#include <functional>
#include <queue>
#include <vector>

std::priority_queue<
    int,
    std::vector<int>,
    std::greater<int>> minHeap;
```

常用函式：

<table>
<tr><th>函式</th><th>用途</th><th>時間</th></tr>
<tr><td>`top()`</td><td>查看最高優先權元素</td><td>O(1)</td></tr>
<tr><td>`push()`</td><td>加入元素</td><td>O(log n)</td></tr>
<tr><td>`pop()`</td><td>移除頂端元素</td><td>O(log n)</td></tr>
<tr><td>`empty()`</td><td>判斷是否為空</td><td>O(1)</td></tr>
<tr><td>`size()`</td><td>取得元素數量</td><td>O(1)</td></tr>
</table>

`pop()` 不會回傳被移除的值。若需要該值，要先讀取 `top()`。

### 27.9 Top K

Top K 題目通常不需要完整排序，只需要最大的 k 個或最小的 k 個元素。

#### 找最大的 k 個元素

可以維護大小最多為 k 的 Min Heap。

- Heap 保存目前最大的 k 個候選。
- Heap 頂端是這 k 個候選中最小的。
- 遇到更大的值時，移除頂端，再加入新值。

```mermaid
flowchart TD
    A[讀取下一個 value] --> B{"Heap 大小是否小於 k"}
    B -->|是| C[直接加入 Min Heap]
    B -->|否| D{"value 是否大於 Heap Top"}
    D -->|是| E[移除 Top 並加入 value]
    D -->|否| F[忽略 value]
```

#### C++ 找第 k 大

```cpp
#include <functional>
#include <queue>
#include <vector>

int findKthLargest(const std::vector<int>& nums, int k)
{
    std::priority_queue<
        int,
        std::vector<int>,
        std::greater<int>> minHeap;

    for (int value : nums)
    {
        minHeap.push(value);

        if (static_cast<int>(minHeap.size()) > k)
        {
            minHeap.pop();
        }
    }

    return minHeap.top();
}
```

時間複雜度為 O(n log k)，額外空間為 O(k)。

容易混淆的是：找最大的 k 個，常用大小為 k 的 Min Heap，因為要快速淘汰候選中最小的值。

### 27.10 K-way Merge

若有 k 個已排序序列，要合併成一個已排序序列，每次只需比較各序列目前最前面的候選。

Min Heap 可以保存：

- 值。
- 來自哪個序列。
- 該序列中的位置。

每次取出最小候選，再把同一序列的下一個元素放入 Heap。

```mermaid
flowchart TD
    A[各序列第一個元素加入 Min Heap] --> B[取出最小候選]
    B --> C[寫入結果]
    C --> D{"來源序列是否還有下一個元素"}
    D -->|是| E[將下一個元素加入 Heap]
    E --> B
    D -->|否| F{"Heap 是否為空"}
    F -->|否| B
    F -->|是| G[合併完成]
```

若總元素數是 N，Heap 大小最多為 k，時間複雜度為 O(N log k)。

### 27.11 Median Maintenance

若數字持續加入，並要隨時取得中位數，可以使用兩個 Heap：

- Max Heap 保存較小的一半。
- Min Heap 保存較大的一半。

維持兩個條件：

1. Max Heap 所有值不大於 Min Heap 所有值。
2. 兩邊大小差不超過 1。

```mermaid
flowchart LR
    A[較小的一半 Max Heap] --> B[中位數]
    C[較大的一半 Min Heap] --> B
```

若兩邊大小相同，中位數通常是兩個 Top 的平均。若一邊多一個元素，中位數是較大那一邊的 Top。

插入與重新平衡通常為 O(log n)，讀取中位數為 O(1)。計算平均時要小心整數溢位與整數除法。

### 27.12 Lazy Deletion

Priority Queue 可以快速移除 Top，但不擅長直接移除中間任意元素。

Lazy Deletion 的想法是：

- 先記錄某個值或某筆資料已經失效。
- 不立即從 Heap 中間移除。
- 等失效資料走到 Top 時，再真正 `pop()`。

```cpp
while (!heap.empty() && isInvalid(heap.top()))
{
    heap.pop();
}
```

這種方法常用於 Sliding Window Median 或需要延後刪除的 Priority Queue 問題。

要注意，Heap 內可能暫時保留失效元素，因此：

- 邏輯大小可能不同於容器實際大小。
- 讀取 Top 前要先清理失效項目。
- 若相同值可出現多次，通常要記錄待刪除次數，而不是單一 Boolean。

### 27.13 常見問題與判讀

<table>
<tr><th>現象</th><th>可能原因</th><th>第一輪檢查</th></tr>
<tr><td>以為 Heap Array 已完整排序</td><td>混淆 Heap Property 與排序順序</td><td>只檢查 Parent 與 Children</td></tr>
<tr><td>Min Heap Pop 後仍不合法</td><td>只和左 Child 比較</td><td>應先找左右 Children 中較小者</td></tr>
<tr><td>Child Index 越界</td><td>計算後未檢查是否小於 n</td><td>先檢查 `left < n`、`right < n`</td></tr>
<tr><td>`priority_queue` 頂端方向錯誤</td><td>忘記預設為 Max Heap</td><td>Min Heap 使用 `std::greater`</td></tr>
<tr><td>呼叫 `top()` 崩潰</td><td>Heap 為空</td><td>先檢查 `empty()`</td></tr>
<tr><td>Top K 使用錯誤 Heap</td><td>沒有先想要淘汰哪個候選</td><td>找最大 k 個時通常淘汰其中最小者</td></tr>
<tr><td>Lazy Deletion 讀到失效資料</td><td>讀取 Top 前未清理</td><td>使用 while 持續移除失效 Top</td></tr>
<tr><td>中位數平均溢位</td><td>先用 int 相加</td><td>轉成較大型別後再加總</td></tr>
</table>

### 27.14 本章檢查表

- 我能說明 Heap 不等於完整排序。
- 我能分辨 Min Heap 與 Max Heap 的 Root 意義。
- 我能使用 Array Index 找出 Parent 與 Children。
- 我能手動追蹤 Insert 與 Heapify Up。
- 我能手動追蹤 Pop 與 Heapify Down。
- 我知道 Heapify Down 要與較適合的 Child 交換。
- 我知道 Bottom-up Build Heap 為 O(n)。
- 我知道 C++ `priority_queue` 預設為 Max Heap。
- 我能建立 C++ Min Heap。
- 我能說明找最大 k 個時為什麼常用大小為 k 的 Min Heap。
- 我能說明 K-way Merge 的 Heap 中要保存哪些資訊。
- 我能說明兩個 Heap 如何維護中位數。
- 我知道 Lazy Deletion 需要在讀取 Top 前清理失效元素。

### 27.15 本章重點

- Heap 適合反覆取得或移除目前最高優先權元素。
- Min Heap 的 Root 是最小值，Max Heap 的 Root 是最大值。
- Heap 只維護 Parent 與 Children 的順序，不是完整排序。
- Complete Binary Tree 可以緊密地放入 Array。
- 插入時先放到尾端，再用 Heapify Up 修復。
- 移除頂端時用最後元素補到 Root，再用 Heapify Down 修復。
- Heap 的 `top` 為 O(1)，`push` 與 `pop` 通常為 O(log n)。
- Bottom-up Build Heap 的時間複雜度是 O(n)。
- C++ `std::priority_queue` 預設為 Max Heap。
- Top K 常用大小為 k 的 Heap，將成本控制在 O(n log k)。
- K-way Merge 用 Heap 保存每個序列目前的最小候選。
- Median Maintenance 可用 Max Heap 與 Min Heap 分別保存資料兩半。
- Lazy Deletion 適合無法直接刪除 Heap 中間元素的情況。
