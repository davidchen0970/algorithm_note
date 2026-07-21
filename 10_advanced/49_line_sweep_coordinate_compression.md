# 第 49 章　Line Sweep 與 Coordinate Compression

在處理區間（Interval）或幾何題目時，我們常會遇到這種看似棘手的困境：

* "有 100 萬個人進出會議室，哪個時間點裡面的人最多？" 
* "給定 1,000 個區間，它們覆蓋的總長度是多少？" 
* "座標範圍大到 $10^9$，陣列根本開不下，該怎麼辦？" 
如果用直覺的暴力法，直接拿一個超大陣列去模擬每一個座標點，不只時間會爆掉，記憶體也會瞬間崩潰。

為了解決這個問題，這一步的核心思維就是 "動態追蹤變化，並去掉不必要的空白" ：

Line Sweep（掃描線）： 想像有一條垂直的掃描線，從左到右滑過整個空間（或時間軸）。我們不需要關心 "沒有事情發生" 的空白區域，只需要在 "起點（事件開始）" 與 "終點（事件結束）" 發生時，更新目前的狀態。
Coordinate Compression（座標壓縮）： 當座標值極大但數值數量很少時，我們只保留數字之間的相對大小與順序，把疏散的大座標映射到緊密的連續小數字上（例如將 $10^9$ 壓縮成小於 $N$ 的索引值）。
學會這兩招後，原本需要高時間與空間複雜度的題目，通常都能優化到 $O(N \log N)$ 甚至更低的級別。這一章我們會由淺入深，一步步把這兩個經典的技巧拆解給你看！


## 適用範圍

本章假設讀者目前會：

- `for` 與 `while` 迴圈。
- `std::vector`。
- `std::sort`。
- `std::pair`。
- Half-open Interval `[left, right)`。

本章不假設讀者一開始就會 Fenwick Tree、Segment Tree 或 Rectangle Union Area。這些內容會放在後半部，並標示為延伸內容。

第一次閱讀的主要目標只有三個：

1. 把每個 Interval 拆成開始與結束 Event。
2. 排序 Event，從左向右更新目前狀態。
3. 理解 Coordinate Compression 只保留順序，不保留實際距離。

只要能獨立寫出 "最大重疊數量" 與 "Interval 聯集長度" ，第一輪就已經完成。

## 閱讀方式

### 第一輪必讀

- 49.1 先從一個時間區間問題開始。
- 49.2 Event 是什麼。
- 49.3 最大重疊數量。
- 49.4 聯集長度。
- 49.5 同座標 Event 與 Half-open Interval。

### 第二輪必讀

- 49.6 Difference Event。
- 49.7 Coordinate Compression。
- 49.8 點壓縮與區段壓縮。

### 第三輪延伸

- 49.9 搭配 Fenwick Tree。
- 49.10 搭配 Segment Tree。
- 49.11 Rectangle Union Area。

如果尚未學過 Fenwick Tree 或 Segment Tree，可以先跳過第三輪。這不影響對 Line Sweep 與 Coordinate Compression 基礎的理解。

## 快速導覽

