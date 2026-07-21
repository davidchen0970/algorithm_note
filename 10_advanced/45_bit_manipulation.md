# 第 45 章　Bit Manipulation

## 適用範圍

本章假設讀者已經會：

- 整數變數。
- `if`、`for`、`while`。
- 基本函式。
- 十進位加減乘除。

本章不假設讀者已經熟悉 Binary、Bitmask、Submask 或 Bitmask DP。學習順序會從「數字如何寫成 0 和 1」開始，每次只增加一種運算。

第一次閱讀的目標只有：

1. 看懂 Binary 與 Bit Position。
2. 分清 `&`、`|`、`^`。
3. 會檢查、設定、清除與切換一個 Bit。
4. 看懂 Single Number 為什麼可以使用 XOR。

集合、Submask 與 Bitmask DP 放在後半章。第一次看不懂可以先跳過。

## 閱讀方式

### 第一輪必讀

- 45.1 Binary 是什麼。
- 45.2 Bit Position 與 Shift。
- 45.3 AND、OR、XOR。
- 45.4 檢查、設定、清除與切換。
- 45.5 完整案例：Single Number。

### 第二輪再讀

- 45.6 常見小技巧。
- 45.7 Bitmask 表示集合。
- 45.8 枚舉所有集合。

### 第三輪延伸

- 45.9 Submask Enumeration。
- 45.10 Bitmask DP。
- 45.11 C++ 型別與 Shift 邊界。

## 快速導覽

