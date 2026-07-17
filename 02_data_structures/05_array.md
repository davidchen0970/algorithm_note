## 第 5 章　Array 與 Dynamic Array

### 適用範圍

本章介紹 Array 與 Dynamic Array 的核心模型，以及它們如何影響 Index 存取、走訪、插入、刪除、記憶體配置與演算法設計。

Array 是多數基礎演算法的共同載體。Two Pointers、Sliding Window、Prefix Sum、Binary Search、Sorting 與 Dynamic Programming，經常都建立在 Array 的索引與區間模型上。因此，學習 Array 不只是記住語法，而是理解以下問題：

- 元素在記憶體中如何排列。
- 為什麼按 Index 存取通常是 O(1)。
- 為什麼中間插入或刪除通常是 O(n)。
- `size` 與 `capacity` 分別代表什麼。
- `reserve` 與 `resize` 為何不能互換。
- `std::vector` 何時可能重新配置記憶體。
- Pointer、Reference 與 Iterator 何時失效。
- 如何使用 Read/Write Pointer 原地整理資料。
- 二維資料是否真的位於一整塊連續記憶體。
- 如何避免越界、Unsigned Underflow 與錯誤的區間表示。

本章會建立一套固定思考方式：

1. 先確認資料的邏輯長度與有效 Index 範圍。
2. 區分固定大小 Array 與可變長度 Dynamic Array。
3. 從元素搬移量判斷插入與刪除成本。
4. 使用 Half-open Interval 表示走訪範圍。
5. 修改 `std::vector` 前，先判斷是否可能 Reallocation。
6. 使用 Invariant 說明 Read/Write Pointer。
7. 對二維資料明確定義 Row、Column 與扁平化公式。
8. 從空資料、單一元素、最後一格與容量邊界建立測試。

### 適用讀者

- 需要建立 Two Pointers、Sliding Window、Prefix Sum 與 Binary Search 基礎的讀者。
- 知道如何宣告 Array，但不清楚連續儲存意義的讀者。
- 常混淆 `size`、`capacity`、`reserve` 與 `resize` 的讀者。
- 經常遇到越界、最後一輪錯誤或倒序迴圈無法結束的讀者。
- 在修改 `std::vector` 後，仍沿用舊 Pointer、Reference 或 Iterator 的讀者。
- 想理解 In-place 與 Read/Write Pointer 的讀者。
- 同時使用 C++ 與 C，需要整理兩者 Array 介面差異的讀者。

### 快速導覽