- [49.1 先從一個時間區間問題開始](#491-先從一個時間區間問題開始)
- [49.2 Event 是什麼](#492-event-是什麼)
- [49.3 完整案例：最大重疊數量](#493-完整案例最大重疊數量)
- [49.4 完整案例：Interval 聯集長度](#494-完整案例interval-聯集長度)
- [49.5 同座標 Event 與 Half-open Interval](#495-同座標-event-與-half-open-interval)
- [49.6 Difference Event](#496-difference-event)
- [49.7 Coordinate Compression](#497-coordinate-compression)
- [49.8 點壓縮與區段壓縮](#498-點壓縮與區段壓縮)
- [49.9 延伸：搭配 Fenwick Tree](#499-延伸搭配-fenwick-tree)
- [49.10 延伸：搭配 Segment Tree](#4910-延伸搭配-segment-tree)
- [49.11 延伸：Rectangle Union Area](#4911-延伸rectangle-union-area)
- [49.12 固定分析流程](#4912-固定分析流程)
- [49.13 常見問題與判讀](#4913-常見問題與判讀)
- [49.14 本章檢查表](#4914-本章檢查表)
- [49.15 本章重點](#4915-本章重點)


## 49.1 先從一個時間區間問題開始

假設有三場會議：

```text
A：[1, 4)
B：[2, 5)
C：[4, 6)
```

`[1, 4)` 表示從時間 1 開始，到時間 4 結束，而且不包含時間 4。

我們想知道：

> 最多有幾場會議同時進行？

### 最直覺的觀察

會議數量只會在下列時間改變：

```text
1, 2, 4, 5, 6
```

在時間 2 到 4 之間，不會突然有新會議開始或結束，所以進行中的會議數量保持不變。

因此不必處理每一個可能時間，只需處理 "狀態發生變化的位置" 。

這些位置稱為 Event。

### 從左向右走一次

```text
時間 1：A 開始，active = 1
時間 2：B 開始，active = 2
時間 4：A 結束，C 開始，active 仍是 2
時間 5：B 結束，active = 1
時間 6：C 結束，active = 0
```

最大值是 2。

這就是最基本的 Line Sweep：

```text
建立 Event
→ 依座標排序
→ 從左向右處理
→ 維護目前狀態
```

### Line Sweep 不一定是幾何線

Sweep Axis 可以是：

- x 座標。
- y 座標。
- 時間。
- 排序後的位置。
- 其他具有先後順序的 Key。

核心不是畫一條線，而是將問題轉成 "依某個值排序後，逐步更新狀態" 。


## 49.2 Event 是什麼

對 Half-open Interval：

```text
[left, right)
```

可以建立兩個 Event：

```text
(left,  +1)  開始
(right, -1)  結束
```

例如：

```text
[2, 5)
```

變成：

```text
(2, +1)
(5, -1)
```

### 最簡單的 Event 結構

```cpp
struct Event
{
    long long coordinate;
    int delta;
};
```

- `coordinate`：事件發生位置。
- `delta`：目前 Active Count 要增加或減少多少。

也可以先使用：

```cpp
std::pair<long long, int>
```

其中：

```text
first  = coordinate
second = delta
```

### 為什麼一定要排序

輸入 Interval 可能沒有依時間排列。Line Sweep 必須按照 Event 發生順序處理，所以先排序座標。

```cpp
std::sort(events.begin(), events.end());
```

對 `pair` 而言，會先比較 `first`，相同時再比較 `second`。

不過，同座標 Start 與 End 的先後可能影響答案。與其依賴 `+1`、`-1` 的排序順序，本章的基礎案例會將同座標 Delta 全部加總，再一次更新。


## 49.3 完整案例：最大重疊數量

### 問題

給定多個 Half-open Interval `[left, right)`，求任意位置同時被多少個 Interval 覆蓋的最大值。

零長度或反向 Interval：

```text
left >= right
```

在本案例中忽略。

### 第一步：建立 Events

```cpp
std::vector<std::pair<long long, int>> events;

for (const auto& [left, right] : intervals)
{
    if (left >= right)
    {
        continue;
    }

    events.push_back({left, +1});
    events.push_back({right, -1});
}
```

### 第二步：排序

```cpp
std::sort(events.begin(), events.end());
```

### 第三步：同座標一起處理

```cpp
int active = 0;
int answer = 0;

for (std::size_t i = 0; i < events.size(); )
{
    const long long coordinate = events[i].first;
    int delta = 0;

    while (i < events.size() &&
           events[i].first == coordinate)
    {
        delta += events[i].second;
        ++i;
    }

    active += delta;
    answer = std::max(answer, active);
}
```

### 完整程式

```cpp
#include <algorithm>
#include <utility>
#include <vector>

int maximumOverlap(
    const std::vector<std::pair<long long, long long>>& intervals)
{
    std::vector<std::pair<long long, int>> events;

    for (const auto& [left, right] : intervals)
    {
        if (left >= right)
        {
            continue;
        }

        events.push_back({left, +1});
        events.push_back({right, -1});
    }

    std::sort(events.begin(), events.end());

    int active = 0;
    int answer = 0;
    std::size_t i = 0;

    while (i < events.size())
    {
        const long long coordinate = events[i].first;
        int delta = 0;

        while (i < events.size() &&
               events[i].first == coordinate)
        {
            delta += events[i].second;
            ++i;
        }

        active += delta;
        answer = std::max(answer, active);
    }

    return answer;
}
```

### 手動追蹤

輸入：

```text
[1, 4)
[2, 5)
[4, 6)
```

Events：

```text
(1, +1)
(2, +1)
(4, -1)
(4, +1)
(5, -1)
(6, -1)
```

依座標分組後：

```text
座標 1：delta = +1，active = 1
座標 2：delta = +1，active = 2
座標 4：delta =  0，active = 2
座標 5：delta = -1，active = 1
座標 6：delta = -1，active = 0
```

答案是 2。

### 為什麼座標 4 的 Delta 是 0

在 `[left, right)` 語意下：

- A 在 4 結束。
- C 在 4 開始。

它們沒有重疊正長度，但進行中的總數仍由 2 變成 2。因此合併後 Delta 為 0。

### 複雜度

若有 `n` 個 Interval：

- 建立 `2n` 個 Event：O(n)。
- 排序：O(n log n)。
- Sweep：O(n)。
- 總時間：O(n log n)。
- 額外空間：O(n)。


## 49.4 完整案例：Interval 聯集長度

### 問題

計算所有 Interval 合併後，總共覆蓋多長。

例如：

```text
[1, 4)
[2, 5)
```

雖然兩個區間長度都是 3，但重疊部分不能重複計算。聯集是：

```text
[1, 5)
```

答案是 4。

### 最大重疊與聯集長度的差異

最大重疊只在 Event 位置更新 `active`。

聯集長度還要計算相鄰 Event 之間的距離：

```text
currentCoordinate - previousCoordinate
```

如果上一段的 `active > 0`，該段就被至少一個 Interval 覆蓋。

### 最重要的更新順序

到達 `coordinate` 時，區段：

```text
[previous, coordinate)
```

仍然由 "目前 Event 尚未套用前" 的舊 `active` 決定。

因此順序一定是：

```text
1. 使用舊 active 計算上一段
2. 合併目前座標的 delta
3. 更新 active
4. previous = coordinate
```

### 完整程式

```cpp
#include <algorithm>
#include <utility>
#include <vector>

long long intervalUnionLength(
    const std::vector<std::pair<long long, long long>>& intervals)
{
    std::vector<std::pair<long long, int>> events;

    for (const auto& [left, right] : intervals)
    {
        if (left >= right)
        {
            continue;
        }

        events.push_back({left, +1});
        events.push_back({right, -1});
    }

    if (events.empty())
    {
        return 0;
    }

    std::sort(events.begin(), events.end());

    long long answer = 0;
    long long previous = events[0].first;
    int active = 0;
    std::size_t i = 0;

    while (i < events.size())
    {
        const long long coordinate = events[i].first;

        if (active > 0)
        {
            answer += coordinate - previous;
        }

        int delta = 0;

        while (i < events.size() &&
               events[i].first == coordinate)
        {
            delta += events[i].second;
            ++i;
        }

        active += delta;
        previous = coordinate;
    }

    return answer;
}
```

### 手動追蹤

輸入：

```text
[1, 4)
[2, 5)
```

```text
到座標 1：
上一段長度 0
套用 +1，active = 1
previous = 1

到座標 2：
active > 0，加入 2 - 1 = 1
套用 +1，active = 2
previous = 2

到座標 4：
active > 0，加入 4 - 2 = 2
套用 -1，active = 1
previous = 4

到座標 5：
active > 0，加入 5 - 4 = 1
套用 -1，active = 0
```

總長度：

```text
1 + 2 + 1 = 4
```

### 常見錯誤

錯誤順序：

```text
先更新 active
再計算 [previous, coordinate)
```

這會把目前座標才發生的變化，錯誤套用到前一段。


## 49.5 同座標 Event 與 Half-open Interval

### 為什麼本章使用 `[left, right)`

對相接區間：

```text
[1, 3)
[3, 5)
```

第一個在 3 結束，第二個在 3 開始，兩者沒有共同覆蓋正長度。

這與 C++ 常見的 `[begin, end)` 區間模型一致，也讓長度直接等於：

```text
right - left
```

### 同座標 Event 為什麼要 Group

如果同一座標同時有多個 Start 和 End，單純依 Event Type 排序，答案可能依題目語意而改變。

對只需要區段右側狀態的 Half-open Interval Count，可以把所有 Delta 先加總：

```cpp
int delta = 0;

while (i < events.size() &&
       events[i].first == coordinate)
{
    delta += events[i].second;
    ++i;
}

active += delta;
```

這能避免依賴 Start 與 End 的任意排序。

### Closed Interval 不能直接照抄

若題目使用：

```text
[left, right]
```

端點 `right` 仍被包含。此時相同座標的 Start、End、Query 先後順序，必須依題目對 "端點是否同時存在" 的定義重新推導。

不要只背：

```text
Start 一定先於 End
```

或：

```text
End 一定先於 Start
```

真正的順序取決於 Interval 與 Query 的語意。


## 49.6 Difference Event

Line Sweep 的 `+1`、`-1` 與 Difference Array 使用相同想法。

對：

```text
[left, right)
```

做：

```text
left 位置 +1
right 位置 -1
```

之後依序累加，就得到每個位置的 Active Count。

### 何時使用 Difference Array

適合：

- 座標是 `0..n-1`。
- `n` 不大。
- 座標密集。
- 最後需要知道每個位置的數值。

### 何時使用排序 Event

適合：

- 座標可能到 `10^9` 或更大。
- 只有少數端點真的出現。
- 只需處理狀態變化位置。
- 所有 Event 可以離線排序。

### 直覺比較

```text
Difference Array：為整個值域準備位置
Event Sorting：只保存真的發生變化的位置
```


## 49.7 Coordinate Compression

### 它要解決什麼問題

假設座標是：

```text
100
5000
1000000
```

若想用 Array 依座標存資料，直接開到 1,000,001 格可能很浪費。

但實際上只有三個座標重要，所以可以映射成：

```text
100      -> 0
5000     -> 1
1000000  -> 2
```

這就是 Coordinate Compression。

### 壓縮保留什麼

保留：

- 相等關係。
- 小於與大於的順序。
- 排名。

不保留：

- 實際距離。

例如：

```text
5000 與 1000000 的壓縮 Index 只差 1
```

但實際距離是：

```text
1000000 - 5000 = 995000
```

### 三個步驟

#### 第一步：收集

```cpp
std::vector<long long> coordinates;
```

加入所有後續需要映射的座標。

#### 第二步：排序與去重

```cpp
std::sort(coordinates.begin(), coordinates.end());

coordinates.erase(
    std::unique(coordinates.begin(), coordinates.end()),
    coordinates.end());
```

#### 第三步：找 Index

```cpp
const int index = static_cast<int>(
    std::lower_bound(
        coordinates.begin(),
        coordinates.end(),
        value)
    - coordinates.begin());
```

### 完整輔助函式

```cpp
#include <algorithm>
#include <vector>

std::vector<long long> compressCoordinates(
    std::vector<long long> coordinates)
{
    std::sort(coordinates.begin(), coordinates.end());

    coordinates.erase(
        std::unique(
            coordinates.begin(),
            coordinates.end()),
        coordinates.end());

    return coordinates;
}

int compressedIndex(
    const std::vector<long long>& coordinates,
    long long value)
{
    return static_cast<int>(
        std::lower_bound(
            coordinates.begin(),
            coordinates.end(),
            value)
        - coordinates.begin());
}
```

### 手動範例

原始資料：

```text
5000, 100, 5000, 1000000
```

排序：

```text
100, 5000, 5000, 1000000
```

去重：

```text
100, 5000, 1000000
```

映射：

```text
100      -> 0
5000     -> 1
1000000  -> 2
```

### Query 值不一定存在

如果要問：

```text
有多少已收集座標 <= q
```

可以使用：

```cpp
const int count = static_cast<int>(
    std::upper_bound(
        coordinates.begin(),
        coordinates.end(),
        q)
    - coordinates.begin());
```

`count` 本身就是不大於 `q` 的座標數量。

若需要最後一個 `<= q` 的 0-based Index：

```cpp
const int index = count - 1;
```

當 `count == 0` 時，`index == -1`，代表沒有符合座標。


## 49.8 點壓縮與區段壓縮

這是 Coordinate Compression 最容易混淆的地方。

### 點壓縮

只關心座標點的順序或排名：

- 某個值是第幾小。
- Point Update。
- Prefix Count。
- Inversion Count。

此時壓縮 Index 代表一個點。

### 區段壓縮

若要計算長度或面積，真正重要的是相鄰座標形成的區段。

若唯一座標為：

```text
coordinates = [100, 5000, 1000000]
```

區段是：

```text
Index 0 代表 [100, 5000)
長度 = 5000 - 100

Index 1 代表 [5000, 1000000)
長度 = 1000000 - 5000
```

三個座標只形成兩個相鄰區段。

一般而言：

```text
m 個唯一座標
→ m - 1 個相鄰區段
```

### 不能使用 Index 差計算長度

錯誤：

```text
Index 2 - Index 1 = 1
所以長度是 1
```

正確：

```text
coordinates[2] - coordinates[1]
```

Compression 保留排序，不保留實際距離。


## 49.9 延伸：搭配 Fenwick Tree

> 若尚未學過 Fenwick Tree，可以先跳過本節。

典型二維問題會：

1. 依 x 排序並 Sweep。
2. 將 y 做 Coordinate Compression。
3. 用 Fenwick Tree 維護已經遇到的 y 次數。

例如，計算每個點左下方有多少點：

```text
依 x 由小到大處理
查詢 y 以下已有多少點
再把目前 y 加入 Fenwick Tree
```

### 相同 x 的點

若題目要求 "嚴格在左側" ，相同 x 的點不能互相計入。

應依 x Group：

```text
對同 x 的所有點先 Query
完成後再一起 Update
```

如果邊查邊更新，同一組中較早處理的點會錯誤影響後面的點。

這和同座標 Event Grouping 是同一類問題：

> 相同 Sweep 座標的資料，是否應互相看見？


## 49.10 延伸：搭配 Segment Tree

> 若尚未學過 Segment Tree 與 Lazy Propagation，可以先跳過。

Sweep 一個維度時，另一個維度可能需要維護：

- Range Add。
- Maximum Count。
- Covered Length。

例如 Rectangle Union Area：

- x 方向使用 Line Sweep。
- y 方向壓縮後交給 Segment Tree。

### Covered Length 的節點概念

若某個 y 區段的 `coverCount > 0`：

```text
整段都被覆蓋
coveredLength = 原始右座標 - 原始左座標
```

若 `coverCount == 0`：

```text
coveredLength = 左子節點 + 右子節點
```

重點是使用原始座標差，不是葉節點數量。


## 49.11 延伸：Rectangle Union Area

> 這是本章綜合題。第一次閱讀只需理解流程，不要求立即寫出完整 Segment Tree。

每個 Rectangle：

```text
[x1, x2) × [y1, y2)
```

建立兩個 x Events：

```text
(x1, y1, y2, +1)
(x2, y1, y2, -1)
```

Sweep x 時，Segment Tree 維護目前 y 方向的聯集長度：

```text
coveredYLength
```

相鄰 x Event 間的面積是：

```text
coveredYLength × (currentX - previousX)
```

### 固定流程

1. 將 Rectangle 轉成 x Events。
2. 收集所有 `y1`、`y2`。
3. 對 y 做區段壓縮。
4. 依 x 排序 Event。
5. 先用舊 `coveredYLength` 計算上一條 x 區段面積。
6. 再套用目前 x 的全部 y Range Updates。
7. 更新 `previousX`。

### 與 Interval Union Length 的關係

一維聯集長度：

```text
若 active > 0
加入 current - previous
```

二維 Rectangle Area：

```text
加入 coveredYLength × (currentX - previousX)
```

核心更新順序完全相同：都先用舊狀態計算上一段，再套用目前 Events。

### 常見錯誤

- 使用 y Index 差而非原始 y 座標差。
- `m` 個 y 座標建立 `m` 個區段，正確通常是 `m - 1`。
- 先更新目前 x Event，再計算上一段面積。
- 沒有忽略零寬或零高 Rectangle。
- 面積乘法使用太小的整數型別。


## 49.12 固定分析流程

### 第一步：選 Sweep Axis

問：

```text
按照哪個值排序後，狀態只會向前變化？
```

可能是 x、y、時間或其他 Key。

### 第二步：定義 Interval 語意

- `[left, right)`？
- `[left, right]`？
- 端點相接是否算重疊？
- 零長度 Interval 是否有效？

### 第三步：建立 Event

每個 Event 必須保留足夠資訊來更新狀態：

- 座標。
- `delta`。
- Event Type。
- 另一維範圍。
- Query ID。

基礎 Count 問題只需要 `(coordinate, delta)`。

### 第四步：決定同座標語意

- 同座標 Delta 是否可以直接加總？
- Query 應在 Update 前還是後？
- 相同 x 的點是否互相計入？

若可以，優先 Group 後一次處理，通常比依賴細微 Tie-breaking 更清楚。

### 第五步：定義 State

例如：

- `active`。
- `coveredLength`。
- Active Set。
- Fenwick Tree。
- Segment Tree。

### 第六步：確認上一段由哪個 State 決定

對長度與面積問題：

```text
[previous, current)
```

通常由處理目前 Event 之前的舊 State 決定。

### 第七步：決定是否需要 Compression

- 座標小而密集：Array 或 Difference Array。
- 座標大而稀疏：Event Sorting 或 Compression。
- 只需排名或 Count：點壓縮。
- 需要長度或面積：區段壓縮並保留原始差值。

### 第八步：建立最小測試

至少測試：

- 空集合。
- 單一 Interval。
- 零長度 Interval。
- 完全不重疊。
- 完全重疊。
- 只在端點相接。
- 多個 Event 同座標。
- 負座標。
- 極大座標。


## 49.13 常見問題與判讀

### 最大重疊在相接端點多算一個

可能混用了 Closed 與 Half-open Interval，或依賴錯誤的 Start / End 排序。

先確認：

```text
[1, 3) 與 [3, 5)
```

在 3 是否應算同時覆蓋。

### Union Length 少算或多算一段

檢查是否依照：

```text
先用舊 active 計算 [previous, current)
再更新 active
```

### 同座標答案受排序方式影響

若事件可以合併，先將同座標 Delta 加總。若不能合併，必須明確寫出 Query、Start、End 的語意順序。

### Compression 後長度變成 1

可能將壓縮 Index 差當成實際距離。

應使用：

```cpp
coordinates[right] - coordinates[left]
```

### 有 m 個座標卻建立錯誤區段數

相鄰區段通常只有：

```text
m - 1
```

### Fenwick Tree 同 x 的點互相計入

如果要求嚴格較小 x，需先 Query 整組，再 Update 整組。

### Rectangle Area Overflow

檢查：

```text
coveredYLength × xDifference
```

乘法前就必須使用足夠寬的型別。

### Query 座標不在壓縮陣列中

- 找精確 Index：`lower_bound`，但要確認真的存在。
- 找 `<= q` 的數量：`upper_bound`。


## 49.14 本章檢查表

### 第一輪：Line Sweep 基礎

- 我知道 Event 是狀態改變的位置。
- 我能將 `[left, right)` 轉成 `(left, +1)` 與 `(right, -1)`。
- 我知道為什麼要先排序 Event。
- 我能將同座標 Delta 分組加總。
- 我能寫出 Maximum Overlap。
- 我知道 Union Length 必須先算上一段，再更新目前 Event。

### 第二輪：Compression

- 我知道 Compression 保留順序與相等關係。
- 我知道 Compression 不保留實際距離。
- 我能排序、去重並使用 `lower_bound` 找 Index。
- 我知道 `upper_bound` 可以計算 `<= q` 的座標數量。
- 我能區分點壓縮與區段壓縮。
- 我知道 `m` 個唯一座標通常形成 `m - 1` 個相鄰區段。

### 第三輪：延伸結構

- 我知道相同 x 的 Query 與 Update 可能需要分組。
- 我知道 Fenwick Tree 常維護點次數或 Prefix Count。
- 我知道 Segment Tree 可維護 Range Cover 與 Covered Length。
- 我知道 Rectangle Area 是 y 聯集長度乘上 x 差。
- 我知道長度與面積必須使用原始座標差。


## 49.15 本章重點

1. Line Sweep 只處理狀態發生變化的 Event，不必逐一走過巨大值域。
2. 一個 Half-open Interval `[left, right)` 可轉成開始 `+1` 與結束 `-1` Event。
3. 同座標 Event 優先考慮 Group，以明確表達該座標處理後的狀態。
4. Maximum Overlap 只維護 `active` 與最大值。
5. Union Length 還要計算相鄰 Event 間距離，且上一段由舊 State 決定。
6. Difference Array 與 Event Sweep 都使用端點差值，差別在是否為整個值域準備空間。
7. Coordinate Compression 保留順序與相等，不保留距離。
8. 點壓縮適合排名與 Count；區段壓縮適合 Length 與 Area。
9. 相同 Sweep 座標的資料是否互相影響，是 Tie-breaking 與 Grouping 的核心問題。
10. Fenwick Tree、Segment Tree 與 Rectangle Area 都是基礎 Sweep 的延伸，不需要第一次閱讀就全部掌握。