- [45.1 Binary 是什麼](#451-binary-是什麼)
- [45.2 Bit Position 與 Shift](#452-bit-position-與-shift)
- [45.3 AND、OR、XOR 與 NOT](#453-andorxor-與-not)
- [45.4 檢查、設定、清除與切換](#454-檢查設定清除與切換)
- [45.5 完整案例：Single Number](#455-完整案例single-number)
- [45.6 常見小技巧](#456-常見小技巧)
- [45.7 Bitmask 表示集合](#457-bitmask-表示集合)
- [45.8 枚舉所有集合](#458-枚舉所有集合)
- [45.9 延伸：枚舉 Submask](#459-延伸枚舉-submask)
- [45.10 延伸：Bitmask DP](#4510-延伸bitmask-dp)
- [45.11 C++ 型別與 Shift 邊界](#4511-c-型別與-shift-邊界)
- [45.12 固定分析流程](#4512-固定分析流程)
- [45.13 常見問題與判讀](#4513-常見問題與判讀)
- [45.14 本章檢查表](#4514-本章檢查表)
- [45.15 本章重點](#4515-本章重點)


## 45.1 Binary 是什麼

我們平常使用十進位，每一位的權重是：

```text
1, 10, 100, 1000, ...
```

Binary 只有 0 和 1，每一位的權重是：

```text
1, 2, 4, 8, 16, ...
```

也就是 2 的次方。

### 用 13 建立直覺

```text
13 = 8 + 4 + 1
```

對應到 Binary：

```text
權重： 8 4 2 1
Bit：  1 1 0 1
```

所以：

```text
13 = 1101₂
```

右下角的 ₂ 表示這是二進位。

### Bit Position 從右邊的 0 開始

```text
Bit Position： 3 2 1 0
Bit：          1 1 0 1
權重：         8 4 2 1
```

因此 13 的：

- Bit 0 是 1。
- Bit 1 是 0。
- Bit 2 是 1。
- Bit 3 是 1。

### 如何把十進位拆成 Binary

可以不斷除以 2，記錄餘數。

以 13 為例：

```text
13 / 2 = 6，餘 1
 6 / 2 = 3，餘 0
 3 / 2 = 1，餘 1
 1 / 2 = 0，餘 1
```

將餘數由下往上讀：

```text
1101
```

### 固定寬度比較容易閱讀

雖然 13 可以寫成：

```text
1101
```

剛開始練習時，建議補成 8-bit：

```text
00001101
```

之後看 NOT、清除 Bit 或 XOR 時會比較清楚。

### 先練三個數字

```text
5  = 0101₂
8  = 1000₂
10 = 1010₂
```

若這一節還不熟，先停在這裡。後面的 Bit 運算都是逐欄處理這些 0 和 1。


## 45.2 Bit Position 與 Shift

### `1 << i` 是什麼

`<<` 表示向左移動 Bit。

```text
1 << 0 = 0001
1 << 1 = 0010
1 << 2 = 0100
1 << 3 = 1000
```

所以：

```cpp
1U << i
```

可以建立一個只有 Bit `i` 為 1 的 Mask。

### Mask 是什麼

Mask 可以先理解成「拿來指定位置的數字」。

如果想處理 Bit 2：

```text
1U << 2 = 0100
```

這個 `0100` 只指出 Bit 2，其他位置都是 0。

### Left Shift

對非負且不超出型別範圍的數字，左移一位常像乘以 2：

```text
0011 = 3
0110 = 6
```

```cpp
3U << 1  // 6
```

但 Bit 題使用 Shift 的主要目的不是代替乘法，而是建立或移動 Bit Pattern。

### Right Shift

對非負整數，右移一位常像除以 2 並捨去餘數：

```text
1000 >> 1 = 0100
0100 >> 1 = 0010
```

### 第一輪先使用小範圍 unsigned

以下範例主要使用：

```cpp
unsigned
```

並假設 `i` 位於合法範圍。型別寬度、負數與過大 Shift 會在 45.11 再說明。


## 45.3 AND、OR、XOR 與 NOT

先不要一次背四個符號。逐一理解它們如何處理同一欄的兩個 Bits。

### AND：兩邊都是 1，結果才是 1

符號：

```cpp
&
```

規則：

```text
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1
```

例子：

```text
  1101
& 0101
---  0101
```

AND 常用來：

- 檢查某個 Bit。
- 只保留指定位置。
- 清除某些位置。

可以把 AND 想成：

> 只有兩邊都允許的位置才保留下來。

### OR：只要一邊是 1，結果就是 1

符號：

```cpp
|
```

規則：

```text
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1
```

例子：

```text
  1000
| 0101
---  1101
```

OR 常用來：

- 將指定 Bit 設成 1。
- 合併兩個 Bitmask。

可以把 OR 想成：

> 任一邊有選到，就保留。

### XOR：兩邊不同時，結果是 1

符號：

```cpp
^
```

規則：

```text
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0
```

例子：

```text
  1101
^ 0101
---  1000
```

XOR 最重要的兩個性質：

```text
x ^ x = 0
x ^ 0 = x
```

XOR 常用來：

- 切換某個 Bit。
- 消除成對出現的相同數字。
- 找出兩個狀態不同的位置。

### NOT：把每一個 Bit 反轉

符號：

```cpp
~
```

用 8-bit 示意：

```text
x  = 00001101
~x = 11110010
```

NOT 會反轉型別中的所有 Bits，不只反轉畫面上有寫出的四位。

因此在 C++ 中，對 signed 整數使用 `~` 時，結果可能看起來像負數。第一輪只需知道它可以配合 AND 清除某一個 Bit。

### 一句話整理

```text
AND：檢查或保留
OR：設成 1
XOR：切換或成對消除
NOT：全部反轉
```


## 45.4 檢查、設定、清除與切換

假設：

```text
mask = 1101₂
```

我們想處理 Bit `i`。

第一步都相同：

```cpp
const unsigned bit = 1U << i;
```

### 檢查 Bit 是否為 1

```cpp
const bool isSet = (mask & bit) != 0;
```

例如檢查 Bit 2：

```text
mask = 1101
bit  = 0100

1101 & 0100 = 0100
```

結果不是 0，所以 Bit 2 原本是 1。

檢查 Bit 1：

```text
mask = 1101
bit  = 0010

1101 & 0010 = 0000
```

結果是 0，所以 Bit 1 原本是 0。

### 設定 Bit 為 1

```cpp
mask |= bit;
```

若原本：

```text
mask = 1001
bit  = 0100
```

OR 後：

```text
1001 | 0100 = 1101
```

其他位置不變，Bit 2 被設成 1。

### 清除 Bit 為 0

```cpp
mask &= ~bit;
```

先看 Bit 2 的 Mask：

```text
bit  = 0100
~bit = 1011    // 只用 4-bit 示意
```

再 AND：

```text
mask = 1101

1101 & 1011 = 1001
```

Bit 2 被清成 0，其他位置保留。

### 切換 Bit

```cpp
mask ^= bit;
```

XOR 與 1 運算時會切換：

```text
0 ^ 1 = 1
1 ^ 1 = 0
```

所以：

- 原本是 0，切換後變 1。
- 原本是 1，切換後變 0。

### 四個小函式

```cpp
bool isSet(unsigned mask, int i)
{
    return (mask & (1U << i)) != 0;
}

unsigned setBit(unsigned mask, int i)
{
    return mask | (1U << i);
}

unsigned clearBit(unsigned mask, int i)
{
    return mask & ~(1U << i);
}

unsigned toggleBit(unsigned mask, int i)
{
    return mask ^ (1U << i);
}
```

這四個函式是第一輪最重要的內容。若還會混淆，先把每一次運算都寫成 8-bit 再逐欄計算。


## 45.5 完整案例：Single Number

### 問題

陣列中每個數字都出現兩次，只有一個數字出現一次。找出這個數字。

```text
[4, 1, 2, 1, 2]
```

答案是 4。

### 先用熟悉的方法思考

可以使用 Hash Map 計算每個數字出現次數，再找次數為 1 的數字。

Bit 解法則利用：

```text
x ^ x = 0
x ^ 0 = x
```

相同數字 XOR 後會消失。

### 逐步追蹤

```text
answer = 0

answer ^= 4  → 4
answer ^= 1  → 4 ^ 1
answer ^= 2  → 4 ^ 1 ^ 2
answer ^= 1  → 4 ^ (1 ^ 1) ^ 2
answer ^= 2  → 4 ^ 0 ^ (2 ^ 2)
             → 4
```

因為 XOR 可以交換順序與重新分組，成對數字最後都變成 0。

### C++ 解法

```cpp
#include <vector>

int singleNumber(const std::vector<int>& numbers)
{
    int answer = 0;

    for (int value : numbers)
    {
        answer ^= value;
    }

    return answer;
}
```

### 這個方法的前置條件

必須符合：

```text
其他數字都剛好出現兩次
只有一個數字出現一次
```

以下情況不能直接套用同一方法：

- 其他數字出現三次。
- 有兩個數字各出現一次。
- 每個數字出現次數不固定。

Bit 技巧通常依賴很明確的出現次數條件。先看規格，再決定能否使用 XOR。


## 45.6 常見小技巧

> 第一次閱讀不必全部背下來。先理解每個技巧依賴什麼 Bit 性質。

### 判斷奇偶

整數的最低位代表 1。

- 最低位為 1：奇數。
- 最低位為 0：偶數。

```cpp
bool isOdd(unsigned value)
{
    return (value & 1U) != 0;
}
```

### 清除最低位的 1

```cpp
value &= value - 1;
```

例子：

```text
value     = 1011000
value - 1 = 1010111
AND       = 1010000
```

最低位的 1 被清除。

### 為什麼 `value - 1` 有這種效果

從 Binary 最右邊看：

- 最低位的 1 會變成 0。
- 它右邊原本的 0 會變成 1。
- 更左邊保持不變。

再與原值 AND，就只會清掉最低位的 1。

### 判斷 Power of Two

Power of Two 的 Binary 只有一個 1：

```text
1  = 0001
2  = 0010
4  = 0100
8  = 1000
```

若清除最低位的 1 後變成 0，表示原本只有一個 1。

```cpp
bool isPowerOfTwo(unsigned value)
{
    return value != 0 &&
           (value & (value - 1)) == 0;
}
```

一定要排除 0，因為 0 不是 Power of Two。

### 計算 1 的數量

```cpp
int countBits(unsigned value)
{
    int count = 0;

    while (value != 0)
    {
        value &= value - 1;
        ++count;
    }

    return count;
}
```

每輪清除一個 1，所以執行次數就是 1 的數量。

C++20 也可以使用：

```cpp
#include <bit>

const int count = std::popcount(value);
```

### 取得最低位的 1

對 unsigned 整數：

```cpp
const unsigned lowbit = value & (~value + 1U);
```

例如：

```text
value  = 1011000
lowbit = 0001000
```

這個技巧常出現在 Fenwick Tree。第一次閱讀只需知道它能取出最低位的 1，不必立刻背下推導。


## 45.7 Bitmask 表示集合

假設有四個元素：

```text
元素 0, 1, 2, 3
```

每個元素只有兩種狀態：

```text
未選或已選
```

可以讓每個 Bit 代表一個元素：

```text
Bit 0 → 元素 0
Bit 1 → 元素 1
Bit 2 → 元素 2
Bit 3 → 元素 3
```

### 範例

```text
mask = 0101₂
```

表示：

- Bit 0 是 1，元素 0 已選。
- Bit 1 是 0，元素 1 未選。
- Bit 2 是 1，元素 2 已選。
- Bit 3 是 0，元素 3 未選。

所以集合是：

```text
{0, 2}
```

### 集合中的四種基本動作

```cpp
bool contains(unsigned mask, int element)
{
    return (mask & (1U << element)) != 0;
}

unsigned addElement(unsigned mask, int element)
{
    return mask | (1U << element);
}

unsigned removeElement(unsigned mask, int element)
{
    return mask & ~(1U << element);
}

unsigned toggleElement(unsigned mask, int element)
{
    return mask ^ (1U << element);
}
```

這其實就是 45.4 的四種 Bit 動作，只是現在每個 Bit 有「元素是否在集合中」的語意。

### 集合聯集、交集與差集

```cpp
const unsigned unionSet = first | second;
const unsigned intersectionSet = first & second;
const unsigned differenceSet = first & ~second;
```

- OR：任一集合有該元素，就放入聯集。
- AND：兩個集合都有，才放入交集。
- `first & ~second`：保留只在 `first`、不在 `second` 的元素。

### Full Mask

若有 `n` 個元素，全選的 Mask 是：

```cpp
const unsigned fullMask = (1U << n) - 1U;
```

例如 `n = 4`：

```text
1 << 4 = 10000
減 1    = 01111
```

前提是 `n` 小於型別的 Bit 數。型別邊界在 45.11 說明。


## 45.8 枚舉所有集合

若有 `n` 個元素，每個元素都有選或不選兩種可能，因此集合總數是：

```text
2^n
```

### `n = 3` 的全部 Masks

```text
000 → {}
001 → {0}
010 → {1}
011 → {0, 1}
100 → {2}
101 → {0, 2}
110 → {1, 2}
111 → {0, 1, 2}
```

### 枚舉所有 Masks

```cpp
for (unsigned mask = 0;
     mask < (1U << n);
     ++mask)
{
    // 處理集合 mask
}
```

### 枚舉 Mask 中選到的元素

```cpp
for (int element = 0; element < n; ++element)
{
    if ((mask & (1U << element)) != 0)
    {
        // element 在集合中
    }
}
```

### 複雜度

- 所有 Masks：O(2^n)。
- 每個 Mask 再檢查 `n` 個元素：O(n × 2^n)。

Bit 運算本身很快，不代表全部集合數量很少。

例如：

```text
2^10  = 1,024
2^20  = 1,048,576
2^30  約 10 億
```

使用 Bitmask 前要先估算 `n`。


## 45.9 延伸：枚舉 Submask

> 第一次閱讀可以跳過。先確定自己會枚舉全部 Masks。

假設：

```text
mask = 1101₂
```

它的 Submask 只能使用原 Mask 中為 1 的位置。

常見寫法：

```cpp
for (unsigned submask = mask;
     submask != 0;
     submask = (submask - 1U) & mask)
{
    // 處理非空 submask
}
```

### 為什麼最後還要 `& mask`

`submask - 1` 可能將右側一些位置變成 1，其中可能包含原 Mask 不允許的位置。

再 AND 原 Mask，就會把不屬於原 Mask 的位置清除。

### 若要包含空集合

```cpp
unsigned submask = mask;

while (true)
{
    // 處理 submask

    if (submask == 0)
    {
        break;
    }

    submask = (submask - 1U) & mask;
}
```

### 複雜度提醒

單一 Mask 的 Submask 數量是 `2^k`，其中 `k` 是 Mask 中 1 的個數。

若枚舉所有 Masks 的所有 Submasks，總量是 O(3^n)。這是進階用法，不要因為程式只有一行更新式就忽略總狀態數量。


## 45.10 延伸：Bitmask DP

> 若尚未學過 Dynamic Programming，可以先跳過。

Bitmask DP 適合：

- `n` 不大。
- 每個元素只有選或不選。
- DP State 需要記住目前已選集合。

### 最基本的 State

```text
dp[mask] = 已選集合為 mask 時的答案
```

例如：

- 已選這些工作時的最小成本。
- 已拜訪這些城市時的最佳路徑。
- 已使用這些資源時的方法數。

### 加入一個尚未選的元素

```cpp
for (int next = 0; next < n; ++next)
{
    if ((mask & (1U << next)) == 0)
    {
        const unsigned nextMask =
            mask | (1U << next);

        // 由 dp[mask] 更新 dp[nextMask]
    }
}
```

這裡只使用前面已學過的兩件事：

- AND 檢查元素是否已選。
- OR 加入元素。

### 狀態數量

```text
2^n
```

若 State 還包含最後位置：

```text
dp[mask][last]
```

狀態數量可能是：

```text
2^n × n
```

若每個 State 再枚舉下一個元素，時間可能到：

```text
O(2^n × n²)
```

Bitmask DP 的第一步不是寫轉移，而是先確認 `2^n` 是否能接受。


## 45.11 C++ 型別與 Shift 邊界

這一節是安全補充。第一次閱讀先使用小型非負數，再回來處理型別細節。

### `1 << i` 的 `1` 是 int

```cpp
1 << i
```

左邊的 `1` 是 `int`。若 `i` 很大，可能超出 `int` 能安全表示的範圍。

依需求使用：

```cpp
1U << i
1ULL << i
```

- `1U`：unsigned。
- `1ULL`：unsigned long long。

### Shift 量必須小於型別寬度

若型別是 64-bit：

```cpp
1ULL << i
```

需要：

```cpp
i < 64
```

`1ULL << 64` 不是合法的 64-bit Bit Mask 建立方式。

### 優先使用 unsigned 表達 Bitmask

signed 整數涉及符號位、負數與右移語意。若資料本來就是旗標或集合狀態，使用 unsigned 型別通常比較清楚。

### 加上括號

建議：

```cpp
if ((mask & (1U << i)) != 0)
{
}
```

括號能直接表達：

1. 先建立 Bit Mask。
2. 再做 AND。
3. 最後和 0 比較。

### `~` 會反轉所有 Bits

```cpp
~(1U << i)
```

不是只產生畫面上看到的幾位，而是依 unsigned 型別寬度反轉全部 Bits。這正是清除 Bit 時需要的結果：只有目標位置為 0，其他位置為 1。


## 45.12 固定分析流程

遇到 Bit 題時，依序回答以下問題。

### 第一步：每個 Bit 代表什麼

例如：

- 權限是否開啟。
- 元素是否已選。
- 某個方向是否可用。
- DP 中某個項目是否已處理。

如果沒有明確語意，Bitmask 很難檢查。

### 第二步：Bit 編號從哪裡開始

本章使用：

```text
最右邊是 Bit 0
```

### 第三步：想做哪種動作

```text
檢查 → AND
設定 → OR
清除 → AND + NOT
切換 → XOR
```

### 第四步：先畫小型 Binary

例如：

```text
mask = 00001101
bit  = 00000100
```

先逐欄確認，再寫程式。

### 第五步：確認型別與最大 Bit

- `unsigned` 是否足夠？
- 是否需要 `unsigned long long`？
- `i` 是否小於型別 Bit 數？

### 第六步：檢查技巧的前置條件

- Single Number：其他數是否恰好出現兩次？
- Power of Two：是否排除 0？
- Full Mask：`n` 是否小於型別寬度？
- Bitmask DP：`2^n` 是否可接受？

### 第七步：先測最小數字

建議測試：

```text
0, 1, 2, 3, 4
```

以及：

- Bit 0。
- 最高允許 Bit。
- 空集合。
- Full Mask。


## 45.13 常見問題與判讀

### 檢查 Bit 的答案相反

先確認：

- 最右邊是否為 Bit 0。
- 是否建立 `1U << i`。
- AND 後是否使用 `!= 0` 判斷。

### Power of Two 將 0 判成 true

可能漏掉：

```cpp
value != 0
```

### 設定 Bit 時其他位置也改變

不要使用不確定是否進位的加法。設定 Bit 應使用：

```cpp
mask | (1U << i)
```

### 清除 Bit 後得到奇怪負數

可能對 signed 整數使用 `~`。Bitmask 優先使用 unsigned 型別。

### XOR 題答案錯誤

重新確認出現次數。`answer ^= value` 的 Single Number 解法要求其他數字恰好出現兩次。

### Shift 結果異常

檢查：

- 左側型別是 `1`、`1U` 還是 `1ULL`。
- Shift 量是否小於型別 Bit 數。
- 是否對負數使用 Shift。

### Full Mask 出錯

```cpp
(1U << n) - 1U
```

要求 `n` 小於 unsigned 的 Bit 數。若 `n` 較大，改用更寬型別，但仍需檢查上限。

### Bitmask DP 超時

單次 Bit 運算是 O(1)，但 State 數可能是 O(2^n)、O(n × 2^n) 或更高。先算狀態數，不要只看每行程式很短。

### Submask 少了空集合

```cpp
submask != 0
```

的迴圈不處理 0。若需要空集合，要另外處理或使用包含 `break` 的版本。


## 45.14 本章檢查表

### 第一輪

- 我能將小型十進位整數寫成 Binary。
- 我知道最右邊是 Bit 0。
- 我知道 `1U << i` 會建立第 `i` 個 Bit Mask。
- 我能用自己的話說明 AND、OR 與 XOR。
- 我能檢查指定 Bit。
- 我能設定指定 Bit。
- 我能清除指定 Bit。
- 我能切換指定 Bit。
- 我知道 Single Number 為何能消除成對數字。

### 第二輪

- 我知道奇偶可由最低位判斷。
- 我能說明 `value & (value - 1)` 清除最低位的 1。
- 我知道 Power of Two 判斷必須排除 0。
- 我能用 Bitmask 表示集合。
- 我能枚舉全部 `2^n` 個 Masks。
- 我知道 Bit 運算快，不代表指數狀態數量少。

### 第三輪

- 我知道如何枚舉非空 Submasks。
- 我知道如何額外處理空 Submask。
- 我知道 Bitmask DP 的 `mask` 必須有固定語意。
- 我會先估算 `2^n` 是否可接受。
- 我會檢查 unsigned 型別與 Shift 範圍。


## 45.15 本章重點

1. Binary 的每一位只有 0 或 1，權重依序為 1、2、4、8。
2. 最右邊是 Bit 0，`1U << i` 可建立第 `i` 個 Bit Mask。
3. AND 常用於檢查與保留，OR 常用於設定，XOR 常用於切換與成對消除。
4. NOT 會反轉型別中的所有 Bits。
5. 檢查、設定、清除與切換是最重要的四種基本動作。
6. Single Number 的 XOR 解法依賴其他數字恰好出現兩次。
7. `value & (value - 1)` 會清除最低位的 1。
8. Power of Two 的 Binary 只有一個 1，但必須另外排除 0。
9. Bitmask 可以表示集合，每個 Bit 代表一個元素是否被選。
10. `n` 個元素共有 `2^n` 個 Masks，Bit 運算本身快速，但狀態數可能非常大。
11. Submask 與 Bitmask DP 是後續延伸，不需要第一次閱讀就全部掌握。
12. C++ 中應確認 unsigned 型別、Bit 寬度與 Shift 上限。