- [Array 到底保存什麼](#51-array-到底保存什麼)：建立連續儲存、長度與 Index 模型。
- [第一步：確認有效範圍](#52-第一步確認有效範圍)：使用 `[0, size)` 避免越界。
- [第二步：從資料搬移理解操作成本](#53-第二步從資料搬移理解操作成本)：分析查找、插入與刪除。
- [第三步：理解 std::vector 的 Size 與 Capacity](#54-第三步理解-stdvector-的-size-與-capacity)：區分元素數量與已配置空間。
- [第四步：區分 reserve 與 resize](#55-第四步區分-reserve-與-resize)：確認容量變化與元素建立。
- [第五步：處理 Reallocation 與 Iterator Invalidation](#56-第五步處理-reallocation-與-iterator-invalidation)：避免沿用失效位置。
- [完整案例：原地移除排序 Array 的重複值](#57-完整案例原地移除排序-array-的重複值)：使用 Read/Write Pointer 與 Invariant。
- [第六步：安全地走訪與倒序](#58-第六步安全地走訪與倒序)：處理空 Array 與 Unsigned Underflow。
- [第七步：理解二維資料](#59-第七步理解二維資料)：比較巢狀 Vector 與扁平配置。
- [完整案例：扁平矩陣走訪](#510-完整案例扁平矩陣走訪)：建立 Row-major Index 公式。
- [C 語言中的 Array](#511-c-語言中的-array)：補充長度、Pointer 與固定容量介面。
- [建立自己的 Array 分析表](#512-建立自己的-array-分析表)：形成固定檢查流程。
- [常見問題與判讀](#513-常見問題與判讀)：整理常見錯誤與第一輪檢查。
- [本章檢查表](#514-本章檢查表)：確認必要觀念是否完整。
- [本章重點](#515-本章重點)：回顧核心方法。

### 5.1 Array 到底保存什麼

Array 將相同型別的元素依序放在連續位置。若第一個元素的位址為 Base Address，每個元素占用 `sizeof(T)` Bytes，則第 `i` 個元素的位置可表示成：

```text
Base Address + i × sizeof(T)
```

這個關係讓程式不需要從第一個元素逐一走到第 `i` 個元素，而是可以直接計算位置。因此，按 Index 讀寫通常是 O(1)。

#### 邏輯模型與記憶體模型

假設有以下整數 Array：

```text
values = [7, 3, 9, 2]
```

它的邏輯模型是：

```text
Index：  0  1  2  3
Value：  7  3  9  2
```

若每個 `int` 占用 4 Bytes，元素位址可能呈現：

```text
values[0]：Base + 0 × 4
values[1]：Base + 1 × 4
values[2]：Base + 2 × 4
values[3]：Base + 3 × 4
```

實際位址由執行環境決定，但相鄰元素的位址差通常等於一個元素的大小。

#### 連續儲存帶來什麼

連續儲存的主要影響包括：

- 可以用 Index 直接定位元素。
- 依序走訪時，存取位置彼此接近。
- 中間插入時，後方元素通常需要搬移。
- 中間刪除時，後方元素通常需要往前補位。
- 若 Dynamic Array 容量不足，可能需要搬到新的連續空間。

#### Array 與 Dynamic Array

固定大小 Array 的長度通常在建立後不變：

```cpp
std::array<int, 4> values{7, 3, 9, 2};
```

Dynamic Array 可以改變元素數量。C++ 常用 `std::vector`：

```cpp
std::vector<int> values{7, 3, 9, 2};
values.push_back(5);
```

`std::vector` 對使用者提供連續元素，但為了支援成長，內部通常會保留尚未建立元素的容量。這就是 `size` 與 `capacity` 需要分開理解的原因。

### 5.2 第一步：確認有效範圍

若 Array 有 `size` 個元素，有效 Index 是：

```text
0, 1, 2, ..., size - 1
```

通常寫成 Half-open Interval：

```text
[0, size)
```

左邊 0 包含在範圍內，右邊 `size` 不包含在範圍內。

#### 為什麼使用 Half-open Interval

Half-open Interval 有幾個實用性質：

- 範圍長度是 `right - left`。
- 空範圍可以表示成 `[x, x)`。
- 相鄰範圍容易拼接，例如 `[0, middle)` 與 `[middle, size)`。
- 迴圈條件自然寫成 `i < size`。

標準走訪方式：

```cpp
for (std::size_t i = 0; i < values.size(); ++i)
{
    use(values[i]);
}
```

#### 常見越界錯誤

```cpp
for (std::size_t i = 0; i <= values.size(); ++i)
{
    use(values[i]);
}
```

當 `i == values.size()` 時，`values[i]` 已在有效範圍之外。

若 `size == 4`，有效 Index 是 0、1、2、3，而不是 0、1、2、3、4。

#### 空 Array

當 `size == 0` 時：

```text
有效範圍 = [0, 0)
```

它是一個空範圍。使用 `i < size` 的正向迴圈不會進入，通常不需要額外分支。

但以下存取仍然無效：

```cpp
values[0]
values.front()
values.back()
```

在讀取第一個或最後一個元素前，需要先確認容器非空，或由 Precondition 保證非空。

#### `operator[]` 與 `at()`

`std::vector` 提供兩種常見 Index 存取方式：

```cpp
int a = values[index];
int b = values.at(index);
```

差異可概括為：

- `operator[]` 不進行標準要求的範圍檢查，越界會造成未定義行為。
- `at()` 會檢查範圍，越界時丟出 `std::out_of_range`。

演算法題常使用 `operator[]`，但前提是區間推理已確認 Index 合法。除錯階段也可以使用 `at()` 協助定位越界。

#### 區間 Precondition

若函式處理 `[left, right)`，常見 Precondition 是：

```text
0 <= left <= right <= size
```

若函式處理 Inclusive Interval `[left, right]`，則常見 Precondition 是：

```text
0 <= left <= right < size
```

兩種都可使用，但同一段程式、註解與測試必須保持一致。

### 5.3 第二步：從資料搬移理解操作成本

不要只背誦複雜度。先問每個動作需要定位多少元素，以及需要搬移多少元素。

<table>
<tr><th>動作</th><th>常見成本</th><th>主要原因</th></tr>
<tr><td>按 Index 讀寫</td><td>O(1)</td><td>由 Base Address 與 Index 直接計算位置</td></tr>
<tr><td>依序走訪全部元素</td><td>O(n)</td><td>每個元素處理一次</td></tr>
<tr><td>未排序資料查找</td><td>O(n)</td><td>最差需要檢查所有元素</td></tr>
<tr><td>尾端追加</td><td>攤銷 O(1)</td><td>多數追加不搬移全部元素，偶爾需要擴容</td></tr>
<tr><td>尾端刪除</td><td>O(1)</td><td>通常只移除最後一個元素</td></tr>
<tr><td>中間插入</td><td>O(n)</td><td>插入位置後方元素需要後移</td></tr>
<tr><td>中間刪除</td><td>O(n)</td><td>刪除位置後方元素需要前移</td></tr>
</table>

#### 未排序查找

若資料未排序，要找數值 9：

```text
[7, 3, 9, 2]
```

可能需要查看 7、3，再看到 9。若目標不存在，最差需要檢查全部元素。

```cpp
bool contains(
    const std::vector<int>& values,
    int target)
{
    for (int value : values)
    {
        if (value == target)
        {
            return true;
        }
    }
    return false;
}
```

此函式的 Loop Invariant 可寫成：

> 每輪開始前，所有已走訪元素都不等於 `target`。

若找到相同元素便回傳 `true`；若完整走訪仍未找到，便能推出輸入中不存在 `target`。

#### 中間插入

在以下資料的 Index 1 插入 8：

```text
插入前：[7, 3, 9, 2]
插入後：[7, 8, 3, 9, 2]
```

原本位於 Index 1 到 3 的元素需要往後移。插入位置越靠近開頭，需要搬移的元素通常越多。

#### 中間刪除

刪除 Index 1 的元素：

```text
刪除前：[7, 3, 9, 2]
刪除後：[7, 9, 2]
```

原本位於後方的 9 與 2 需要往前補位。因此，即使找到刪除位置只需 O(1)，保持元素連續仍需要搬移。

#### 為什麼尾端追加是攤銷 O(1)

`push_back` 在容量足夠時，通常只需要在尾端建立一個元素，成本接近 O(1)。

容量不足時，Vector 可能需要：

1. 配置一塊更大的連續空間。
2. 將原有元素搬移或複製到新空間。
3. 釋放舊空間。
4. 在尾端建立新元素。

單次 Reallocation 可能是 O(n)，但不會每次追加都發生。將一連串追加的總成本平均到每次追加，稱為攤銷 O(1)。

這不表示每一次 `push_back` 都保證 O(1)。若某一輪的延遲特別敏感，仍需考慮 Reallocation。

### 5.4 第三步：理解 std::vector 的 Size 與 Capacity

`std::vector` 至少要區分三個概念：

<table>
<tr><th>概念</th><th>意義</th></tr>
<tr><td>size</td><td>目前已建立、可合法存取的元素數量</td></tr>
<tr><td>capacity</td><td>目前配置空間在下次 Reallocation 前可容納的元素數量</td></tr>
<tr><td>data</td><td>連續元素區域的起始位置</td></tr>
</table>

永遠有：

```text
size <= capacity
```

但 `capacity - size` 的空間不是已建立元素，不能直接用 Index 當成有效元素存取。

#### 基本觀察

```cpp
std::vector<int> values;
```

此時：

```text
values.size() == 0
```

`capacity()` 的具體值不應被演算法假設。接著：

```cpp
values.push_back(7);
values.push_back(3);
```

此時：

```text
size == 2
capacity >= 2
```

合法 Index 仍只有：

```text
0 與 1
```

即使 `capacity == 4`，`values[2]` 與 `values[3]` 也不是已建立元素。

#### Size 是邏輯範圍

演算法走訪、輸出與邊界檢查通常應以 `size()` 為準：

```cpp
for (std::size_t i = 0; i < values.size(); ++i)
{
    process(values[i]);
}
```

`capacity()` 主要和記憶體配置與後續成長有關，不代表目前資料長度。

#### 不要依賴容量成長倍率

不同標準函式庫可能使用不同容量成長策略。程式可以依賴以下關係：

```text
capacity >= size
```

但不應假設每次一定加倍，也不應把特定成長倍率寫成演算法正確性的條件。

### 5.5 第四步：區分 reserve 與 resize

`reserve` 與 `resize` 名稱相近，但改變的狀態不同。

#### `reserve(n)`

`reserve(n)` 要求 Vector 能在不再次 Reallocation 的情況下，至少容納 `n` 個元素。它不會把元素數量直接改成 `n`。

```cpp
std::vector<int> values;
values.reserve(100);
```

執行後：

```text
values.size() == 0
values.capacity() >= 100
```

因此下面仍然錯誤：

```cpp
values[0] = 7;
```

因為 Index 0 尚不在 `[0, size)` 內。

合法加入方式是：

```cpp
values.push_back(7);
```

#### `resize(n)`

`resize(n)` 會將元素數量改成 `n`。

```cpp
std::vector<int> values;
values.resize(100);
```

執行後：

```text
values.size() == 100
values.capacity() >= 100
```

對 `std::vector<int>` 而言，新建立的元素會進行值初始化，因此可以合法存取 `values[0]` 到 `values[99]`。

#### 縮小 `resize`

```cpp
values.resize(3);
```

若原本 `size()` 大於 3，Index 3 之後的元素會被移除。容量不一定同步縮小。

因此：

```text
resize 改變 size
但不保證 capacity 等於新的 size
```

#### 使用時機

若已知即將 `push_back` 大量元素，但不需要事先建立它們：

```cpp
std::vector<int> values;
values.reserve(expectedCount);

for (...)
{
    values.push_back(nextValue);
}
```

若演算法需要先建立固定數量元素，再按 Index 填入：

```cpp
std::vector<int> values(count);

for (std::size_t i = 0; i < count; ++i)
{
    values[i] = compute(i);
}
```

#### `reserve` 不是必要的正確性條件

多數情況下，不呼叫 `reserve` 仍能得到相同邏輯結果。它主要用於：

- 減少已知成長過程中的 Reallocation。
- 降低元素反覆搬移的成本。
- 在特定流程中，協助維持 Pointer 或 Iterator 的有效性，但仍需遵守容量限制。

若追加數量超過保留容量，Reallocation 仍可能發生。

### 5.6 第五步：處理 Reallocation 與 Iterator Invalidation

`std::vector` 的元素位於連續空間。當原本空間無法容納更多元素時，Vector 可能搬到新的位置。搬移後，指向舊元素位置的 Pointer、Reference 與 Iterator 便不能再沿用。

#### Pointer 失效案例

```cpp
std::vector<int> values{10, 20, 30};
int* first = &values[0];

values.push_back(40);

// 若 push_back 造成 Reallocation，first 已失效。
```

問題不在於 `values[0]` 消失，而是它可能已搬到新位址。

#### Reference 與 Iterator 也有相同風險

```cpp
int& firstValue = values[0];
auto iterator = values.begin();

values.push_back(50);
```

若發生 Reallocation：

- `firstValue` 失效。
- `iterator` 失效。
- 原本取得的元素 Pointer 失效。

#### 如何降低風險

##### 修改後重新取得位置

```cpp
values.push_back(50);
auto iterator = values.begin();
```

若流程允許，修改容器後重新取得 Pointer、Reference 或 Iterator，是較直接的方式。

##### 保存 Index，而不是保存 Iterator

如果只需要記錄邏輯位置，可以保存 Index：

```cpp
std::size_t position = 0;
values.push_back(50);
process(values[position]);
```

但仍需確認修改後該 Index 是否存在，以及插入或刪除是否改變了元素的邏輯位置。

##### 預先 `reserve`

若可預估最大元素數量：

```cpp
values.reserve(maximumCount);
```

在 `size()` 未超過保留容量前，尾端追加通常不需 Reallocation。但中間插入與刪除仍可能使部分 Iterator 失效。

#### 中間 Insert 與 Erase

即使沒有 Reallocation，中間插入也會搬移插入位置之後的元素。因此，指向該位置及其後方的 Pointer、Reference 與 Iterator 可能失效。

`erase` 會讓後方元素前移，因此刪除位置及其後方的 Iterator 也不能直接沿用。

常見安全模式是使用 `erase` 回傳的新 Iterator：

```cpp
for (auto it = values.begin(); it != values.end(); )
{
    if (shouldRemove(*it))
    {
        it = values.erase(it);
    }
    else
    {
        ++it;
    }
}
```

`erase(it)` 回傳指向刪除元素後一個位置的新 Iterator。若忽略回傳值並繼續遞增舊 Iterator，可能使用已失效位置。

#### 修改容器前的檢查問題

在對 `std::vector` 執行 `push_back`、`insert`、`erase`、`resize` 或 `reserve` 前，可以先問：

- 是否保存了元素 Pointer？
- 是否保存了元素 Reference？
- 是否保存了 Iterator？
- 修改是否可能 Reallocation？
- 即使沒有 Reallocation，元素是否會被搬移？
- 修改後是否應重新取得位置？

### 5.7 完整案例：原地移除排序 Array 的重複值

#### 問題規格

給定一個已由小到大排序的整數 Array，原地整理資料，使前 `k` 個位置保存所有不同值，每個值只出現一次，並回傳 `k`。

例如：

```text
輸入：[1, 1, 2, 2, 2, 4]
回傳：3
有效結果：[1, 2, 4]
```

函式回傳後，只有 `[0, k)` 被定義為結果。`k` 之後的內容不需要符合特定格式。

#### Precondition

- `nums` 已依非遞減順序排序。

排序條件很重要。相同值會相鄰，演算法才能只比較目前讀取值和最後一個保留值。

#### Postcondition

令回傳值為 `k`：

- `0 <= k <= nums.size()`。
- `[0, k)` 中沒有重複值。
- `[0, k)` 依非遞減順序排列。
- `[0, k)` 恰好包含原輸入中所有不同值。

#### C++ 解法

```cpp
#include <vector>

int removeDuplicates(std::vector<int>& nums)
{
    if (nums.empty())
    {
        return 0;
    }

    int write = 1;

    for (int read = 1;
         read < static_cast<int>(nums.size());
         ++read)
    {
        if (nums[read] != nums[write - 1])
        {
            nums[write] = nums[read];
            ++write;
        }
    }

    return write;
}
```

#### Read 與 Write 各自代表什麼

- `read`：下一個要檢查的輸入位置。
- `write`：下一個不同值應寫入的位置，也是目前結果長度。

可以將 Array 分成三段：

```text
[0, write)    ：已整理的不重複結果
[write, read) ：已讀取但不屬於最終有效結果的舊內容
[read, size)  ：尚未處理的輸入
```

#### Loop Invariant

每輪開始前：

1. `[0, write)` 是原輸入 `[0, read)` 的不重複結果。
2. `[0, write)` 保持排序。
3. `nums[write - 1]` 是目前已保留的最後一個不同值。
4. `1 <= write <= read`。

#### Initialization

輸入非空時：

```text
write = 1
read = 1
```

已處理範圍 `[0, 1)` 只有第一個元素：

- 它沒有重複。
- 它已排序。
- 它就是原輸入前一個元素的不重複結果。

因此 Invariant 成立。

#### Maintenance

本輪檢查 `nums[read]`。

##### 情況一：和最後保留值相同

```cpp
nums[read] == nums[write - 1]
```

因為輸入已排序，目前值屬於同一組重複值，不需要寫入。`write` 保持不變，已整理結果仍正確。

##### 情況二：和最後保留值不同

```cpp
nums[read] != nums[write - 1]
```

因為輸入已排序，目前值大於最後保留值，是新的不同值。將它寫到 `nums[write]`，再增加 `write`：

```cpp
nums[write] = nums[read];
++write;
```

更新後 `[0, write)` 加入一個新的不同值，仍保持排序，也恰好代表擴大後已處理範圍的不重複結果。

#### Termination

當 `read == nums.size()` 時，所有輸入元素都已處理。由 Invariant 可得：

- `[0, write)` 恰好包含原輸入的全部不同值。
- `write` 是不同值的數量。

因此回傳 `write` 符合 Postcondition。

#### 逐輪執行

輸入：`[1, 1, 2, 2, 2, 4]`

<table>
<tr><th>read</th><th>目前值</th><th>最後保留值</th><th>動作</th><th>write</th><th>有效結果</th></tr>
<tr><td>1</td><td>1</td><td>1</td><td>略過重複值</td><td>1</td><td>[1]</td></tr>
<tr><td>2</td><td>2</td><td>1</td><td>寫入 Index 1</td><td>2</td><td>[1, 2]</td></tr>
<tr><td>3</td><td>2</td><td>2</td><td>略過重複值</td><td>2</td><td>[1, 2]</td></tr>
<tr><td>4</td><td>2</td><td>2</td><td>略過重複值</td><td>2</td><td>[1, 2]</td></tr>
<tr><td>5</td><td>4</td><td>2</td><td>寫入 Index 2</td><td>3</td><td>[1, 2, 4]</td></tr>
</table>

#### 為什麼可以覆寫尚未使用的位置

Invariant 包含 `write <= read`。因此寫入位置不會超過目前讀取位置：

- 若 `write < read`，正在覆寫已處理過的位置。
- 若 `write == read`，只是把元素寫回原位置。

尚未處理範圍從 `read` 開始，演算法不會覆寫未來需要讀取的資料。

#### 時間與額外空間

- 時間複雜度：O(n)，每個元素最多讀取一次。
- 額外空間複雜度：O(1)，只使用 `read` 與 `write` 等固定數量變數。

這裡的 In-place 表示不另外建立和輸入大小成比例的容器。它不表示完全沒有寫入動作。

#### 邊界案例

<table>
<tr><th>輸入</th><th>回傳 k</th><th>有效結果</th><th>目的</th></tr>
<tr><td>[]</td><td>0</td><td>[]</td><td>空輸入</td></tr>
<tr><td>[5]</td><td>1</td><td>[5]</td><td>單一元素</td></tr>
<tr><td>[2, 2, 2]</td><td>1</td><td>[2]</td><td>全部相同</td></tr>
<tr><td>[1, 2, 3]</td><td>3</td><td>[1, 2, 3]</td><td>完全沒有重複</td></tr>
<tr><td>[-2, -2, 0, 3, 3]</td><td>3</td><td>[-2, 0, 3]</td><td>負數與重複值</td></tr>
</table>

#### 未排序輸入會發生什麼

例如：

```text
[2, 1, 2]
```

第二個 2 不和第一個 2 相鄰。此演算法只和最後保留值比較，可能保留兩次 2。因此，排序是演算法成立的必要 Precondition，不只是效能提示。

### 5.8 第六步：安全地走訪與倒序

#### 優先使用 Range-based for

若只需要依序讀取每個值，不需要 Index：

```cpp
for (int value : values)
{
    process(value);
}
```

若需要修改元素：

```cpp
for (int& value : values)
{
    value *= 2;
}
```

若元素較大且只讀，可使用 `const` Reference：

```cpp
for (const Item& item : items)
{
    process(item);
}
```

#### 需要 Index 時再使用 Index 迴圈

以下情況通常需要 Index：

- 要比較相鄰元素。
- 要處理特定區間。
- 要回傳位置。
- 同時走訪多個 Array。
- 要使用 Read/Write Pointer。

#### Unsigned Underflow

`size_t` 是 Unsigned Integer。以下倒序寫法有風險：

```cpp
for (std::size_t i = values.size() - 1; i >= 0; --i)
{
    process(values[i]);
}
```

問題包括：

- 空 Array 時，`values.size() - 1` 會 Underflow。
- `i >= 0` 對 Unsigned Integer 永遠成立。
- 當 `i == 0` 再執行 `--i`，會繞到很大的值。

#### 安全倒序方式一

```cpp
for (std::size_t i = values.size(); i-- > 0; )
{
    process(values[i]);
}
```

此寫法使用 `i` 作為尚未處理元素數量。空 Array 時條件立即失敗。

不過 `i-- > 0` 的閱讀成本較高，註解應說明範圍語意。

#### 安全倒序方式二

使用 Reverse Iterator：

```cpp
for (auto it = values.rbegin(); it != values.rend(); ++it)
{
    process(*it);
}
```

若不需要原始 Index，Reverse Iterator 通常更清楚。

#### 安全倒序方式三

若確定大小可轉成 Signed Integer，先明確轉型：

```cpp
for (int i = static_cast<int>(values.size()) - 1;
     i >= 0;
     --i)
{
    process(values[i]);
}
```

此方式需要額外確認 `values.size()` 不超過 `int` 可表示範圍。因此，轉型本身也屬於 Precondition 或限制條件的一部分。

### 5.9 第七步：理解二維資料

二維資料在邏輯上以 Row 與 Column 存取：

```text
matrix[row][column]
```

但不同表示方式的記憶體模型可能不同。

#### `vector<vector<int>>`

```cpp
std::vector<std::vector<int>> matrix(
    rows,
    std::vector<int>(columns));
```

每一個內層 `vector<int>` 各自管理自己的連續空間。因此：

- 同一列內的元素連續。
- 不同列之間不保證相鄰。
- 每列長度可以不同。

以下是不規則二維資料：

```cpp
std::vector<std::vector<int>> rows{
    {1, 2, 3},
    {4},
    {5, 6}
};
```

所以不能只用第一列長度假設所有列都有相同 Column 數。

#### 二維邊界

對 `matrix[row][column]`，需要分兩步檢查：

```text
0 <= row < matrix.size()
0 <= column < matrix[row].size()
```

若 `matrix` 為空，不能先讀取 `matrix[0].size()`。

#### 扁平化配置

若希望整個矩陣位於單一連續區域，可使用一維 Vector：

```cpp
std::vector<int> matrix(rows * columns);
```

採 Row-major 排列時，位置公式是：

```text
index = row × columns + column
```

存取方式：

```cpp
int& value = matrix[row * columns + column];
```

#### 為什麼公式成立

每一個完整 Row 有 `columns` 個元素。

在 `row` 之前，共有：

```text
row × columns
```

個元素。再往後移動 `column` 個位置，就得到：

```text
row × columns + column
```

#### Row-major 走訪

```cpp
for (std::size_t row = 0; row < rows; ++row)
{
    for (std::size_t column = 0;
         column < columns;
         ++column)
    {
        process(matrix[row * columns + column]);
    }
}
```

這個順序依序存取同一 Row 的相鄰元素，符合 Row-major 配置。

若交換兩層迴圈：

```cpp
for (std::size_t column = 0; column < columns; ++column)
{
    for (std::size_t row = 0; row < rows; ++row)
    {
        process(matrix[row * columns + column]);
    }
}
```

邏輯結果可能相同，但每次存取會跨過一整列。在大型資料上，存取位置較分散，可能影響實際執行效率。

#### 配置大小的 Overflow

以下乘法也需要檢查：

```cpp
rows * columns
```

如果兩者很大，乘積可能超過 `size_t` 可表示範圍。安全介面應在配置前確認：

```text
columns == 0
或
rows <= max_size / columns
```

題目限制若已保證乘積很小，可以將它列為 Precondition；正式系統程式則應考慮檢查失敗情況。

### 5.10 完整案例：扁平矩陣走訪

#### 問題規格

給定一個以 Row-major 方式保存在一維 Vector 中的 `rows × columns` 整數矩陣，計算所有元素總和。

#### Precondition

- `rows * columns == matrix.size()`，且乘法沒有 Overflow。

#### Postcondition

回傳值等於矩陣每一格的總和。

#### C++ 解法

```cpp
#include <cstddef>
#include <vector>

long long matrixSum(
    const std::vector<int>& matrix,
    std::size_t rows,
    std::size_t columns)
{
    long long sum = 0;

    for (std::size_t row = 0; row < rows; ++row)
    {
        for (std::size_t column = 0;
             column < columns;
             ++column)
        {
            const std::size_t index =
                row * columns + column;
            sum += matrix[index];
        }
    }

    return sum;
}
```

#### 外層 Invariant

每次外層迴圈開始前：

> `sum` 等於 Row `[0, row)` 中所有元素的總和。

#### 內層 Invariant

固定目前 `row`，每次內層迴圈開始前：

> `sum` 等於所有先前 Row，加上目前 Row 的 Column `[0, column)` 的總和。

#### Initialization

開始時：

```text
row = 0
column 尚未開始
sum = 0
```

尚未處理任何 Row，空範圍總和為 0，因此外層 Invariant 成立。

每個 Row 的內層迴圈從 `column = 0` 開始，目前 Row 尚未處理任何元素，所以內層 Invariant 也成立。

#### Maintenance

每輪內層迴圈將：

```cpp
matrix[row * columns + column]
```

加入 `sum`。因此，目前 Row 已處理範圍會從 `[0, column)` 擴大為 `[0, column + 1)`。

內層結束時，整個目前 Row 都已加入。外層進入下一輪後，外層 Invariant 仍成立。

#### Termination

當 `row == rows` 時，Row `[0, rows)` 已全部處理。由 Invariant 可得：

- `sum` 等於矩陣全部元素的總和。

#### 可以直接走訪一維 Vector 嗎

若只需要計算全部元素總和，確實可以直接寫成：

```cpp
long long sum = 0;
for (int value : matrix)
{
    sum += value;
}
```

這個版本更簡單。本案例使用 Row 與 Column，是為了說明扁平化 Index 與二維區間。若演算法不需要座標資訊，通常不必刻意還原 Row 與 Column。

### 5.11 C 語言中的 Array

#### Array 參數不會自動帶入長度

```c
void process(const int values[]);
```

在函式參數中，`values` 不包含元素數量資訊。通常需要另外傳入：

```c
void process(
    const int values[],
    size_t length);
```

函式的 Precondition 應包含：

- `values` 指向至少 `length` 個有效元素。
- 若 `length > 0`，`values` 不可為 `NULL`。

#### 固定容量與邏輯長度

C 中可使用固定容量 Array 模擬可變長度資料：

```c
#define CAPACITY 100

int values[CAPACITY];
size_t length = 0;
```

此時要區分：

- `CAPACITY`：最多可容納多少元素。
- `length`：目前有多少有效元素。

合法資料範圍是：

```text
[0, length)
```

不是 `[0, CAPACITY)`。

#### 尾端加入元素

```c
#include <stdbool.h>
#include <stddef.h>

bool push_back(
    int values[],
    size_t capacity,
    size_t *length,
    int value)
{
    if (values == NULL || length == NULL)
    {
        return false;
    }

    if (*length >= capacity)
    {
        return false;
    }

    values[*length] = value;
    ++(*length);
    return true;
}
```

這個函式的 Precondition 與 Postcondition可整理為：

- `values` 指向至少 `capacity` 個元素的空間。
- `*length <= capacity`。
- 成功時，新值寫入原本的尾端，`*length` 增加 1。
- 容量已滿時回傳 `false`，資料與長度保持不變。

#### C 的二維 Array

固定 Column 數量的二維 Array：

```c
void process_matrix(
    size_t rows,
    size_t columns,
    int matrix[rows][columns]);
```

若使用支援 Variable Length Array 的 C 版本與編譯器，函式可以接收執行期 Column 數。專案若不使用 VLA，也可以傳入扁平 Pointer：

```c
int value = matrix[row * columns + column];
```

不論介面形式如何，Row 與 Column 的邊界條件仍需分別確認。

#### `sizeof` 的限制

在同一個 Scope 內對真正的固定 Array 使用：

```c
size_t length = sizeof(values) / sizeof(values[0]);
```

可以計算元素數量。但 Array 傳入函式後，參數通常調整為 Pointer，這個公式不能再取得原始 Array 長度。因此函式仍應明確接收 `length`。

### 5.12 建立自己的 Array 分析表

遇到 Array 題目時，可以先填以下內容：

<table>
<tr><th>欄位</th><th>要回答的問題</th></tr>
<tr><td>資料型別</td><td>元素是整數、字元、物件，還是其他型別？</td></tr>
<tr><td>邏輯長度</td><td>目前有效元素有多少？是否可能為 0？</td></tr>
<tr><td>容量</td><td>固定容量還是 Dynamic Array？容量是否可能不足？</td></tr>
<tr><td>有效區間</td><td>使用 `[0, size)`、`[left, right)` 還是 `[left, right]`？</td></tr>
<tr><td>排序條件</td><td>已排序、部分排序還是未排序？</td></tr>
<tr><td>修改權限</td><td>可否改變元素、順序或長度？</td></tr>
<tr><td>操作種類</td><td>查找、更新、尾端追加、中間插入或刪除？</td></tr>
<tr><td>搬移成本</td><td>該動作需要搬移多少元素？</td></tr>
<tr><td>額外空間</td><td>可否建立新 Array，還是要求 In-place？</td></tr>
<tr><td>Pointer 有效性</td><td>修改 Vector 後，是否仍持有 Pointer、Reference 或 Iterator？</td></tr>
<tr><td>Read/Write State</td><td>Read、Write 各自代表哪個區間？兩者關係是什麼？</td></tr>
<tr><td>二維布局</td><td>每列獨立配置，還是單一 Row-major 空間？</td></tr>
<tr><td>型別安全</td><td>Index 是 Signed 還是 Unsigned？乘法或轉型是否安全？</td></tr>
<tr><td>邊界案例</td><td>空、單一元素、最後一格、容量已滿與最大尺寸如何處理？</td></tr>
</table>

這張表的目的不是增加形式工作，而是讓長度、容量、區間與修改行為在寫程式前保持一致。

### 5.13 常見問題與判讀

<table>
<tr><th>現象</th><th>可能原因</th><th>第一輪檢查</th></tr>
<tr><td>最後一次走訪越界</td><td>使用 `i <= size`</td><td>有效 Index 是否為 `[0, size)`</td></tr>
<tr><td>空 Array 立即失敗</td><td>直接讀取第一個或最後一個元素</td><td>是否需要 `empty()` 檢查或非空 Precondition</td></tr>
<tr><td>`reserve` 後寫入 Index 0 失敗</td><td>混淆 Size 與 Capacity</td><td>是否應使用 `resize` 或 `push_back`</td></tr>
<tr><td>`resize` 後多出許多 0</td><td>誤以為 `resize` 只保留容量</td><td>是否應改用 `reserve`</td></tr>
<tr><td>追加元素後 Pointer 突然無效</td><td>Vector 發生 Reallocation</td><td>修改後是否重新取得位置</td></tr>
<tr><td>Erase 後迴圈漏掉元素</td><td>舊 Iterator 失效或額外遞增</td><td>是否使用 `erase` 的回傳 Iterator</td></tr>
<tr><td>倒序迴圈無法結束</td><td>Unsigned Index 永遠大於等於 0</td><td>改用 Reverse Iterator 或安全倒序條件</td></tr>
<tr><td>空 Array 倒序從巨大 Index 開始</td><td>`size() - 1` 發生 Underflow</td><td>是否在減 1 前確認非空</td></tr>
<tr><td>原地整理覆寫尚未讀取資料</td><td>Read/Write 關係不成立</td><td>是否能證明 `write <= read`</td></tr>
<tr><td>移除重複值仍保留重複</td><td>輸入未排序</td><td>排序是否為必要 Precondition</td></tr>
<tr><td>二維資料某些列越界</td><td>假設每列長度相同</td><td>是否使用 `matrix[row].size()` 檢查各列</td></tr>
<tr><td>扁平矩陣位置錯誤</td><td>Row-major 公式寫反</td><td>確認 `row * columns + column`</td></tr>
<tr><td>大型矩陣配置異常</td><td>`rows * columns` Overflow</td><td>乘法前是否檢查最大可配置數量</td></tr>
<tr><td>C 函式無法得知 Array 長度</td><td>參數只收到 Pointer</td><td>是否另外傳入 `length`</td></tr>
<tr><td>中間插入比預期慢</td><td>後方元素需要搬移</td><td>是否能改成尾端追加或改用其他資料結構</td></tr>
</table>

### 5.14 本章檢查表

- 我能說明連續儲存和 O(1) Index 存取的關係。
- 我知道 Array 的有效 Index 通常是 `[0, size)`。
- 我能分辨 Inclusive Interval 與 Half-open Interval。
- 我會在讀取 `front()`、`back()` 或 Index 0 前處理空 Array。
- 我能從元素搬移量說明中間插入與刪除為何是 O(n)。
- 我知道 `push_back` 是攤銷 O(1)，不是每一輪都保證 O(1)。
- 我能區分 `std::vector` 的 `size()` 與 `capacity()`。
- 我知道 `capacity()` 範圍內但 `size()` 之外不是有效元素。
- 我能區分 `reserve` 與 `resize`。
- 我不會在只呼叫 `reserve` 後直接用 Index 寫入尚未建立的元素。
- 我知道 Reallocation 可能讓 Pointer、Reference 與 Iterator 失效。
- 我知道中間 `insert` 或 `erase` 即使沒有 Reallocation，也可能使後方位置失效。
- 我會在修改 Vector 後重新取得必要的位置。
- 我能使用 `erase` 回傳的 Iterator 繼續走訪。
- 我能定義 Read Pointer、Write Pointer 與各段區間的意義。
- 我能用 Invariant 說明 `[0, write)` 保存什麼結果。
- 我能證明 In-place 寫入不會破壞尚未讀取資料。
- 我能安全處理空 Array 與倒序走訪。
- 我知道 Unsigned Integer 在 0 再減 1 會 Underflow。
- 我知道 `vector<vector<int>>` 不保證所有列位於同一塊連續空間。
- 我能使用 `row * columns + column` 存取 Row-major 扁平矩陣。
- 我會分別檢查 Row 與 Column 邊界。
- 我會考慮 `rows * columns` 是否可能 Overflow。
- 我知道 C Array 參數通常需要另外傳入長度。
- 我能區分 C 中的固定容量與目前邏輯長度。

### 5.15 本章重點

- Array 的核心模型是相同型別元素依序放在連續位置。
- 連續儲存讓程式能由 Base Address 與 Index 直接定位元素，因此隨機存取通常是 O(1)。
- Array 的有效 Index 為 `[0, size)`，`size` 本身不是合法 Index。
- Half-open Interval 能自然表示長度、空範圍與相鄰區間。
- 未排序查找通常需要 O(n)，中間插入與刪除也通常需要 O(n)。
- `std::vector::size()` 表示已建立元素數，`capacity()` 表示目前配置可容納的數量。
- `reserve` 只保留容量，不建立元素；`resize` 會改變元素數量。
- Vector 擴容可能發生 Reallocation，使原有 Pointer、Reference 與 Iterator 失效。
- 中間 Insert 與 Erase 會搬移元素，也會影響相關位置的有效性。
- Read/Write Pointer 可在同一個 Array 中整理資料，核心是清楚定義 `[0, write)` 與尚未處理範圍。
- 原地移除排序 Array 的重複值依賴排序 Precondition 與 `write <= read`。
- Unsigned Index 的倒序走訪需要避免 Underflow。
- `vector<vector<int>>` 的每列各自配置，若要整體連續，可使用一維 Vector 扁平化矩陣。
- Row-major 位置公式是 `row * columns + column`，Row 與 Column 邊界都必須檢查。
- C 函式中的 Array 參數不會自動攜帶長度，通常需要另外傳入 `length`。
- Array 是 Two Pointers、Sliding Window、Prefix Sum、Binary Search 與多種後續演算法的共同基礎。
