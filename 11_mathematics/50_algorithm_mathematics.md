## 第 50 章　演算法常用數學

### 適用範圍

本章不是公式清單，而是一條由淺入深的解題路線。每個主題都先回答「它在解什麼問題」，再建立直覺、推導性質、寫出 C++，最後補充邊界條件與常見錯誤。

本章依序處理：

1. 整除、因數與倍數。
2. 最大公因數 GCD 與 Euclidean Algorithm。
3. 從整數 GCD 延伸到字串共同週期。
4. 最小公倍數 LCM 與同步問題。
5. 質數判斷、Sieve 與質因數分解。
6. 模運算、Fast Power 與 Modular Inverse。
7. 排列、組合與 Modulo 下的組合數。

數學工具的目的不是讓程式看起來更短，而是把候選空間縮小，或把原問題轉成已知模型。例如，字串共同週期看似是 String 題，但共同 Pattern 的長度會連結到整數的最大公因數。

### 適用讀者

- 會寫基本 C++，但看到數學題時不知道從哪裡開始的讀者。
- 知道 GCD、LCM、質數與組合公式，但不清楚何時使用的讀者。
- 容易在 Overflow、負數取餘數或 Modulo 除法上出錯的讀者。
- 想理解公式成立原因，而不是只記住寫法的讀者。
- 想把直覺解法逐步推導成較有效率解法的讀者。

### 閱讀方式

#### 第一輪必讀

先讀 50.1 至 50.7，掌握：

- 整除、因數與倍數。
- GCD 的意義。
- Euclidean Algorithm。
- 字串共同週期案例。
- LCM 與同步問題。

#### 第二輪再讀

50.8 至 50.10 處理質數、Sieve 與質因數分解。

#### 進階內容

50.11 至 50.15 涉及模運算、Fast Power、Modular Inverse 與組合數。第一次閱讀若尚未遇到相關題目，可以先知道各工具解決什麼問題，再於實際需要時回來。

### 快速導覽

