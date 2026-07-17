## 第 4 章　正確性與測試方法

### 適用範圍

本章說明完成演算法後，如何確認程式不只是通過 Sample，而是在所有符合題目條件的輸入下，都能產生正確結果並正常終止。

很多程式錯誤不是語法問題，而是以下情況尚未被確認：
- 函式預期收到什麼樣的輸入。
- 函式完成後必須保證什麼結果。
- 迴圈每一輪保存的狀態代表什麼。
- 遞迴是否一定會到達 Base Case。
- 空輸入、單一元素、重複值與極端數值如何處理。
- 測試失敗時，要相信哪一個答案。
- 如何把大型失敗資料縮小成容易追蹤的案例。

測試可以指出某些輸入沒有問題，但有限次測試無法單獨證明所有輸入都正確。因此，本章會同時建立兩條驗證路線：
- 使用 Precondition、Postcondition、Loop Invariant 與遞迴推理說明演算法為何正確。
- 使用邊界案例、測試 Oracle、對拍、Property-based Testing 與 Regression Test 找出實作錯誤。

本章會建立一套固定流程：
1. 先寫出 Precondition 與 Postcondition。
2. 區分部分正確性與終止性。
3. 為迴圈找出可維持的 Invariant。
4. 為遞迴確認 Base Case、問題縮小與答案組合。
5. 從規格推導邊界案例。
6. 建立可信的測試 Oracle。
7. 使用對拍或一般性質大量測試。
8. 將失敗輸入縮小，定位第一個錯誤狀態。
9. 修正後保留案例，避免相同問題再次出現。

### 適用讀者

- 程式能通過 Sample，但常在隱藏測資失敗的讀者。
- 能寫出解法，但不確定如何說明正確性的讀者。
- 遇到 Wrong Answer 時，只能反覆修改條件的讀者。
- 迴圈容易出現少一次、多一次或邊界錯誤的讀者。
- 遞迴程式偶爾發生無窮遞迴或 Stack Overflow 的讀者。
- 需要建立系統化測試與 Debug 流程的讀者。
- 同時使用 C++ 與 C，希望區分演算法推理與語言介面的讀者。

### 快速導覽

