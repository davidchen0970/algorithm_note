## 第 48 章　String Matching

### 適用範圍

本章假設讀者已經會：

- `for` 迴圈。
- 兩層迴圈。
- `if` 判斷。
- 使用 Index 讀取 `std::string`。
- 使用 `std::vector` 保存答案。

不假設讀者已經理解 Prefix Function、Border、Rolling Hash 或 Z Box。本章會先把最直接的 Naive Matching 寫清楚，再從「哪裡重複比較了」逐步走向 KMP。

第一次閱讀不需要同時學會 KMP、Rabin-Karp 與 Z Algorithm。建議分成三輪：

1. 第一輪：只學 Naive Matching，確認自己真的會找 Pattern。
2. 第二輪：學 Prefix、Suffix、Border 與 KMP。
3. 第三輪：再把 Rabin-Karp 與 Z Algorithm 當成其他思考方式。

### 這一章真正要解決什麼問題

給定兩個字串：

```text
text    = 很長的原始字串
pattern = 想尋找的較短字串
```

我們想知道：

- Pattern 是否出現在 Text 中？
- 第一個出現位置在哪裡？
- 所有出現位置在哪裡？
- Pattern 出現幾次？

例如：

```text
text    = "abcabc"
pattern = "abc"
```

Pattern 出現在 Index 0 與 Index 3。

### 閱讀方式

#### 第一輪必讀

- 48.1 問題與規格。
- 48.2 先用手找一次。
- 48.3 Naive Matching。
- 48.4 為什麼 Naive 可能慢。

讀完第一輪後，你應該能獨立寫出正確的 O(nm) 解法。這已經是完整解法，不是失敗版本。

#### 第二輪必讀

- 48.5 Prefix、Suffix 與 Border。
- 48.6 Prefix Function。
- 48.7 KMP 搜尋。

讀完第二輪後，再要求自己理解 KMP 如何避免重複比較。

#### 第三輪延伸

- 48.8 Rabin-Karp。
- 48.9 Z Algorithm。

這兩種方法不是理解 KMP 的前置條件。若第一次看不懂，可以先跳過。

### 快速導覽