- [50.1 遇到數學題先問什麼](#501-遇到數學題先問什麼)
- [50.2 整除、因數與倍數](#502-整除因數與倍數)
- [50.3 GCD：最大共同單位](#503-gcd最大共同單位)
- [50.4 Euclidean Algorithm](#504-euclidean-algorithm)
- [50.5 完整案例：字串的最大公因數](#505-完整案例字串的最大公因數)
- [50.6 LCM：最早再次同步](#506-lcm最早再次同步)
- [50.7 GCD 與 LCM 題型整理](#507-gcd-與-lcm-題型整理)
- [50.8 質數判斷](#508-質數判斷)
- [50.9 Sieve of Eratosthenes](#509-sieve-of-eratosthenes)
- [50.10 Prime Factorization](#5010-prime-factorization)
- [50.11 模運算基礎](#5011-模運算基礎)
- [50.12 Fast Power](#5012-fast-power)
- [50.13 Modular Inverse](#5013-modular-inverse)
- [50.14 排列與組合](#5014-排列與組合)
- [50.15 Modulo 下的組合數](#5015-modulo-下的組合數)
- [50.16 固定分析流程](#5016-固定分析流程)
- [50.17 常見錯誤與判讀](#5017-常見錯誤與判讀)
- [50.18 本章檢查表](#5018-本章檢查表)
- [50.19 本章重點](#5019-本章重點)


### 50.1 遇到數學題先問什麼

看到數字、公式或取餘數時，不要立即搜尋記憶中的公式。先把題目翻成下列問題。

#### 問題一：題目在問最大共同單位，還是最早同步時間

- 最大可切成多長的相同單位：常連結到 GCD。
- 兩個週期最早何時再相遇：常連結到 LCM。

這兩種題目都會出現「共同」，但方向不同：

```text
GCD：向下找可以共同分割兩者的最大單位
LCM：向上找同時包含兩者的最小總量
```

#### 問題二：題目在問存在性，還是要列出全部結果

- 只判斷一個數是否為質數：Trial Division 可能足夠。
- 多次查詢 `1..N` 是否為質數：考慮 Sieve。
- 需要每個質因數與次方：使用 Prime Factorization。

#### 問題三：答案是否可能非常大

先估算：

- 單一輸入是否放得進 `int`？
- 加法是否可能超過型別範圍？
- 乘法是否在取 Modulo 前就 Overflow？
- 階乘或組合數是否快速成長？

#### 問題四：題目是否要求取餘數

若要求答案 `% MOD`，還要確認：

- `MOD` 是否大於 0？
- `MOD` 是否為質數？
- 是否包含除法？
- 分母是否在該 Modulo 下有乘法反元素？

#### 問題五：順序是否重要

- 選出哪些元素，順序不重要：Combination。
- 選出元素後，排列順序也算不同結果：Permutation。

#### 本章的固定策略

```text
先辨認模型
    ↓
寫出最直接的正確方法
    ↓
找出重複計算或過大的候選空間
    ↓
使用數學性質縮小問題
    ↓
最後檢查型別、Overflow 與前置條件
```


### 50.2 整除、因數與倍數

#### 整除的定義

若整數 `a` 可以寫成：

```text
a = b × k
```

其中 `k` 是整數，則稱 `b` 整除 `a`。

在 C++ 中通常檢查：

```cpp
b != 0 && a % b == 0
```

必須先確認 `b != 0`，因為不能對 0 取餘數。

#### 因數與倍數

若 `b` 整除 `a`：

- `b` 是 `a` 的因數。
- `a` 是 `b` 的倍數。

例如：

```text
12 = 3 × 4
```

因此：

- 3 與 4 都是 12 的因數。
- 12 是 3 與 4 的倍數。

#### 為什麼長度問題常出現整除

假設一段長度為 `p` 的 Pattern 重複若干次後，剛好形成長度為 `n` 的資料：

```text
n = p × 重複次數
```

因此 `p` 必須整除 `n`。

這個觀念不只用於數字，也會出現在：

- 重複字串。
- 週期陣列。
- 固定大小分組。
- 矩形切割。
- 循環狀態。

#### 必要條件不一定是充分條件

若 Pattern 長度能整除字串長度，只代表長度可能成立，不代表字元內容真的重複。

```text
字串：ABAC
長度：4
候選長度：2
```

2 能整除 4，但 `"AB"` 重複兩次是 `"ABAB"`，不是 `"ABAC"`。

因此：

> 整除常用來排除不可能的候選，但內容是否符合仍需另外驗證。


### 50.3 GCD：最大共同單位

GCD 是 Greatest Common Divisor，中文為最大公因數。

```text
gcd(a, b) = 同時整除 a 與 b 的最大正整數
```

例如：

```text
12 的正因數：1, 2, 3, 4, 6, 12
18 的正因數：1, 2, 3, 6, 9, 18
共同因數：    1, 2, 3, 6
最大共同因數：6
```

所以：

```text
gcd(12, 18) = 6
```

#### 先從直覺解法開始

可以從 `min(a, b)` 往下嘗試，找到第一個同時整除兩者的數：

```cpp
#include <algorithm>
#include <cstdlib>

long long gcdBySearch(long long a, long long b)
{
    a = std::llabs(a);
    b = std::llabs(b);

    for (long long divisor = std::min(a, b);
         divisor >= 1;
         --divisor)
    {
        if (a % divisor == 0 && b % divisor == 0)
        {
            return divisor;
        }
    }

    return 0;
}
```

這個方法有助於確認 GCD 的定義，但當數字很大時，逐一嘗試會太慢。

#### GCD 常在哪裡出現

- 將分數約分。
- 判斷兩個數是否互質。
- 尋找最大相同分組大小。
- 尋找最大共同步長。
- 判斷共同週期的基本長度。
- 計算 LCM。

#### 互質

若：

```text
gcd(a, b) = 1
```

則 `a` 與 `b` 互質。

互質不代表兩者都是質數。例如 8 與 15 都不是同一組質數，但：

```text
gcd(8, 15) = 1
```

#### C++ 標準函式

```cpp
#include <numeric>

const auto value = std::gcd(a, b);
```

`std::gcd` 位於 `<numeric>`。對整數輸入，它回傳非負結果。


### 50.4 Euclidean Algorithm

#### 為什麼需要另一種方法

逐一列舉共同因數，沒有使用餘數中已經存在的資訊。Euclidean Algorithm 利用：

```text
gcd(a, b) = gcd(b, a mod b)
```

反覆把問題縮小，直到餘數為 0。

#### 先看一個例子

計算：

```text
gcd(48, 18)
```

依序得到：

```text
48 = 18 × 2 + 12
18 = 12 × 1 + 6
12 =  6 × 2 + 0
```

最後一個非零餘數是 6，所以：

```text
gcd(48, 18) = 6
```

#### 核心性質為什麼成立

令：

```text
a = bq + r
```

其中：

```text
r = a mod b
```

若某個數 `d` 同時整除 `a` 與 `b`，它也會整除：

```text
a - bq = r
```

反過來，若 `d` 同時整除 `b` 與 `r`，它也會整除：

```text
bq + r = a
```

因此 `(a, b)` 與 `(b, r)` 具有相同共同因數，最大公因數也相同。

#### 迭代版本

```cpp
#include <cstdlib>

long long gcdIterative(long long a, long long b)
{
    a = std::llabs(a);
    b = std::llabs(b);

    while (b != 0)
    {
        const long long remainder = a % b;
        a = b;
        b = remainder;
    }

    return a;
}
```

#### 遞迴版本

```cpp
#include <cstdlib>

long long gcdRecursive(long long a, long long b)
{
    if (b == 0)
    {
        return std::llabs(a);
    }

    return gcdRecursive(b, a % b);
}
```

#### 邊界情況

```text
gcd(a, 0) = |a|
gcd(0, b) = |b|
gcd(0, 0) = 0    // C++ std::gcd 的定義
```

題目若要求「最大正共同因數」，通常不會同時給兩個 0，或需要另外定義語意。

#### 複雜度

Euclidean Algorithm 的時間複雜度通常寫成：

```text
O(log min(|a|, |b|))
```

它比從 `min(a, b)` 向下列舉快得多。


### 50.5 完整案例：字串的最大公因數

#### 這題為什麼放在數學章

題目表面上處理字串，實際上結合兩個模型：

1. 字串是否由同一個 Pattern 重複形成。
2. 最大共同 Pattern 的長度是多少。

第一部分是字串週期，第二部分會轉成整數長度的 GCD。

#### 字串整除的定義

若字串 `pattern` 重複一或多次後，剛好得到 `text`，就稱 `pattern` 整除 `text`。

```text
"ABC" × 2 = "ABCABC"
```

因此 `"ABC"` 整除 `"ABCABC"`。

但普通共同 Substring 不一定能整除整個字串。例如 `"BC"` 同時出現在 `"ABC"` 與 `"ABCABC"` 中，卻無法重複形成這兩個完整字串。

#### 第一步：先寫出 Pattern 驗證

```cpp
#include <string_view>

bool isRepeatedBy(
    std::string_view text,
    std::string_view pattern)
{
    if (pattern.empty() ||
        text.size() % pattern.size() != 0)
    {
        return false;
    }

    for (std::size_t i = 0; i < text.size(); ++i)
    {
        if (text[i] != pattern[i % pattern.size()])
        {
            return false;
        }
    }

    return true;
}
```

`i % pattern.size()` 會讓 Pattern Index 循環：

```text
Pattern 長度為 3

i：       0 1 2 3 4 5 6 7 8
i % 3：   0 1 2 0 1 2 0 1 2
```

#### 第二步：列舉候選 Prefix

共同 Pattern 一定是兩個字串的 Prefix，因為重複必須從開頭開始。

```cpp
#include <algorithm>
#include <string>

std::string gcdOfStringsBySearch(
    const std::string& first,
    const std::string& second)
{
    const std::size_t maximumLength =
        std::min(first.size(), second.size());

    for (std::size_t length = maximumLength;
         length > 0;
         --length)
    {
        if (first.size() % length != 0 ||
            second.size() % length != 0)
        {
            continue;
        }

        const std::string candidate =
            first.substr(0, length);

        if (isRepeatedBy(first, candidate) &&
            isRepeatedBy(second, candidate))
        {
            return candidate;
        }
    }

    return "";
}
```

這個版本已經正確。它先用長度整除排除不可能候選，再檢查內容。

#### 第三步：候選長度連結到 GCD

若 Pattern 長度為 `p`，而它能形成兩個字串，則：

```text
first.size()  = p × 某個整數
second.size() = p × 某個整數
```

因此 `p` 必須同時整除兩個長度。最大可能的 `p` 就是：

```text
gcd(first.size(), second.size())
```

但長度符合仍不足以保證內容具有共同週期。

#### 第四步：如何判斷共同週期是否存在

若兩個字串都由同一個 Pattern 重複形成，交換串接順序後應得到相同結果：

```text
first + second == second + first
```

成功例子：

```text
first  = "ABABAB"
second = "ABAB"

first + second  = "ABABABABAB"
second + first  = "ABABABABAB"
```

失敗例子：

```text
first  = "LEET"
second = "CODE"

first + second  = "LEETCODE"
second + first  = "CODELEET"
```

兩者不同，代表不存在共同重複 Pattern。

#### 最終解法

```cpp
#include <numeric>
#include <string>

std::string gcdOfStrings(
    const std::string& first,
    const std::string& second)
{
    if (first + second != second + first)
    {
        return "";
    }

    const std::size_t gcdLength =
        std::gcd(first.size(), second.size());

    return first.substr(0, gcdLength);
}
```

#### 為什麼不能只取 GCD 長度

考慮：

```text
first  = "AB"
second = "AC"
```

兩者長度 GCD 是 2，但 `"AB"` 無法形成 `"AC"`。因此必須先驗證共同週期存在。

#### 複雜度

令兩個字串長度為 `n` 與 `m`：

- 串接與比較：O(n + m)。
- GCD：O(log min(n, m))。
- 建立結果字串：O(g)，其中 `g = gcd(n, m)`。

整體時間為 O(n + m)，額外空間取決於串接產生的暫時字串。

#### 本案例真正要帶走的觀念

```text
先寫出可驗證 Pattern 的直覺解
    ↓
發現候選長度必須整除兩個長度
    ↓
最大候選長度是長度 GCD
    ↓
再用字串性質驗證共同週期存在
```

這不是只靠記住一行公式，而是由字串結構逐步轉成整數問題。


### 50.6 LCM：最早再次同步

LCM 是 Least Common Multiple，中文為最小公倍數。

```text
lcm(a, b) = 同時為 a 與 b 倍數的最小非負整數
```

對正整數而言，通常理解為最小正共同倍數。

例如：

```text
12 的倍數：12, 24, 36, 48, ...
18 的倍數：18, 36, 54, ...
```

第一個共同倍數是 36：

```text
lcm(12, 18) = 36
```

#### 常見情境

- 事件 A 每 4 秒發生一次，事件 B 每 6 秒發生一次，何時再次同時發生？
- 兩個循環長度何時回到共同起點？
- 多個分母的共同尺度。
- 週期系統的同步時間。

#### 公式

```text
lcm(a, b) = |a / gcd(a, b) × b|
```

先除再乘可以降低中間乘法 Overflow 的風險。

不要優先寫成：

```text
|a × b| / gcd(a, b)
```

因為 `a × b` 可能在除法前就超過型別範圍。

#### C++ 寫法

```cpp
#include <cstdlib>
#include <numeric>

long long safeLcm(long long a, long long b)
{
    if (a == 0 || b == 0)
    {
        return 0;
    }

    return std::llabs(a / std::gcd(a, b) * b);
}
```

即使先除再乘，若真正答案超過 `long long`，仍然會 Overflow。先除再乘只能降低風險，不能保證任何輸入都安全。

#### 多個數的 LCM

可以依序合併：

```text
lcm(a, b, c) = lcm(lcm(a, b), c)
```

每次合併都要檢查中間結果是否超過型別範圍。


### 50.7 GCD 與 LCM 題型整理

#### 看到「最大可切單位」

常見方向：GCD。

例如：

- 將兩根不同長度的木條切成相同且最長的小段。
- 將網格步長化成最小整數比例。
- 尋找兩個重複結構的最大共同基本長度。

#### 看到「最早再次同步」

常見方向：LCM。

例如：

- 不同週期的燈何時再次同時亮起。
- 兩個循環何時同時回到起點。
- 不同批次間隔何時重合。

#### 看到「共同週期」時先不要直接選

「共同週期」可能問：

- 最大共同基本單位：GCD。
- 最早共同到達時間：LCM。

必須先判斷題目是在向下分割，還是向上同步。

#### 整數性質只是必要條件嗎

像字串、陣列或幾何題常同時有：

1. 數值條件。
2. 結構條件。

長度可整除不代表內容一定週期相同。GCD 給出最大可能長度，仍需確認結構符合。


### 50.8 質數判斷

#### 定義

質數是大於 1，且只有 1 與自己兩個正因數的整數。

```text
2, 3, 5, 7, 11, 13, ...
```

0、1 與負數都不是質數。

#### 最直接的判斷

可以測試 `2..n-1` 是否有因數，但時間為 O(n)。

#### 為什麼只需檢查到平方根

若 `n` 是合數，可以寫成：

```text
n = a × b
```

若 `a` 與 `b` 都大於 `sqrt(n)`，乘積就會大於 `n`，產生矛盾。因此至少有一個因數不超過 `sqrt(n)`。

#### 避免 `d * d` Overflow

不要只寫：

```cpp
for (long long d = 2; d * d <= n; ++d)
```

當 `d` 很大時，`d * d` 可能先 Overflow。可改成：

```cpp
d <= n / d
```

#### C++ 解法

```cpp
bool isPrime(long long n)
{
    if (n < 2)
    {
        return false;
    }

    if (n == 2)
    {
        return true;
    }

    if (n % 2 == 0)
    {
        return false;
    }

    for (long long divisor = 3;
         divisor <= n / divisor;
         divisor += 2)
    {
        if (n % divisor == 0)
        {
            return false;
        }
    }

    return true;
}
```

#### 複雜度

```text
O(sqrt(n))
```

若只判斷少量數字，通常足夠。若要回答大量 `1..N` 內的查詢，考慮 Sieve。


### 50.9 Sieve of Eratosthenes

#### 它解決什麼問題

若需要一次找出 `1..N` 中所有質數，對每個數分別做 O(sqrt(n)) 試除會重複很多工作。

Sieve 的想法是：

1. 先假設所有數都是質數。
2. 從 2 開始。
3. 若 `p` 尚未被標記為合數，則 `p` 是質數。
4. 將 `p` 的倍數標記為合數。

#### 為什麼從 `p * p` 開始

`2p、3p、...、(p-1)p` 都含有小於 `p` 的因數，已經在更早階段處理過。

#### C++ 解法

```cpp
#include <vector>

std::vector<bool> buildPrimeTable(int n)
{
    std::vector<bool> isPrime(n + 1, true);

    if (n >= 0)
    {
        isPrime[0] = false;
    }

    if (n >= 1)
    {
        isPrime[1] = false;
    }

    for (int p = 2; p <= n / p; ++p)
    {
        if (!isPrime[p])
        {
            continue;
        }

        for (int multiple = p * p;
             multiple <= n;
             multiple += p)
        {
            isPrime[multiple] = false;
        }
    }

    return isPrime;
}
```

#### 複雜度

- 時間：O(n log log n)。
- 空間：O(n)。

#### 何時選 Trial Division，何時選 Sieve

- 少量獨立數字：Trial Division。
- 大量查詢且上限 `N` 可接受：Sieve。
- `N` 極大但查詢範圍有限：可能需要 Segmented Sieve 或其他方法。


### 50.10 Prime Factorization

Prime Factorization 是將整數表示成質數乘積。

例如：

```text
60 = 2² × 3 × 5
```

#### 試除法分解

```cpp
#include <utility>
#include <vector>

std::vector<std::pair<long long, int>> factorize(
    long long n)
{
    std::vector<std::pair<long long, int>> factors;

    for (long long prime = 2;
         prime <= n / prime;
         ++prime)
    {
        if (n % prime != 0)
        {
            continue;
        }

        int exponent = 0;

        while (n % prime == 0)
        {
            n /= prime;
            ++exponent;
        }

        factors.push_back({prime, exponent});
    }

    if (n > 1)
    {
        factors.push_back({n, 1});
    }

    return factors;
}
```

此函式假設 `n >= 1`。若題目允許 0 或負數，應先另外定義語意。

#### 為什麼最後的 `n > 1` 是質數

迴圈已移除所有不大於當前平方根的質因數。若最後仍剩下大於 1 的 `n`，它不可能再拆成兩個都大於 1 的因數，否則其中至少有一個會不超過平方根，應已被找到。

#### 因數個數公式

若：

```text
n = p1^a1 × p2^a2 × ... × pk^ak
```

則正因數個數是：

```text
(a1 + 1)(a2 + 1)...(ak + 1)
```

原因是每個質因數 `pi` 可以選擇使用 `0..ai` 次，各選擇彼此獨立。

例如：

```text
60 = 2² × 3¹ × 5¹
```

正因數個數：

```text
(2 + 1)(1 + 1)(1 + 1) = 12
```


### 50.11 模運算基礎

#### 取餘數

```text
a mod m
```

C++ 寫成：

```cpp
const long long remainder = a % m;
```

前提是 `m != 0`。

#### 基本性質

```text
(a + b) mod m = ((a mod m) + (b mod m)) mod m
(a - b) mod m = ((a mod m) - (b mod m)) mod m
(a × b) mod m = ((a mod m) × (b mod m)) mod m
```

這些性質允許在計算過程中持續縮小數值。

#### 負數正規化

C++ 的負數餘數可能為負。若 `mod > 0`，而需求要求結果位於 `[0, mod)`：

```cpp
long long normalizeModulo(
    long long value,
    long long mod)
{
    return ((value % mod) + mod) % mod;
}
```

#### 乘法仍可能先 Overflow

```cpp
(a * b) % mod
```

取 Modulo 發生在乘法之後。如果 `a * b` 已超過型別範圍，後面的 `% mod` 無法修復結果。

對 `int` 輸入，常先提升成 `long long`：

```cpp
const long long result = (1LL * a * b) % mod;
```

若輸入接近 64-bit 上限，仍可能需要 `__int128` 或其他安全乘法方法，需依平台與題目限制決定。

#### Modulo 下的除法不同

一般不能把：

```text
a / b mod m
```

直接寫成整數除法後再 `% m`。Modulo 下的除法需要乘法反元素，50.13 會說明。


### 50.12 Fast Power

#### 它解決什麼問題

直接將 `base` 乘 `exponent` 次，需要 O(exponent) 時間。

Binary Exponentiation 利用：

```text
base^(2k)     = (base^k)²
base^(2k + 1) = base × (base^k)²
```

每次將指數減半，時間降為 O(log exponent)。

#### C++ 解法

```cpp
long long modPow(
    long long base,
    long long exponent,
    long long mod)
{
    base = normalizeModulo(base, mod);
    long long result = 1 % mod;

    while (exponent > 0)
    {
        if ((exponent & 1LL) != 0)
        {
            result = result * base % mod;
        }

        base = base * base % mod;
        exponent >>= 1;
    }

    return result;
}
```

#### 前置條件

- `mod > 0`。
- `exponent >= 0`。
- 中間乘法不超過 `long long`，或已採用更安全乘法。
- `0^0` 的語意由題目定義。

#### 如何理解 Bit

若指數是 13：

```text
13 = 8 + 4 + 1
```

因此：

```text
base^13 = base^8 × base^4 × base^1
```

演算法依序檢查指數的 Binary Bits，遇到 1 就將對應平方值乘進答案。


### 50.13 Modular Inverse

#### 為什麼不能直接除

Modulo 等價類中，普通整數除法不一定保留同樣意義。因此：

```text
a / b mod m
```

通常要改寫成：

```text
a × inv(b) mod m
```

其中 `inv(b)` 滿足：

```text
b × inv(b) ≡ 1 (mod m)
```

#### 反元素何時存在

`b` 在 Modulo `m` 下有乘法反元素，當且僅當：

```text
gcd(b, m) = 1
```

這表示 `b` 與 `m` 必須互質。

#### 質數 Modulo 下使用 Fermat's Little Theorem

若 `m` 是質數，且 `b` 不是 `m` 的倍數：

```text
b^(m - 1) ≡ 1 (mod m)
```

所以：

```text
b^(m - 2) ≡ b^(-1) (mod m)
```

C++：

```cpp
long long modInversePrime(
    long long value,
    long long mod)
{
    return modPow(value, mod - 2, mod);
}
```

#### 前置條件

- `mod` 是質數。
- `value % mod != 0`。
- `modPow` 的乘法安全條件成立。

若 `mod` 不是質數，可以考慮 Extended Euclidean Algorithm，但仍需先確認 `gcd(value, mod) == 1`。

#### 常見錯誤

- 在 Modulo 下直接使用 `/`。
- `mod` 不是質數，卻直接使用 `value^(mod-2)`。
- 分母與 `mod` 不互質。
- 分母在 Modulo 下等於 0。


### 50.14 排列與組合

#### Factorial

```text
n! = 1 × 2 × ... × n
0! = 1
```

階乘成長非常快，使用前要先估算範圍。

#### Permutation：順序重要

從 `n` 個不同元素中選 `k` 個並排列：

```text
P(n, k) = n! / (n-k)!
```

例如從 A、B、C 選兩個：

```text
AB, AC, BA, BC, CA, CB
```

共有 6 種。

#### Combination：順序不重要

從 `n` 個不同元素中選 `k` 個：

```text
C(n, k) = n! / (k!(n-k)!)
```

同一例子中：

```text
AB, AC, BC
```

共有 3 種，`AB` 與 `BA` 視為同一組選擇。

#### 先問一句話

> 選到相同元素，但順序不同，是否算不同答案？

- 算不同：Permutation。
- 不算不同：Combination。

#### Combination 的基本性質

```text
C(n, 0) = 1
C(n, n) = 1
C(n, k) = C(n, n-k)
C(n, k) = C(n-1, k-1) + C(n-1, k)
```

最後一條是 Pascal's Identity。它將所有選法分成：

- 有選某個指定元素。
- 沒有選該元素。

#### 小範圍計算

```cpp
#include <algorithm>

long long combinationSmall(int n, int k)
{
    if (k < 0 || k > n)
    {
        return 0;
    }

    k = std::min(k, n - k);
    long long result = 1;

    for (int i = 1; i <= k; ++i)
    {
        result = result * (n - k + i) / i;
    }

    return result;
}
```

這個版本只適合答案與中間乘法都能放進 `long long` 的範圍。即使最後答案沒有超過，中間的 `result * value` 仍可能先 Overflow。


### 50.15 Modulo 下的組合數

若組合數極大，而題目要求：

```text
C(n, k) mod MOD
```

在 `MOD` 為質數且範圍合適時，可以預處理：

- `factorial[i] = i! mod MOD`
- `inverseFactorial[i] = (i!)^(-1) mod MOD`

接著：

```text
C(n, k) = factorial[n]
          × inverseFactorial[k]
          × inverseFactorial[n-k]
          mod MOD
```

#### C++ 寫法

```cpp
#include <vector>

struct CombinationMod
{
    long long mod;
    std::vector<long long> factorial;
    std::vector<long long> inverseFactorial;

    CombinationMod(int maxN, long long modValue)
        : mod(modValue),
          factorial(maxN + 1, 1),
          inverseFactorial(maxN + 1, 1)
    {
        for (int i = 1; i <= maxN; ++i)
        {
            factorial[i] =
                factorial[i - 1] * i % mod;
        }

        inverseFactorial[maxN] =
            modPow(factorial[maxN], mod - 2, mod);

        for (int i = maxN; i >= 1; --i)
        {
            inverseFactorial[i - 1] =
                inverseFactorial[i] * i % mod;
        }
    }

    long long choose(int n, int k) const
    {
        if (k < 0 || k > n)
        {
            return 0;
        }

        return factorial[n]
             * inverseFactorial[k] % mod
             * inverseFactorial[n - k] % mod;
    }
};
```

#### 前置條件

- `mod` 是質數。
- `0 <= n <= maxN`。
- 預處理範圍中需要反轉的 Factorial 在 Modulo 下不等於 0。
- 中間乘法不超過使用型別。

特別是當 `maxN >= mod` 時，`factorial[maxN] % mod` 可能等於 0，不能直接使用上述反 Factorial 方法。此時需要依題目限制選擇其他方法。

#### 若 MOD 不是質數

不能直接套用 Fermat Inverse。可能方法包括：

- Pascal DP。
- Extended GCD。
- 分解質因數後處理。
- Chinese Remainder Theorem。

應依 `n`、查詢次數與 `MOD` 性質選擇，不存在一個適用所有情況的固定寫法。


### 50.16 固定分析流程

遇到數學題時，可以依序完成以下檢查。

#### 第一步：翻譯題目關係

把敘述改寫成：

- `a = b × k`
- `a % b == 0`
- 最大共同分割單位。
- 最早共同同步時間。
- 從 `n` 個物件選 `k` 個。

#### 第二步：先寫直接方法

例如：

- GCD：先列舉共同因數。
- 質數：先列舉可能因數。
- 字串共同 Pattern：先列舉 Prefix。
- 組合計數：先確認小範圍 DP 或枚舉。

直接方法有助於確認你理解了題目，也能作為小資料測試基準。

#### 第三步：尋找可縮小候選的性質

- 因數成對出現，所以只需檢查到平方根。
- GCD 可用餘數遞迴縮小。
- Pattern 長度必須整除完整長度。
- 大指數可用 Binary 分解。
- Combination 可利用對稱性 `k = min(k, n-k)`。

#### 第四步：確認前置條件

- 除數不能為 0。
- Modulo 必須大於 0。
- Fermat Inverse 要求質數 Modulo。
- Modular Inverse 要求互質。
- 固定公式是否適用 0 或負數？

#### 第五步：估算中間值

不要只看最終答案。檢查：

- `a * b`。
- `p * p`。
- Factorial。
- 組合公式中的乘法。
- LCM 中先乘後除。

#### 第六步：設計邊界測試

至少包含：

- 0、1、2。
- 兩數相等。
- 其中一個整除另一個。
- 互質數。
- 負數是否允許。
- 接近型別上限。
- `MOD = 1` 或分母在 Modulo 下為 0。


### 50.17 常見錯誤與判讀

#### GCD 答案不符預期

先確認題目要的是：

- 最大共同分割單位。
- 最早共同同步時間。

後者通常是 LCM，不是 GCD。

#### 字串長度 GCD 正確，但答案字串錯誤

可能只檢查了長度，沒有檢查共同週期是否存在。

加入結構驗證：

```cpp
first + second == second + first
```

#### LCM 結果變負或異常

可能是 `a * b` 先 Overflow。改成先除 GCD 再乘，並確認真正答案仍在型別範圍內。

#### 1 被判成質數

質數定義要求大於 1。先處理：

```cpp
if (n < 2)
{
    return false;
}
```

#### 質數判斷在大數時出錯

可能是 `d * d` Overflow。改用：

```cpp
d <= n / d
```

#### Sieve 在很小的 N 越界

建立 `isPrime[0]` 與 `isPrime[1]` 前，要確認容器確實包含這些位置。

#### Modulo 結果為負

若 `mod > 0` 且需要 `[0, mod)`：

```cpp
((value % mod) + mod) % mod
```

#### 取 Modulo 後仍然 Overflow

問題可能發生在 `% mod` 之前的乘法。先檢查中間型別，而不是只看最終餘數。

#### Modulo 除法錯誤

Modulo 下不能直接使用普通除法。先確認反元素存在，並確認使用的方法符合 `mod` 性質。

#### 組合數結果錯誤

先檢查：

- 順序是否重要。
- 是否有重複元素。
- 中間乘法是否 Overflow。
- `MOD` 是否為質數。
- `maxN` 是否小於 `MOD`，或是否需要其他組合數方法。


### 50.18 本章檢查表

#### 整除、GCD 與 LCM

- 我能用 `a = b × k` 解釋整除。
- 我知道取餘數前除數不能是 0。
- 我能區分因數與倍數。
- 我知道 GCD 尋找最大共同單位。
- 我知道 LCM 尋找最早共同同步量。
- 我能說明 Euclidean Algorithm 為何保留共同因數。
- 我知道 LCM 應先除 GCD 再乘。
- 我知道先除再乘仍不保證結果一定不 Overflow。

#### 字串共同週期

- 我知道共同 Pattern 必須是兩個字串的 Prefix。
- 我知道 Pattern 長度必須整除完整字串長度。
- 我知道長度符合不代表內容符合。
- 我能先寫出列舉 Prefix 的直覺解。
- 我能解釋為何共同週期存在時，兩種串接順序相同。
- 我知道最大共同 Pattern 長度是兩個字串長度的 GCD。

#### 質數與因數分解

- 我知道 0、1 與負數不是質數。
- 我能解釋為何只需檢查到平方根。
- 我會用 `d <= n / d` 避免平方 Overflow。
- 我能判斷何時用 Trial Division，何時用 Sieve。
- 我知道質因數次方如何轉成因數個數。

#### Modulo

- 我知道 Modulo 的除數不能是 0。
- 我知道 C++ 負數餘數可能為負。
- 我知道乘法可能在 `% mod` 前 Overflow。
- 我能使用 Fast Power 將時間降為 O(log exponent)。
- 我知道 Modular Inverse 存在的條件是互質。
- 我知道 Fermat Inverse 還要求質數 Modulo。

#### 排列與組合

- 我能用「順序是否重要」區分 Permutation 與 Combination。
- 我知道 Factorial 與 Combination 成長很快。
- 我會檢查中間乘法，而不只看最終答案。
- 我知道反 Factorial 方法有 Modulo 與範圍前置條件。


### 50.19 本章重點

1. 先辨認題目模型，再選數學工具，不要只依關鍵字套公式。
2. 整除表示某數可以寫成另一數乘以整數；取餘數前除數不能為 0。
3. GCD 適合最大共同分割單位，LCM 適合最早共同同步量。
4. Euclidean Algorithm 利用 `gcd(a, b) = gcd(b, a % b)` 反覆縮小問題。
5. 字串共同週期是 String 與 Math 的交叉模型：先驗證共同 Pattern，再以長度 GCD 取得最大候選。
6. 數值條件有時只是必要條件；長度可整除不代表內容一定符合。
7. 質數判斷只需檢查到平方根，大量範圍查詢可使用 Sieve。
8. Prime Factorization 能將因數、倍數與計數問題轉成質因數次方。
9. Modulo 可以在加、減、乘過程中持續縮小數值，但不能修復已發生的 Overflow。
10. Modulo 下的除法需要乘法反元素，且反元素不一定存在。
11. Fast Power 將線性次乘法轉成 O(log exponent)。
12. Combination 與 Permutation 的核心差別是順序是否重要。
13. 所有公式都應搭配前置條件、型別範圍與邊界測試一起理解。