- [正確性到底要確認什麼](#41-正確性到底要確認什麼)：區分規格、部分正確性與終止性。
- [第一步：寫出 Precondition 與 Postcondition](#42-第一步寫出-precondition-與-postcondition)：先定義允許的輸入與應保證的輸出。
- [第二步：使用 Loop Invariant 說明迴圈](#43-第二步使用-loop-invariant-說明迴圈)：用 Initialization、Maintenance、Termination 建立推理。
- [完整案例：找出最大值](#44-完整案例找出最大值)：從規格、Invariant 到邊界測試。
- [第三步：確認遞迴正確性](#45-第三步確認遞迴正確性)：檢查 Base Case、問題縮小與答案組合。
- [完整案例：遞迴計算陣列總和](#46-完整案例遞迴計算陣列總和)：同時說明正確性與終止性。
- [第四步：從規格推導邊界案例](#47-第四步從規格推導邊界案例)：避免只憑直覺列測資。
- [第五步：建立測試 Oracle](#48-第五步建立測試-oracle)：決定如何判斷輸出是否正確。
- [第六步：使用對拍大量比較](#49-第六步使用對拍大量比較)：以直接解法檢查最佳化解法。
- [第七步：使用 Property-based Testing](#410-第七步使用-property-based-testing)：測試一般性質，而非只比固定答案。
- [第八步：縮小失敗案例](#411-第八步縮小失敗案例)：找出第一個錯誤狀態。
- [建立自己的正確性與測試表](#412-建立自己的正確性與測試表)：形成固定驗證流程。
- [常見問題與判讀](#413-常見問題與判讀)：整理常見失敗現象與檢查方向。
- [本章檢查表](#414-本章檢查表)：確認必要工作是否完成。
- [本章重點](#415-本章重點)：回顧核心方法。

### 4.1 正確性到底要確認什麼

假設題目如下：

給定一組非空整數，回傳其中的最大值。

看到這個題目後，可以很快寫出一個迴圈，但「程式看起來合理」和「程式對所有合法輸入都正確」是兩件不同的事。

完整的正確性至少包含以下內容：

<table>
<tr><th>項目</th><th>要回答的問題</th></tr>
<tr><td>Precondition</td><td>函式可以假設輸入滿足哪些條件？</td></tr>
<tr><td>Postcondition</td><td>函式結束時一定要保證什麼？</td></tr>
<tr><td>部分正確性</td><td>如果函式正常結束，回傳結果是否符合 Postcondition？</td></tr>
<tr><td>終止性</td><td>函式是否一定會在有限步驟後結束？</td></tr>
<tr><td>邊界行為</td><td>最小輸入、極端數值與特殊排列是否仍符合規格？</td></tr>
</table>

#### 部分正確性與完整正確性

部分正確性表示：

> 如果演算法結束，它產生的答案是正確的。

這仍然沒有排除無窮迴圈。完整正確性還需要終止性：

> 演算法不只在結束時答案正確，而且對每個符合 Precondition 的輸入都一定會結束。

例如下面的程式即使 `answer` 一直保存正確資訊，也因為 `i` 沒有增加而不會終止：

```cpp
int i = 0;
while (i < n)
{
    answer += values[i];
    // 遺漏 ++i
}
```

所以分析迴圈時，需要同時確認：
- 目前狀態是否維持正確。
- 控制變數是否朝結束條件前進。

#### 通過 Sample 代表什麼

通過 Sample 只能表示：
- 對這幾組輸入，程式輸出和題目提供的答案一致。

它不能直接推出：
- 所有合法輸入都正確。
- 沒有 Overflow。
- 空輸入或單一元素正確。
- 大型資料一定能在時間限制內結束。
- 多答案題目的回傳內容符合全部限制。

測試是用具體資料尋找錯誤；證明則是用規格與推理涵蓋整個合法輸入範圍。實務上兩者應一起使用。

### 4.2 第一步：寫出 Precondition 與 Postcondition

#### Precondition 是呼叫前必須成立的條件

以最大值函式為例：

```cpp
int maximum(const std::vector<int>& nums);
```

若函式直接讀取 `nums[0]`，就需要以下 Precondition：
- `nums` 至少包含一個元素。

如果題目允許空輸入，介面便需要另外定義空輸入結果，例如：
- 回傳 `std::optional<int>`。
- 使用例外回報無效輸入。
- 回傳成功狀態，並透過輸出參數帶回答案。

Precondition 不是用來忽略問題，而是用來清楚劃分責任。如果輸入不符合 Precondition，函式的行為必須由介面約定決定。

#### Postcondition 是函式結束後必須成立的條件

最大值函式不能只寫成「回傳最大值」，可以更精確地描述為：

令回傳值為 `answer`，則：
1. `answer` 等於輸入中的某個元素。
2. 對每個輸入元素 `value`，都有 `answer >= value`。

第一點確認答案確實來自輸入；第二點確認沒有其他元素比它更大。兩個條件合起來才完整描述最大值。

#### 不要把實作方法寫進 Postcondition

以下敘述不是合適的 Postcondition：
- 使用 Hash Set 找到重複值。
- 先排序再比較相鄰元素。
- 使用兩層迴圈檢查 Pair。

這些是方法，不是結果規格。同一個 Postcondition 可以由不同演算法完成。

例如「判斷是否有重複值」的 Postcondition 可以寫成：
- 回傳 `true`，若且唯若存在不同位置 `i` 與 `j`，使 `nums[i] == nums[j]`。

這個敘述沒有綁定 Hash Set、排序或直接列舉，因此可以用來比較不同解法是否完成同一件事。

#### C 語言介面的 Precondition

C 的 Array 參數不會自動帶入長度：

```c
int maximum(const int values[], size_t length);
```

這個介面通常需要以下 Precondition：
- `values` 指向至少 `length` 個有效 `int`。
- `length > 0`。

若要讓函式自行處理空輸入，可以改成：

```c
bool maximum(
    const int values[],
    size_t length,
    int *result);
```

此介面可約定：
- `result` 不可為 `NULL`。
- 找到最大值時回傳 `true`，並寫入 `*result`。
- `length == 0` 時回傳 `false`，且呼叫端不應讀取 `*result`。

演算法核心沒有改變，差異在於介面如何表達「可能沒有答案」。

### 4.3 第二步：使用 Loop Invariant 說明迴圈

Loop Invariant 是在迴圈固定位置上，每一輪都成立的敘述。它用來說明：迴圈雖然不斷改變變數，但某個和答案有關的正確關係始終被保留。

#### Invariant 不是單純描述變數型別

以下敘述資訊不足：
- `i` 是目前 Index。
- `answer` 是一個整數。
- `seen` 是一個 Set。

較有用的 Invariant 會連結：
- 已處理的輸入範圍。
- 目前保存的 State。
- State 所代表的數學或邏輯意義。

例如：
- 每輪開始前，`seen` 恰好包含 `nums[0, i)` 中出現過的值。
- 每輪結束後，`answer` 是 `nums[0, i]` 的最大值。
- 每輪開始前，`sum` 等於前 `i` 個元素的總和。

#### 三段式推理

使用 Invariant 說明正確性時，通常分成三部分。

##### Initialization

確認第一次進入迴圈前，Invariant 成立。

例如 `answer = nums[0]`，第一輪處理 `i = 1` 前：
- 已處理範圍只有 `nums[0]`。
- `answer` 等於 `nums[0]`。
- 因此 `answer` 是已處理範圍的最大值。

##### Maintenance

假設某一輪開始前 Invariant 成立，確認執行本輪後仍成立。

如果原本 `answer` 是 `nums[0, i)` 的最大值，本輪執行：

```cpp
answer = std::max(answer, nums[i]);
```

更新後 `answer` 會是：
- 原本已處理元素的最大值。
- 與新元素 `nums[i]`。
- 兩者之中的較大值。

因此更新後，`answer` 是 `nums[0, i]` 的最大值。

##### Termination

迴圈結束時，把 Invariant 和結束條件合起來，推出 Postcondition。

當 `i == nums.size()` 時，所有元素都已處理。由 Invariant 可得：
- `answer` 是整個 `nums` 的最大值。

這正是 Postcondition。

#### Invariant 應放在哪個時間點

同一個迴圈可以用「每輪開始前」或「每輪結束後」描述，但區間必須一致。

例如：
- 每輪開始前，`answer` 是 `[0, i)` 的最大值。
- 每輪結束後，`answer` 是 `[0, i]` 的最大值。

兩種都可使用。常見錯誤是文字寫 `[0, i)`，推理時卻把 `nums[i]` 當成已處理元素。

### 4.4 完整案例：找出最大值

#### 問題規格

輸入：一組非空整數。

輸出：其中的最大值。

範例：

```text
輸入：[4, 2, 7, 1]
輸出：7
```

#### Precondition

- `nums.size() > 0`。

#### Postcondition

令回傳值為 `answer`：
- `answer` 是 `nums` 中的某個元素。
- 對每個 `value` in `nums`，`answer >= value`。

#### C++ 解法

```cpp
#include <algorithm>
#include <vector>

int maximum(const std::vector<int>& nums)
{
    // Precondition：nums 至少包含一個元素。
    int answer = nums[0];

    for (std::size_t i = 1; i < nums.size(); ++i)
    {
        answer = std::max(answer, nums[i]);
    }

    return answer;
}
```

#### Invariant

每輪開始前：

> `answer` 是 Half-open Interval `[0, i)` 中所有元素的最大值。

#### Initialization

第一次進入迴圈時 `i == 1`：
- `[0, 1)` 只有 `nums[0]`。
- `answer` 被初始化為 `nums[0]`。
- 因此 Invariant 成立。

#### Maintenance

假設本輪開始前：
- `answer` 是 `[0, i)` 的最大值。

本輪加入 `nums[i]`，並執行：

```cpp
answer = std::max(answer, nums[i]);
```

更新後：
- 如果 `nums[i]` 較大，`answer` 變成 `nums[i]`。
- 否則保留原本最大值。

因此 `answer` 成為 `[0, i + 1)` 的最大值。下一輪 `i` 增加一，Invariant 仍成立。

#### Termination

迴圈在 `i == nums.size()` 時結束。此時 `[0, i)` 就是整個輸入範圍。由 Invariant 可得：
- `answer` 是所有輸入元素的最大值。

因此 Postcondition 成立。

#### 終止性

- `i` 從 1 開始。
- 每輪增加 1。
- `nums.size()` 是固定有限值。
- 當 `i == nums.size()` 時停止。

所以迴圈一定在有限輪數後結束。

#### 逐輪執行

輸入：`[4, 2, 7, 1]`

<table>
<tr><th>進入本輪時 i</th><th>已處理範圍</th><th>更新前 answer</th><th>目前元素</th><th>更新後 answer</th></tr>
<tr><td>1</td><td>[4]</td><td>4</td><td>2</td><td>4</td></tr>
<tr><td>2</td><td>[4, 2]</td><td>4</td><td>7</td><td>7</td></tr>
<tr><td>3</td><td>[4, 2, 7]</td><td>7</td><td>1</td><td>7</td></tr>
</table>

迴圈結束後，所有元素均已處理，`answer == 7`。

#### 邊界案例

<table>
<tr><th>輸入</th><th>目的</th><th>預期結果</th></tr>
<tr><td>[5]</td><td>最小合法長度</td><td>5</td></tr>
<tr><td>[-7, -2, -9]</td><td>全部為負數</td><td>-2</td></tr>
<tr><td>[3, 3, 3]</td><td>全部相同</td><td>3</td></tr>
<tr><td>[9, 4, 1]</td><td>最大值在左邊界</td><td>9</td></tr>
<tr><td>[1, 4, 9]</td><td>最大值在右邊界</td><td>9</td></tr>
<tr><td>[INT_MIN, INT_MAX]</td><td>極端整數</td><td>INT_MAX</td></tr>
</table>

這個函式只比較整數，不做加法或乘法，因此 `INT_MIN` 與 `INT_MAX` 本身不會造成算術 Overflow。

#### 常見錯誤：把 answer 初始化為 0

```cpp
int answer = 0;
```

若輸入全部為負數，例如 `[-7, -2, -9]`，函式會錯誤回傳 0，而 0 甚至不在輸入中。

這個錯誤同時破壞兩件事：
- Initialization 無法保證 `answer` 是已處理範圍的最大值。
- Postcondition 中「答案來自輸入」不成立。

使用 `nums[0]` 初始化，是直接從 Precondition 與 Invariant 推導出的設計。

#### C 語言版本

```c
#include <stdbool.h>
#include <stddef.h>

bool maximum(
    const int values[],
    size_t length,
    int *result)
{
    if (values == NULL || result == NULL || length == 0)
    {
        return false;
    }

    int answer = values[0];

    for (size_t i = 1; i < length; ++i)
    {
        if (values[i] > answer)
        {
            answer = values[i];
        }
    }

    *result = answer;
    return true;
}
```

這個版本使用回傳值表示函式是否成功，使用 `*result` 輸出最大值。成功路徑中的 Invariant 與 C++ 版本相同。

### 4.5 第三步：確認遞迴正確性

遞迴正確性通常需要回答三個問題：
1. Base Case 是否直接正確？
2. 每次遞迴呼叫是否處理更小的問題？
3. 假設較小問題的答案正確，目前層是否能組合出正確答案？

#### Base Case

Base Case 是不再遞迴、可以直接回答的最小問題。

例如計算前 `n` 個元素的總和：
- 當 `n == 0` 時，空 Prefix 的總和是 0。

Base Case 必須同時滿足：
- 回傳值符合該最小問題的 Postcondition。
- 所有合法遞迴路徑最終都能到達它。

#### 問題必須嚴格縮小

只寫出 Base Case 還不夠。每次呼叫都必須朝 Base Case 前進。

例如：

```cpp
return sumPrefix(nums, n) + nums[n - 1];
```

這裡仍呼叫相同的 `n`，問題沒有縮小，會持續遞迴。

正確方向應為：

```cpp
return sumPrefix(nums, n - 1) + nums[n - 1];
```

可以使用一個非負整數作為縮小量，例如：
- 尚未處理的元素數量。
- 搜尋區間長度。
- Tree 高度。
- 距離 Base Case 的差值。

每次呼叫時，此數值必須嚴格減少，且不能無限低於 0。

#### 假設較小問題正確

遞迴推理不是假設整個函式已經正確，而是：
- 假設函式對嚴格較小的輸入正確。
- 證明目前這一層可以利用較小答案得到正確結果。

這和數學歸納法的結構相近。

### 4.6 完整案例：遞迴計算陣列總和

#### 問題規格

給定整數陣列 `nums` 與數量 `n`，計算前 `n` 個元素的總和。

例如：

```text
nums = [3, 5, 2, 7]
n = 3
答案 = 3 + 5 + 2 = 10
```

#### Precondition

- `0 <= n <= nums.size()`。

#### Postcondition

回傳值等於：

```text
nums[0] + nums[1] + ... + nums[n - 1]
```

當 `n == 0` 時，回傳 0。

#### C++ 解法

```cpp
#include <cstddef>
#include <vector>

long long sumPrefix(
    const std::vector<int>& nums,
    std::size_t n)
{
    if (n == 0)
    {
        return 0;
    }

    return sumPrefix(nums, n - 1) + nums[n - 1];
}
```

#### Base Case 正確性

當 `n == 0`：
- 前 0 個元素形成空集合。
- 空集合總和定義為 0。
- 函式回傳 0。

因此 Base Case 符合 Postcondition。

#### 遞迴步驟正確性

假設 `sumPrefix(nums, n - 1)` 能正確回傳前 `n - 1` 個元素的總和：

```text
nums[0] + nums[1] + ... + nums[n - 2]
```

目前層再加上第 `n` 個元素 `nums[n - 1]`：

```text
前 n - 1 個元素總和 + nums[n - 1]
```

結果正好是前 `n` 個元素的總和，因此目前層符合 Postcondition。

#### 終止性

- `n` 是非負整數。
- 每次呼叫都從 `n` 變成 `n - 1`。
- 經過有限次呼叫後一定到達 `n == 0`。

因此遞迴一定終止。

#### 逐層展開

對 `nums = [3, 5, 2]`、`n = 3`：

```text
sumPrefix(nums, 3)
= sumPrefix(nums, 2) + 2
= sumPrefix(nums, 1) + 5 + 2
= sumPrefix(nums, 0) + 3 + 5 + 2
= 0 + 3 + 5 + 2
= 10
```

#### Overflow 仍需另外確認

雖然函式回傳 `long long`，但若元素很多或數值很大，總和仍可能超過 `long long` 範圍。正確性必須以題目給定的值域與數量為基礎，確認中間結果可以被回傳型別表示。

「演算法推理正確」不代表「有限寬度整數運算一定安全」。型別範圍也是實作正確性的一部分。

#### C 語言版本

```c
#include <stdbool.h>
#include <stddef.h>

bool sum_prefix(
    const int values[],
    size_t length,
    size_t n,
    long long *result)
{
    if (values == NULL || result == NULL || n > length)
    {
        return false;
    }

    if (n == 0)
    {
        *result = 0;
        return true;
    }

    long long previous = 0;
    if (!sum_prefix(values, length, n - 1, &previous))
    {
        return false;
    }

    *result = previous + values[n - 1];
    return true;
}
```

這個 C 版本為了檢查邊界，另外接收 `length`。在大量元素下，遞迴版本可能消耗較多 Call Stack；若題目允許很大的 `n`，迭代版本通常較穩定。

### 4.7 第四步：從規格推導邊界案例

邊界案例不只是列出「空、單一元素、最大值」。較穩定的方式是從規格中的每個限制推導測試。

#### 從輸入長度推導

若長度範圍是 `0 <= n <= 100000`，至少考慮：
- `n == 0`。
- `n == 1`。
- `n == 2`。
- 接近最大長度。
- 等於最大長度。

其中 `n == 2` 常用於檢查比較方向、左右邊界與迴圈是否少跑一輪。

#### 從數值範圍推導

若元素是 `int`，考慮：
- 0。
- 正數與負數。
- 全部為負數。
- `INT_MIN`。
- `INT_MAX`。
- 會讓加法或乘法接近型別上限的組合。

例如平均值若寫成：

```cpp
int middle = (left + right) / 2;
```

`left + right` 可能 Overflow。常見改寫為：

```cpp
int middle = left + (right - left) / 2;
```

但這個改寫仍依賴 `right - left` 可被型別表示，且通常假設 `left <= right`。

#### 從排列方式推導

對 Array 題目，可測試：
- 已遞增排序。
- 已遞減排序。
- 全部相同。
- 大量重複。
- 答案在第一個位置。
- 答案在最後一個位置。
- 答案出現在中間。

#### 從答案存在性推導

若題目可能無答案，需要測試：
- 剛好一個答案。
- 多個合法答案。
- 沒有答案。
- 答案位於規格邊界。

多答案題目還要確認：
- 題目接受任意合法答案，還是要求字典序最小、Index 最小或特定順序。

#### 從演算法分支推導

每個 `if`、`else`、提前 `return` 都應有案例經過。

例如 Binary Search：
- 第一次比較就找到。
- 持續往左縮小。
- 持續往右縮小。
- 最後一個候選才找到。
- 搜尋區間縮成空集合仍未找到。

#### 建立邊界測試表

<table>
<tr><th>規格來源</th><th>案例</th><th>要檢查的風險</th></tr>
<tr><td>最小長度</td><td>空輸入或單一元素</td><td>非法存取、初始化錯誤</td></tr>
<tr><td>最大長度</td><td>大量元素</td><td>時間、記憶體、Stack 深度</td></tr>
<tr><td>值域</td><td>INT_MIN、INT_MAX</td><td>Overflow、錯誤初始值</td></tr>
<tr><td>排序條件</td><td>遞增、遞減、重複</td><td>比較方向、提前停止</td></tr>
<tr><td>答案位置</td><td>第一個、最後一個</td><td>Off-by-one</td></tr>
<tr><td>答案數量</td><td>零個、一個、多個</td><td>無答案表示、多答案規格</td></tr>
</table>

### 4.8 第五步：建立測試 Oracle

Oracle 是判斷測試輸出是否正確的基準。產生很多輸入並不困難，真正重要的是知道每組輸入的正確答案。

常見 Oracle 包括：
- 人工可驗證的小型案例。
- 較慢但容易確認的直接解法。
- 已知數學性質。
- 另一份獨立完成的實作。
- 題目規格中的合法性檢查器。

#### 小型案例人工驗證

例如排序 `[3, 1, 2]`，預期結果可直接寫成 `[1, 2, 3]`。

優點：
- 容易建立。
- 適合測試已知邊界。

限制：
- 案例數量少。
- 人工預期值也可能寫錯。

#### 直接解法作為 Oracle

若最佳化解法複雜，可以保留一份只處理小型輸入的直接解法。

例如 Two Sum：
- Oracle 使用兩層迴圈列舉所有 Pair，時間 O(n²)。
- 最佳化版本使用 Hash Map，平均時間 O(n)。

在小型隨機資料上，O(n²) 足夠快，而且邏輯較容易檢查。

#### Oracle 也可能有錯

不要因為函式名稱叫 `bruteForce` 就自動相信它。Oracle 應具備：
- 邏輯獨立，避免和受測版本共用同一段可疑程式。
- 只使用容易驗證的步驟。
- 能完整檢查候選空間。
- 先以手動案例測試 Oracle 本身。

如果兩份實作共用相同的錯誤假設，對拍可能會同時得到錯誤答案。

#### 多答案問題的 Oracle

若題目接受多個答案，不一定能直接比較：

```cpp
expected == actual
```

例如題目要求回傳任意一組總和等於 Target 的 Pair。直接解法可能回傳 `(0, 3)`，最佳化解法可能回傳 `(1, 2)`，兩者都合法。

此時應檢查 Postcondition：
- 兩個 Index 是否不同。
- Index 是否在範圍內。
- 對應數值總和是否等於 Target。
- 無答案時是否確實不存在合法 Pair。

測試目標是確認結果符合規格，不是要求不同實作走完全相同的路徑。

### 4.9 第六步：使用對拍大量比較

對拍是用相同輸入執行兩份解法，並比較結果：
- 一份是容易確認的 Oracle。
- 一份是要驗證的目標解法。

#### 基本流程

```cpp
for (int test = 0; test < 10000; ++test)
{
    auto input = generateSmallRandomInput();

    auto expected = bruteForce(input);
    auto actual = optimized(input);

    if (!equivalent(expected, actual, input))
    {
        printFailure(input, expected, actual);
        break;
    }
}
```

#### 為什麼使用小型隨機輸入

直接解法通常較慢，所以輸入不需要很大。小型資料有三個優點：
- Oracle 可以完整列舉候選。
- 能在短時間執行大量測試。
- 失敗後較容易人工追蹤。

例如 Array 長度可隨機選在 0 到 10，元素值選在 -5 到 5。小值域會自然產生：
- 重複值。
- 負數。
- 0。
- 相同總和的不同 Pair。

#### 隨機種子要能重現

發現失敗後，必須能再次產生相同輸入。可採用：
- 使用固定 Seed。
- 失敗時印出 Seed。
- 直接印出完整輸入並保存。

若每次執行都改變 Seed，卻沒有記錄失敗資料，錯誤可能只出現一次，之後難以追蹤。

#### 不要只產生平均案例

隨機產生器如果只產生長度 50、數值均勻分布的資料，可能很少碰到真正的邊界。

可以混合兩類測試：
- 明確列出的邊界案例。
- 大量隨機案例。

也可以讓產生器刻意偏向：
- 空輸入。
- 單一元素。
- 全部相同。
- 已排序。
- 反向排序。
- 包含極端值。

#### 對拍結果不一致時

不要立刻假設最佳化版本有錯，也可能是：
- Oracle 有錯。
- `equivalent` 比較方式不符合多答案規格。
- 隨機輸入違反 Precondition。
- 兩份實作使用不同的區間定義。
- 測試程式本身修改了共用輸入。

第一步應先把失敗輸入、兩邊輸出和規格放在一起人工確認。

### 4.10 第七步：使用 Property-based Testing

有些函式很難為每個隨機輸入準備完整答案，但可以檢查輸出必須滿足的一般性質。這類方法稱為 Property-based Testing。

#### 排序的性質

排序後應滿足：
- 輸出長度與輸入相同。
- 輸出為非遞減順序。
- 輸入與輸出的元素 Multiset 相同。

只檢查順序仍不夠。錯誤程式若永遠輸出 `[1, 2, 3]`，結果可能有序，但沒有保留原始元素。

#### 反轉的性質

對任意序列 `x`：

```text
reverse(reverse(x)) == x
```

這個性質不需要事先知道 `reverse(x)` 的固定答案，也能檢查許多輸入。

#### Lower Bound 的性質

若 `position` 是排序序列中第一個不小於 `target` 的位置，應滿足：
- `position` 範圍是 `[0, n]`。
- 所有位於 `position` 前方的元素都小於 `target`。
- 若 `position < n`，則 `nums[position] >= target`。

這些條件直接來自 Postcondition。

#### BFS 距離的性質

從起點進行 BFS 後：
- 起點距離為 0。
- 每個可達節點距離為非負值。
- 對每條連接兩個可達節點的無權邊 `(u, v)`，距離差的絕對值不超過 1。
- 每個距離大於 0 的可達節點，應有一個距離少 1 的前驅節點。

單一性質不一定足以完整證明 BFS 結果，但多項性質能有效找出錯誤。

#### Metamorphic Relation

有時可以改變輸入，預測輸出之間的關係。例如：
- 對所有元素加上同一常數，最大值也應增加該常數，前提是沒有 Overflow。
- 將輸入順序打亂，不應改變「是否存在重複值」的答案。
- 對排序輸入加入一個比所有元素更大的值，新值應位於排序結果最後。

這種測試不一定知道單次輸出的完整答案，但知道兩次執行結果應滿足的關係。

### 4.11 第八步：縮小失敗案例

大型失敗輸入可能包含大量與錯誤無關的資料。縮小案例的目的是：
- 保留錯誤。
- 移除不影響錯誤的內容。
- 讓控制流程與狀態變化容易觀察。

#### 基本縮小方式

對 Array 或 Sequence：
- 刪除前半段或後半段。
- 一次刪除一個元素。
- 縮短重複區段。
- 將數值往 0、1、-1 靠近。
- 將數值替換成較小的代表值。
- 保留答案附近的元素。

對 Graph：
- 刪除不相關節點。
- 刪除不影響錯誤的 Edge。
- 縮短 Path。

對字串：
- 刪除 Prefix、Suffix 或中間區段。
- 減少字元種類。
- 將字元替換成較簡單的內容。

每次修改後都重新執行測試。若錯誤仍存在，就保留較小版本；若錯誤消失，就還原該次修改。

#### 記錄最小失敗案例

建議至少保存：

```text
最小輸入：
Precondition：
預期輸出：
實際輸出：
第一個錯誤 iteration 或遞迴層：
該輪開始前的 State：
本輪更新：
更新後的 State：
被破壞的 Invariant 或 Postcondition：
```

#### 找第一個錯誤狀態

最終答案錯誤只是結果，真正有用的是找出狀態第一次偏離正確條件的位置。

例如 Binary Search 最後回傳 `-1`，不要只看最後一輪。可以逐輪記錄：

```text
left, right, middle, nums[middle], target
```

並檢查每輪 Invariant：
- 若 Target 存在，它是否仍位於目前候選區間？

第一輪把可能答案錯誤排除時，通常就是較接近問題的位置。

#### 從失敗案例建立 Regression Test

修正後，不要刪除最小失敗案例。應將它加入固定測試集合：
- 先確認舊版本會失敗。
- 修正後確認它會通過。
- 重新執行全部既有測試。

這樣可以避免後續修改讓相同錯誤再次出現，也能檢查修正是否影響其他情況。

### 4.12 建立自己的正確性與測試表

完成一題後，可以填寫以下內容：

<table>
<tr><th>欄位</th><th>要回答的問題</th></tr>
<tr><td>Precondition</td><td>哪些輸入是合法的？是否允許空輸入、負數或重複值？</td></tr>
<tr><td>Postcondition</td><td>回傳值或輸出資料必須滿足哪些條件？</td></tr>
<tr><td>終止性</td><td>哪個量會嚴格前進或縮小？</td></tr>
<tr><td>Loop Invariant</td><td>每輪固定位置上，State 代表哪段已處理資料？</td></tr>
<tr><td>Initialization</td><td>第一次進入迴圈前，Invariant 為何成立？</td></tr>
<tr><td>Maintenance</td><td>本輪更新後，Invariant 為何仍成立？</td></tr>
<tr><td>Termination</td><td>結束條件加上 Invariant，如何推出 Postcondition？</td></tr>
<tr><td>遞迴 Base Case</td><td>最小問題是否直接正確？</td></tr>
<tr><td>遞迴縮小量</td><td>每次呼叫如何更接近 Base Case？</td></tr>
<tr><td>邊界案例</td><td>長度、值域、排列、答案位置與答案數量有哪些邊界？</td></tr>
<tr><td>Oracle</td><td>用人工答案、直接解法、數學性質或合法性檢查器？</td></tr>
<tr><td>對拍策略</td><td>如何產生小型輸入？如何保存 Seed 與失敗資料？</td></tr>
<tr><td>一般性質</td><td>輸出一定滿足哪些不變性或輸入輸出關係？</td></tr>
<tr><td>最小失敗案例</td><td>如何刪除資料或縮小數值，同時保留錯誤？</td></tr>
<tr><td>Regression</td><td>修正後要保留哪些案例並重新執行哪些測試？</td></tr>
</table>

這張表不要求每題都寫成正式證明。練習初期完整填寫，可以讓「我覺得應該對」逐步轉成可檢查的規格、狀態與測試依據。

### 4.13 常見問題與判讀

<table>
<tr><th>現象</th><th>可能原因</th><th>第一輪檢查</th></tr>
<tr><td>Sample 通過但隱藏測資失敗</td><td>邊界或特殊排列未涵蓋</td><td>空輸入、單一元素、重複、無答案、極端值</td></tr>
<tr><td>只在全部負數時失敗</td><td>最大值初始化為 0</td><td>初始 State 是否滿足 Invariant</td></tr>
<tr><td>迴圈少處理最後一個元素</td><td>區間與結束條件不一致</td><td>使用 `[left, right]` 還是 `[left, right)`</td></tr>
<tr><td>無窮迴圈</td><td>控制 State 未嚴格前進</td><td>Pointer、Index 或搜尋區間是否每輪縮小</td></tr>
<tr><td>Stack Overflow</td><td>遞迴未到達 Base Case或深度過大</td><td>縮小量、最大深度與迭代替代方案</td></tr>
<tr><td>多答案對拍失敗</td><td>比較方式要求完全相同</td><td>改為檢查 Postcondition 與合法性</td></tr>
<tr><td>只在大數失敗</td><td>中間運算 Overflow</td><td>運算發生前的型別與最大可能值</td></tr>
<tr><td>隨機測試偶爾失敗但無法重現</td><td>未保存 Seed 或輸入</td><td>固定 Seed 並印出完整失敗資料</td></tr>
<tr><td>對拍兩邊結果相同但仍是錯的</td><td>共用相同錯誤假設</td><td>檢查 Oracle 是否獨立並加入人工案例</td></tr>
<tr><td>修正一個案例後另一個案例失敗</td><td>修正只針對單一輸入或缺少 Regression</td><td>重新檢查 Invariant，執行全部舊案例</td></tr>
<tr><td>最後輸出錯誤但找不到原因</td><td>只觀察最終結果</td><td>尋找第一個違反 Invariant 的 iteration</td></tr>
<tr><td>測試很多仍不放心</td><td>測試沒有從規格推導</td><td>建立 Precondition、Postcondition 與邊界分類</td></tr>
</table>

### 4.14 本章檢查表

- 我能明確寫出函式的 Precondition。
- 我能用結果條件描述 Postcondition，而不是描述實作方法。
- 我知道通過 Sample 不能代替完整正確性。
- 我能區分部分正確性與終止性。
- 我能指出迴圈中嚴格前進或縮小的 State。
- 我能說明 Invariant 對應「每輪開始前」還是「每輪結束後」。
- 我能用 Initialization、Maintenance、Termination 說明迴圈。
- 我能確認遞迴的 Base Case 直接正確。
- 我能指出每次遞迴呼叫如何處理更小的問題。
- 我能說明較小問題的答案如何組合成目前答案。
- 我會從長度、值域、排列、答案位置與答案數量推導邊界案例。
- 我會檢查中間運算，而不只檢查回傳型別是否足夠大。
- 我有明確且可檢查的測試 Oracle。
- 我知道多答案問題應檢查 Postcondition，不一定比較完全相同的輸出。
- 我能使用小型隨機資料進行對拍。
- 我會保存隨機 Seed、完整輸入、預期輸出與實際輸出。
- 我能列出受測函式必須滿足的一般性質。
- 我能將大型失敗輸入縮小成容易追蹤的案例。
- 我會尋找第一個錯誤狀態，而不只觀察最終輸出。
- 修正後，我會保留最小失敗案例並重新執行既有測試。

### 4.15 本章重點

- 正確性以 Precondition 與 Postcondition 為起點。
- 完整正確性同時包含結果正確與演算法一定終止。
- Sample 只能驗證少數具體輸入，不能涵蓋所有合法資料。
- Loop Invariant 應連結已處理範圍、目前 State 與其代表的意義。
- Initialization、Maintenance、Termination 可以系統化說明迴圈正確性。
- 遞迴需要正確的 Base Case、嚴格縮小的問題，以及正確的答案組合方式。
- 邊界案例應從輸入長度、值域、排列、答案位置與分支條件推導。
- Oracle 必須容易確認且盡量獨立，不能只因名稱是直接解法就自動可信。
- 多答案題目應驗證結果是否符合 Postcondition。
- 對拍適合使用直接解法檢查小型隨機資料，失敗時必須能重現輸入。
- Property-based Testing 可驗證排序性、Multiset、可逆性與其他一般關係。
- Debug 時應找出第一個違反 Invariant 的狀態。
- 最小失敗案例應保留為 Regression Test，避免相同問題再次出現。