- [48.1 先把問題說清楚](#481-先把問題說清楚)
- [48.2 先用手找一次](#482-先用手找一次)
- [48.3 Naive Matching](#483-naive-matching)
- [48.4 Naive Matching 為什麼可能慢](#484-naive-matching-為什麼可能慢)
- [48.5 Prefix、Suffix 與 Border](#485-prefixsuffix-與-border)
- [48.6 Prefix Function：先只理解它記錄什麼](#486-prefix-function先只理解它記錄什麼)
- [48.7 KMP：使用 Prefix Function 搜尋](#487-kmp使用-prefix-function-搜尋)
- [48.8 延伸：Rabin-Karp](#488-延伸rabin-karp)
- [48.9 延伸：Z Algorithm](#489-延伸z-algorithm)
- [48.10 方法選擇](#4810-方法選擇)
- [48.11 常見題型](#4811-常見題型)
- [48.12 固定分析流程](#4812-固定分析流程)
- [48.13 常見問題與判讀](#4813-常見問題與判讀)
- [48.14 本章檢查表](#4814-本章檢查表)
- [48.15 本章重點](#4815-本章重點)

---

### 48.1 先把問題說清楚

#### 基本範例

```text
text    = "ababcabcabababd"
pattern = "ababd"
```

Pattern 從 Text 的 Index 10 開始完整出現：

```text
Text Index：  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
Text：        a b a b c a b c a b  a  b  a  b  d
Pattern：                         a  b  a  b  d
```

因此第一個匹配位置是 10。

#### 題目可能要求不同輸出

同樣是 String Matching，輸出可能是：

- 是否存在：`true` 或 `false`。
- 第一個位置：例如 10，找不到時回傳 `-1`。
- 所有位置：例如 `{0, 3}`。
- 出現次數：例如 2。

演算法很接近，但找到一次後是否停止會不同。

#### 先定義空 Pattern

不同函式庫或題目可能採用不同定義。本章為了簡化搜尋案例，規定：

```text
pattern 為空時，回傳空的位置集合。
```

這不是唯一合理規格。實際題目若另有定義，應依題目調整。

#### 是否允許重疊

```text
text    = "aaaa"
pattern = "aa"
```

允許重疊時，位置是：

```text
0, 1, 2
```

因為：

```text
Index 0：aa..
Index 1：.aa.
Index 2：..aa
```

本章預設列出所有重疊匹配。

#### 本章的字元單位

所有範例都將 `std::string` 視為 Byte / `char` 序列。這適合 ASCII 題目。若需求是完整 Unicode Code Point 或 Grapheme Cluster Matching，需要先使用合適的文字處理方式。

---

### 48.2 先用手找一次

先不要想 KMP。

```text
text    = "abcabc"
pattern = "abc"
```

Pattern 長度是 3，Text 長度是 6。

可能的起點只有：

```text
0, 1, 2, 3
```

為什麼沒有 4 和 5？

因為從 4 或 5 開始，剩餘長度不足以放下長度 3 的 Pattern。

#### 起點 0

```text
text[0] == pattern[0]   a == a
text[1] == pattern[1]   b == b
text[2] == pattern[2]   c == c
```

全部相同，所以記錄 0。

#### 起點 1

```text
text[1] == pattern[0]   b != a
```

第一個字元就不同，不必繼續比較。

#### 起點 2

```text
text[2] == pattern[0]   c != a
```

失敗。

#### 起點 3

```text
text[3] == pattern[0]   a == a
text[4] == pattern[1]   b == b
text[5] == pattern[2]   c == c
```

全部相同，所以記錄 3。

答案是：

```text
0, 3
```

這就是 Naive Matching。它沒有特殊公式，只是列舉每個可能起點，再逐字比較。

---

### 48.3 Naive Matching

#### 先寫最基本版本

```cpp
#include <string>
#include <vector>

std::vector<int> naiveSearch(
    const std::string& text,
    const std::string& pattern)
{
    std::vector<int> positions;

    if (pattern.empty())
    {
        return positions;
    }

    const int n = static_cast<int>(text.size());
    const int m = static_cast<int>(pattern.size());

    for (int start = 0; start + m <= n; ++start)
    {
        bool matched = true;

        for (int j = 0; j < m; ++j)
        {
            if (text[start + j] != pattern[j])
            {
                matched = false;
                break;
            }
        }

        if (matched)
        {
            positions.push_back(start);
        }
    }

    return positions;
}
```

#### 外層迴圈在做什麼

```cpp
for (int start = 0; start + m <= n; ++start)
```

`start` 表示 Pattern 嘗試放在 Text 的哪個起點。

條件：

```cpp
start + m <= n
```

表示從 `start` 開始，後方至少還有 `m` 個字元可比較。

也可以寫成：

```cpp
start <= n - m
```

但只有在先確認 `m <= n` 且型別不會產生 Unsigned Underflow 時才適合。`start + m <= n` 在目前的 `int` 寫法中較直觀。

#### 內層迴圈在做什麼

```cpp
for (int j = 0; j < m; ++j)
```

`j` 表示目前比較 Pattern 的第幾個字元。

對應的 Text 位置是：

```cpp
start + j
```

所以比較：

```cpp
text[start + j] != pattern[j]
```

#### 為什麼不同時可以 `break`

只要有一個位置不同，整個 Pattern 就不可能在這個起點完整匹配。

因此不必比較剩下內容。

#### 若只需要第一個位置

```cpp
#include <string>

int findFirst(
    const std::string& text,
    const std::string& pattern)
{
    if (pattern.empty())
    {
        return 0;
    }

    const int n = static_cast<int>(text.size());
    const int m = static_cast<int>(pattern.size());

    for (int start = 0; start + m <= n; ++start)
    {
        int j = 0;

        while (j < m &&
               text[start + j] == pattern[j])
        {
            ++j;
        }

        if (j == m)
        {
            return start;
        }
    }

    return -1;
}
```

#### 複雜度

若 Text 長度為 `n`，Pattern 長度為 `m`：

- 可能起點約有 O(n) 個。
- 每個起點最多比較 O(m) 次。
- 最差時間是 O(nm)。
- 不含輸出時，額外空間是 O(1)。

#### Naive 並不是不好的解法

適合使用它的情況：

- 輸入很短。
- 題目限制允許 O(nm)。
- 想先確保邏輯正確。
- 想用它測試 KMP 或其他進階方法。

先寫出正確的 Naive 解法，再決定是否需要最佳化，是合理的解題流程。

---

### 48.4 Naive Matching 為什麼可能慢

考慮：

```text
text    = "aaaaaaaaab"
pattern = "aaaab"
```

在起點 0：

```text
a == a
a == a
a == a
a == a
a != b
```

比較到很後面才失敗。

移到起點 1 後，又會重新比較大量 `a`：

```text
a == a
a == a
a == a
a == a
a != b
```

問題不是比較本身，而是：

> 失敗後完全忘記前面已經知道的 Pattern 結構。

KMP 的目標就是保留其中一部分資訊。

#### 暫時不要急著看 KMP 程式

在進入 KMP 前，先建立三個詞：

- Prefix：從開頭開始。
- Suffix：到結尾結束。
- Border：同時是 Prefix 與 Suffix。

KMP 只是利用 Border 決定失敗後還能保留多少匹配。

---

### 48.5 Prefix、Suffix 與 Border

#### Prefix

Prefix 必須從字串開頭開始。

對：

```text
s = "abcab"
```

Prefix 有：

```text
"a"
"ab"
"abc"
"abca"
"abcab"
```

#### Suffix

Suffix 必須在字串結尾結束。

```text
"b"
"ab"
"cab"
"bcab"
"abcab"
```

#### Proper Prefix 與 Proper Suffix

Proper 表示不包含整個字串本身。

對 `"abab"`：

```text
Proper Prefix："a", "ab", "aba"
Proper Suffix："b", "ab", "bab"
```

#### Border

Border 是同時為 Proper Prefix 與 Proper Suffix 的內容。

對 `"abab"`：

```text
"ab"
```

同時在開頭與結尾出現，因此是 Border。

```text
字串：  a b a b
Prefix：a b
Suffix：    a b
```

#### 再看 `"aaaa"`

Proper Prefix：

```text
"a", "aa", "aaa"
```

Proper Suffix：

```text
"a", "aa", "aaa"
```

Border 有多個：

```text
"a", "aa", "aaa"
```

最長 Border 長度是 3。

#### Border 和 KMP 有什麼關係

假設目前已經匹配：

```text
"abab"
```

下一個字元卻失敗。

由於 `"abab"` 的結尾 `"ab"` 同時也是 Pattern 的開頭 `"ab"`，這兩個字元仍可能作為下一次匹配的開頭，不必全部丟掉。

因此匹配長度可以從 4 回到 2，而不是回到 0。

這就是 KMP 的核心直覺。

---

### 48.6 Prefix Function：先只理解它記錄什麼

#### `pi[i]` 的意思

`pi[i]` 表示：

```text
pattern[0..i] 的最長 Border 長度
```

例如：

```text
pattern = "ababd"
Index：     0 1 2 3 4
字元：      a b a b d
pi：        0 0 1 2 0
```

逐格理解：

- `i = 0`，字串是 `"a"`，沒有 Proper Border，所以是 0。
- `i = 1`，字串是 `"ab"`，沒有 Border，所以是 0。
- `i = 2`，字串是 `"aba"`，開頭與結尾都有 `"a"`，所以是 1。
- `i = 3`，字串是 `"abab"`，最長 Border 是 `"ab"`，所以是 2。
- `i = 4`，字串是 `"ababd"`，沒有 Border，所以是 0。

第一輪先確定看懂這張表，不需要立即自己寫出建立流程。

#### 先用慢方法建立 `pi`

下面的版本不是最有效率，但比較符合定義：對每個結尾位置，嘗試所有可能 Border 長度。

```cpp
#include <string>
#include <vector>

std::vector<int> buildPrefixFunctionSlow(
    const std::string& pattern)
{
    const int m = static_cast<int>(pattern.size());
    std::vector<int> pi(m, 0);

    for (int end = 0; end < m; ++end)
    {
        const int currentLength = end + 1;

        for (int length = currentLength - 1;
             length >= 1;
             --length)
        {
            bool same = true;

            for (int j = 0; j < length; ++j)
            {
                const int suffixStart =
                    currentLength - length;

                if (pattern[j] !=
                    pattern[suffixStart + j])
                {
                    same = false;
                    break;
                }
            }

            if (same)
            {
                pi[end] = length;
                break;
            }
        }
    }

    return pi;
}
```

這個版本的目的，是把「最長 Prefix 等於 Suffix」直接寫成程式。

#### 再看線性版本

理解 `pi` 的含義後，再看正式版本：

```cpp
#include <string>
#include <vector>

std::vector<int> buildPrefixFunction(
    const std::string& pattern)
{
    const int m = static_cast<int>(pattern.size());
    std::vector<int> pi(m, 0);

    for (int i = 1; i < m; ++i)
    {
        int length = pi[i - 1];

        while (length > 0 &&
               pattern[i] != pattern[length])
        {
            length = pi[length - 1];
        }

        if (pattern[i] == pattern[length])
        {
            ++length;
        }

        pi[i] = length;
    }

    return pi;
}
```

#### 每個變數代表什麼

- `i`：目前要加入 Pattern Prefix 的新位置。
- `length`：目前嘗試延伸的 Border 長度。
- `pattern[length]`：若目前 Border 可以延伸，下一個應匹配的位置。

#### 為什麼失敗時是 `pi[length - 1]`

如果長度為 `length` 的 Border 無法加上目前字元，就改試這個 Border 自己的最長 Border。

```cpp
length = pi[length - 1];
```

這不是隨意減一，而是跳到「下一個仍有可能成立的 Border 長度」。

#### 用 `"ababd"` 看幾個位置

在 `i = 2`：

```text
目前字元 pattern[2] = 'a'
length = pi[1] = 0
比較 pattern[2] 與 pattern[0]
'a' == 'a'
length 變成 1
pi[2] = 1
```

在 `i = 3`：

```text
目前字元 pattern[3] = 'b'
length = pi[2] = 1
比較 pattern[3] 與 pattern[1]
'b' == 'b'
length 變成 2
pi[3] = 2
```

在 `i = 4`：

```text
目前字元 pattern[4] = 'd'
length = pi[3] = 2
比較 pattern[4] 與 pattern[2]
'd' != 'a'
回退到 pi[1] = 0
再比較 pattern[4] 與 pattern[0]
'd' != 'a'
pi[4] = 0
```

#### 複雜度

線性版本是 O(m)。雖然裡面有 `while`，但 `length` 在回退時會快速減小，整體不會對每個 `i` 都重新掃描完整 Pattern。

第一次閱讀若還無法自行重寫這段很正常。先確保能拿一個短 Pattern 手算 `pi`，再回來練寫程式。

---

### 48.7 KMP：使用 Prefix Function 搜尋

#### KMP 需要記住的狀態

掃描 Text 時，只需要記錄：

```text
matched = 目前已經匹配 Pattern 前幾個字元
```

例如：

```text
matched = 3
```

表示 Pattern 的：

```text
pattern[0], pattern[1], pattern[2]
```

已經與目前 Text 結尾對上，下一個想比較 `pattern[3]`。

#### 先看主要流程

對每個 `text[i]`：

1. 如果與下一個 Pattern 字元不同，就依 `pi` 回退 `matched`。
2. 如果相同，就讓 `matched` 加一。
3. 如果 `matched == pattern.size()`，代表找到完整匹配。
4. 記錄答案後，依 `pi` 回退，以保留重疊匹配的可能。

#### 完整程式

```cpp
#include <string>
#include <vector>

std::vector<int> kmpSearch(
    const std::string& text,
    const std::string& pattern)
{
    std::vector<int> positions;

    if (pattern.empty())
    {
        return positions;
    }

    const std::vector<int> pi =
        buildPrefixFunction(pattern);

    const int n = static_cast<int>(text.size());
    const int m = static_cast<int>(pattern.size());
    int matched = 0;

    for (int i = 0; i < n; ++i)
    {
        while (matched > 0 &&
               text[i] != pattern[matched])
        {
            matched = pi[matched - 1];
        }

        if (text[i] == pattern[matched])
        {
            ++matched;
        }

        if (matched == m)
        {
            positions.push_back(i - m + 1);
            matched = pi[matched - 1];
        }
    }

    return positions;
}
```

#### 為什麼起點是 `i - m + 1`

找到時，`i` 是匹配最後一個字元的位置。

若 Pattern 長度是 `m`：

```text
起點 = 終點 - 長度 + 1
```

所以：

```cpp
i - m + 1
```

#### 找到後為什麼不把 `matched` 設成 0

```text
text    = "aaaa"
pattern = "aa"
```

找到 Index 0 的 `"aa"` 後，最後一個 `a` 仍可能是下一個匹配的第一個 `a`。

Pattern `"aa"` 的最長 Border 是 `"a"`，長度 1。因此：

```cpp
matched = pi[matched - 1];
```

會保留 1，接著能找到位置 1 與 2。

#### KMP 與 Naive 的差異

Naive 失敗時：

```text
移動起點，Pattern 從 0 重新比較
```

KMP 失敗時：

```text
Text 不回頭，Pattern 使用 Border 決定 matched 回退到哪裡
```

#### 複雜度

- 建立 Prefix Function：O(m)。
- 掃描 Text：O(n)。
- 總時間：O(n + m)。
- 額外空間：O(m)，不含輸出。

#### 如果現在還寫不出 KMP

先分成兩個小目標：

1. 給一個 Pattern，手算 `pi`。
2. 給定已經算好的 `pi`，追蹤 `matched` 如何變化。

不要要求自己第一次就同時寫出建表與搜尋。

---

### 48.8 延伸：Rabin-Karp

> 第一次閱讀可以跳過。本節需要 Modulo 與 Hash 的基本觀念。

#### 核心直覺

Naive Matching 逐字比較每個長度為 `m` 的 Window。Rabin-Karp 則先為 Pattern 與 Window 計算數值摘要，也就是 Hash。

```text
Hash 不同：一定不相同
Hash 相同：可能相同，仍可能發生 Collision
```

Hash Collision 表示不同字串算出相同 Hash。

因此若答案必須完全正確，Hash 相同後還要逐字驗證。

#### 先看簡化流程

```text
計算 Pattern Hash
計算第一個 Window Hash
對每個 Window：
    Hash 相同嗎？
        否：移到下一個 Window
        是：逐字驗證
    更新成下一個 Window Hash
```

#### Rolling Hash 的目的

從一個 Window 移到下一個時，不重新計算全部 `m` 個字元，而是：

1. 移除最左字元的影響。
2. 將其餘內容向前移位。
3. 加入新的右端字元。

這就是 Rolling 的意思。

#### 一個完整版本

```cpp
#include <string>
#include <vector>

std::vector<int> rabinKarpSearch(
    const std::string& text,
    const std::string& pattern)
{
    std::vector<int> positions;

    if (pattern.empty())
    {
        return positions;
    }

    const int n = static_cast<int>(text.size());
    const int m = static_cast<int>(pattern.size());

    if (m > n)
    {
        return positions;
    }

    const long long base = 256;
    const long long mod = 1'000'000'007;
    long long patternHash = 0;
    long long windowHash = 0;
    long long highestPower = 1;

    for (int i = 0; i < m; ++i)
    {
        patternHash =
            (patternHash * base +
             static_cast<unsigned char>(pattern[i])) % mod;

        windowHash =
            (windowHash * base +
             static_cast<unsigned char>(text[i])) % mod;

        if (i + 1 < m)
        {
            highestPower =
                highestPower * base % mod;
        }
    }

    for (int start = 0; start + m <= n; ++start)
    {
        if (patternHash == windowHash)
        {
            bool same = true;

            for (int j = 0; j < m; ++j)
            {
                if (text[start + j] != pattern[j])
                {
                    same = false;
                    break;
                }
            }

            if (same)
            {
                positions.push_back(start);
            }
        }

        if (start + m < n)
        {
            const long long removed =
                static_cast<unsigned char>(text[start])
                * highestPower % mod;

            windowHash =
                (windowHash - removed + mod) % mod;

            windowHash =
                (windowHash * base +
                 static_cast<unsigned char>(text[start + m]))
                % mod;
        }
    }

    return positions;
}
```

#### 目前只需要記住

- Hash 不同可以快速排除。
- Hash 相同不保證字串相同。
- 要求完全正確時，Hash 相同後逐字確認。
- 更新 Window Hash 時要避免負數。

不用在第一次閱讀時背下整段 Rolling Hash。

---

### 48.9 延伸：Z Algorithm

> 第一次閱讀可以跳過。本節是另一種利用 Prefix 資訊的方法。

#### `z[i]` 的意思

```text
z[i] = 從位置 i 開始的內容，與整個字串 Prefix
       最多連續相同幾個字元
```

例如：

```text
s = "ababa"
```

從 Index 2 開始：

```text
s[2..] = "aba"
Prefix = "aba"
```

前 3 個字元相同，所以：

```text
z[2] = 3
```

Z Array 為：

```text
[0, 0, 3, 0, 1]
```

#### 如何拿來搜尋 Pattern

將三段串起來：

```text
pattern + separator + text
```

例如：

```text
pattern = "aba"
text    = "ababa"
combined = "aba#ababa"
```

若 Text 對應位置的 Z 值至少等於 Pattern 長度，就代表 Pattern 在該位置完整匹配。

Separator 必須是不會出現在 Pattern 與 Text 中的字元。

#### 先看直接建立 Z Array 的慢版本

```cpp
#include <string>
#include <vector>

std::vector<int> buildZArraySlow(
    const std::string& text)
{
    const int n = static_cast<int>(text.size());
    std::vector<int> z(n, 0);

    for (int i = 1; i < n; ++i)
    {
        while (i + z[i] < n &&
               text[z[i]] == text[i + z[i]])
        {
            ++z[i];
        }
    }

    return z;
}
```

這個版本直接依定義比較，最差可能是 O(n²)，但容易理解 `z[i]` 在算什麼。

#### 線性版本

```cpp
#include <algorithm>
#include <string>
#include <vector>

std::vector<int> buildZArray(
    const std::string& text)
{
    const int n = static_cast<int>(text.size());
    std::vector<int> z(n, 0);
    int left = 0;
    int right = 0;

    for (int i = 1; i < n; ++i)
    {
        if (i < right)
        {
            z[i] = std::min(
                right - i,
                z[i - left]);
        }

        while (i + z[i] < n &&
               text[z[i]] == text[i + z[i]])
        {
            ++z[i];
        }

        if (i + z[i] > right)
        {
            left = i;
            right = i + z[i];
        }
    }

    return z;
}
```

`[left, right)` 表示目前已知和 Prefix 相同、而且延伸最右的區間。位置 `i` 落在區間內時，可以先重用已知結果，再向右繼續比較。

#### 搜尋 Pattern

```cpp
#include <string>
#include <vector>

std::vector<int> zSearch(
    const std::string& text,
    const std::string& pattern)
{
    std::vector<int> positions;

    if (pattern.empty())
    {
        return positions;
    }

    const char separator = '#';
    const std::string combined =
        pattern + separator + text;

    const std::vector<int> z =
        buildZArray(combined);

    const int m = static_cast<int>(pattern.size());

    for (int i = m + 1;
         i < static_cast<int>(combined.size());
         ++i)
    {
        if (z[i] >= m)
        {
            positions.push_back(i - m - 1);
        }
    }

    return positions;
}
```

#### 目前只需要記住

- KMP 的 `pi` 問「每個 Prefix 的最長 Border」。
- Z Array 問「每個位置和整體 Prefix 相同多長」。
- 兩者都能用 Prefix 資訊完成線性搜尋。
- 不需要第一次就同時熟練兩者。

---

### 48.10 方法選擇

#### 使用 Naive Matching

當：

- 資料規模小。
- 想先寫出正確答案。
- 想建立測試基準。

#### 使用 KMP

當：

- 單一 Pattern 搜尋。
- 需要確定性的 O(n + m)。
- 題目涉及 Border、Prefix Function 或重複結構。

#### 使用 Rabin-Karp

當：

- 題目自然適合 Hash。
- 需要多次比較等長 Substring。
- 願意處理 Collision，或會在 Hash 相同後驗證。

#### 使用 Z Algorithm

當：

- 題目大量詢問各位置和 Prefix 的匹配長度。
- 問題與 Prefix Match、Border 或週期結構密切相關。

#### 不要看到 String Matching 就一次使用全部方法

先看限制：

```text
O(nm) 能通過嗎？
```

- 能：Naive 可能已經足夠。
- 不能：再選 KMP、Z 或 Hash 方法。

---

### 48.11 常見題型

#### 判斷 Pattern 是否存在

- 小資料：Naive 或 `std::string::find`。
- 大資料且需要確定性線性時間：KMP 或 Z Algorithm。

#### 找所有出現位置

- Naive。
- KMP。
- Z Algorithm。
- Rabin-Karp 加二次驗證。

#### 最長 Border

Prefix Function 最後一格：

```cpp
pi.back()
```

前提是字串非空。

#### 判斷是否由重複 Pattern 組成

若字串長度為 `n`，且：

```text
borderLength = pi[n - 1]
periodLength = n - borderLength
```

若：

```text
borderLength > 0
n % periodLength == 0
```

則字串可由長度 `periodLength` 的 Prefix 重複形成。

第一次看到這個性質時，可以先用 `"ababab"` 驗證：

```text
n = 6
最長 Border = "abab"，長度 4
periodLength = 6 - 4 = 2
6 % 2 == 0
Pattern = "ab"
```

#### 多 Pattern Matching

若要同時搜尋很多 Pattern，可能使用 Trie 或 Aho-Corasick。這不是本章第一次閱讀的重點。

---

### 48.12 固定分析流程

#### 第一步：定義輸出

- 是否存在？
- 第一個位置？
- 所有位置？
- 出現次數？

#### 第二步：定義邊界

- 空 Pattern 如何處理？
- Pattern 比 Text 長時怎麼辦？
- 是否允許重疊？
- 是否區分大小寫？
- 處理 Byte、ASCII 還是 Unicode 文字單位？

#### 第三步：先寫 Naive

先列舉起點，再逐字比較。

如果限制允許 O(nm)，可以直接完成。

#### 第四步：確認瓶頸

若大量起點都比較到很後面才失敗，Naive 可能重複比較。

#### 第五步：選擇重用資訊的方法

- 重用 Border：KMP。
- 重用 Prefix Match 區間：Z Algorithm。
- 重用 Window Hash：Rabin-Karp。

#### 第六步：使用短字串手動追蹤

KMP 不要先用很長範例。先使用：

```text
pattern = "abab"
pattern = "aaaa"
pattern = "aaba"
```

手算 `pi` 後再看程式。

#### 第七步：用 Naive 做測試基準

對小型隨機輸入，同時執行 Naive 與進階方法，確認結果相同。

---

### 48.13 常見問題與判讀

#### 不知道外層迴圈的上限

Pattern 從 `start` 放入 Text 時，必須滿足：

```cpp
start + patternLength <= textLength
```

#### Naive 漏掉重疊匹配

找到後不要直接將起點增加 Pattern 長度。若要所有重疊答案，外層起點每次只加一。

#### Prefix Function 看不懂

先不要看線性版本。先做三件事：

1. 列出每個 Prefix。
2. 找該 Prefix 的所有 Proper Prefix 與 Proper Suffix。
3. 記錄最長相同長度。

這就是 `pi[i]`。

#### KMP 漏掉重疊匹配

找到一次後若將 `matched` 設為 0，可能漏掉重疊答案。應使用：

```cpp
matched = pi[matched - 1];
```

#### KMP 存取 Pattern 越界

先處理空 Pattern。搜尋過程中，一旦 `matched == m`，完成記錄並立即回退，避免下一輪存取 `pattern[m]`。

#### Rabin-Karp 偶爾誤判

可能發生 Hash Collision。若需要完全正確，Hash 相同後逐字比較。

#### Rolling Hash 出現負數

移除舊字元後應正規化：

```cpp
(windowHash - removed + mod) % mod
```

#### Z Algorithm 回傳位置偏移

對：

```text
pattern + separator + text
```

Text 的起始偏移是：

```text
pattern.size() + 1
```

因此原 Text 位置為：

```cpp
i - patternLength - 1
```

#### Separator 可能出現在輸入中

必須選擇不會出現在 Pattern 與 Text 中的分隔方式；若輸入可包含任意 Byte，單一 `char` Separator 不一定安全，應改用其他組合方式或 KMP 搜尋流程。

---

### 48.14 本章檢查表

#### 第一輪：Naive

- 我知道 Text 與 Pattern 各自代表什麼。
- 我能列舉 Pattern 的所有可能起點。
- 我知道為什麼條件是 `start + m <= n`。
- 我能用內層迴圈逐字比較。
- 我知道找到一個不同字元就可以停止目前起點。
- 我知道 Naive 最差時間是 O(nm)。

#### 第二輪：KMP

- 我能區分 Prefix 與 Suffix。
- 我知道 Proper 不包含完整字串本身。
- 我能找出短字串的 Border。
- 我知道 `pi[i]` 是 `pattern[0..i]` 的最長 Border 長度。
- 我能手算 `"ababd"` 的 Prefix Function。
- 我知道 KMP 的 `matched` 表示已匹配的 Pattern Prefix 長度。
- 我知道失敗時使用 `pi` 回退，而不是讓 Text 回頭。
- 我知道找到答案後仍需回退，以保留重疊匹配。

#### 第三輪：延伸方法

- 我知道 Hash 不同一定不匹配，但 Hash 相同可能 Collision。
- 我知道要求完全正確時要做二次驗證。
- 我知道 `z[i]` 表示位置 `i` 與整體 Prefix 的最長匹配長度。
- 我知道第一次閱讀不需要同時熟練 KMP、Rabin-Karp 與 Z Algorithm。

---

### 48.15 本章重點

1. String Matching 的基本問題，是尋找 Pattern 在 Text 中的完整出現位置。
2. 實作前先定義空 Pattern、重疊匹配、大小寫與文字單位。
3. Naive Matching 列舉每個起點，再逐字比較；它簡單、正確，而且常已足夠。
4. Naive 的最差時間為 O(nm)，瓶頸是失敗後重複比較已知內容。
5. Border 是同時為 Proper Prefix 與 Proper Suffix 的內容。
6. Prefix Function 記錄每個 Pattern Prefix 的最長 Border 長度。
7. KMP 失敗時使用 Border 回退 Pattern 的匹配長度，Text 不需要回頭。
8. 學 KMP 時先手算 `pi`，再追蹤 `matched`，不要一次要求自己完成全部程式。
9. Rabin-Karp 使用 Rolling Hash，但 Hash Collision 必須納入正確性考量。
10. Z Algorithm 記錄各位置與整體 Prefix 的最長匹配長度。
11. 第一次閱讀先學會 Naive，再學 KMP；Rabin-Karp 與 Z Algorithm 可以稍後處理。
