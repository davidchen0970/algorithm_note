# 第 6 章　String 與字元處理

## 適用範圍

本章從最基本的 `std::string` 使用方式開始，逐步建立字串題需要的觀念。第一次閱讀時，不需要先理解 Unicode、Dynamic Programming 或 `string_view` 的全部細節。每一節都會先說明「目前要解決什麼問題」，再建立直覺、寫出程式，最後補上邊界與進階內容。

本章依照下列順序前進：

1. 先把 `std::string` 當成可以依序存放 `char` 的容器。
2. 學會長度、Index、走訪與區間。
3. 再釐清 Byte、ASCII 與 UTF-8 的差異。
4. 分清 Substring 與 Subsequence。
5. 使用 Two Pointers 解回文。
6. 使用 Frequency Array 解字元統計與 Anagram。
7. 理解 `substr` 的功能與複製成本。
8. 再學習不擁有資料的 `string_view`。
9. 最後處理字串建構、解析、C String 與 Unicode 邊界。

本章的核心不是記住所有 API，而是每次看到字串題時，能依序回答：

- 我現在處理的是 Byte、ASCII 字元，還是 Unicode 文字？
- 題目要求連續片段，還是只要求保持順序？
- 比較時是否忽略大小寫、空白或標點？
- 我是否在不必要地複製字串？
- 如果使用 View，原始資料會不會先失效？

## 適用讀者

- 第一次使用 C++ `std::string` 解題的讀者。
- 容易混淆 Substring、Subsequence 與 Subset 的讀者。
- 知道 Two Pointers，但不知道如何套用到字串的讀者。
- 看過 UTF-8、Code Point、Grapheme Cluster 等名詞，但尚未建立清楚關係的讀者。
- 使用 `substr` 或 `string_view` 時，不清楚複製成本與生命週期的讀者。
- 需要處理 Token、Delimiter 或 C String 的讀者。

## 閱讀方式

### 第一輪必讀

建議依序閱讀 6.1 至 6.10。第一輪先掌握：

- `std::string` 的基本模型。
- Substring 與 Subsequence 的差異。
- 回文與頻率統計的基本方法。
- `substr` 的用途與成本。

### 第二輪再讀

6.11 至 6.14 涉及較多 C++ 工程細節，包括 `string_view`、資料生命週期、解析規格與 C String。

### 延伸內容

6.10.1 的最長共同子串 DP 是延伸內容。若尚未學過 Dynamic Programming，可以先看問題分類、暴力解法與 `substr` 的角色，DP 部分可等讀完 Dynamic Programming 章節後再回來。

## 快速導覽

- [6.1 先把 String 當成一排 char](#61-先把-string-當成一排-char)
- [6.2 長度、Index、走訪與區間](#62-長度index走訪與區間)
- [6.3 Byte、ASCII 與 UTF-8](#63-byteascii-與-utf-8)
- [6.4 先定義比較規則](#64-先定義比較規則)
- [6.5 Substring 與 Subsequence](#65-substring-與-subsequence)
- [6.6 完整案例：判斷 Subsequence](#66-完整案例判斷-subsequence)
- [6.7 從最簡單的回文開始](#67-從最簡單的回文開始)
- [6.8 完整案例：忽略標點與大小寫的回文](#68-完整案例忽略標點與大小寫的回文)
- [6.9 字元頻率與 Anagram](#69-字元頻率與-anagram)
- [6.10 substr：切出內容與複製成本](#610-substr切出內容與複製成本)
- [6.10.1 從共同子串問題理解 substr 的角色](#6101-從共同子串問題理解-substr-的角色)
- [6.11 string_view：只看資料，不擁有資料](#611-string_view只看資料不擁有資料)
- [6.12 有效率地建構字串](#612-有效率地建構字串)
- [6.13 字串解析與 Split](#613-字串解析與-split)
- [6.14 補充：C String](#614-補充c-string)
- [6.15 字串題的固定分析流程](#615-字串題的固定分析流程)
- [6.16 常見問題與判讀](#616-常見問題與判讀)
- [6.17 本章檢查表](#617-本章檢查表)
- [6.18 本章重點](#618-本章重點)


## 6.1 先把 String 當成一排 char

### 這一節要解決什麼問題

在討論 UTF-8、回文或 `string_view` 以前，先建立最簡單的模型：

> `std::string` 是一個依序存放 `char` 的容器。

先看最基本的字串：

```cpp
#include <string>

std::string text = "abc";
```

可以暫時把它想成：

```text
Index:  0    1    2
Value: 'a'  'b'  'c'
```

因此：

```cpp
text.size();   // 3
text[0];       // 'a'
text[1];       // 'b'
text[2];       // 'c'
```

這個模型對只含 ASCII 的演算法題非常好用。後面才會補充它在 UTF-8 文字中的限制。

### 字串和 Array 的相似之處

`std::string` 與 Array 或 `std::vector` 有許多相似用法：

- 可以取得元素數量。
- 可以使用 Index 讀取元素。
- 可以從左到右走訪。
- 可以改寫既有元素。
- 可以在尾端加入內容。

```cpp
std::string text = "cat";
text[0] = 'b';
text.push_back('s');

// text 現在是 "bats"
```

### 空字串

```cpp
std::string text;
```

此時：

```cpp
text.empty();  // true
text.size();   // 0
```

空字串沒有可合法存取的元素，所以不能讀取 `text[0]`。

### `size()` 的型別

`text.size()` 回傳 `std::size_t`，它是 Unsigned Integer Type。常見走訪方式如下：

```cpp
for (std::size_t i = 0; i < text.size(); ++i)
{
    process(text[i]);
}
```

因為 `std::size_t` 不表示負數，稍後從右向左走訪時，必須特別注意 `size() - 1` 在空字串上的問題。

### 先記住

- `std::string` 可以依 Index 存取 `char`。
- 合法 Index 是 `[0, size())`。
- 空字串的 `size()` 是 0，不能存取 `text[0]`。
- 目前先假設輸入只含 ASCII，6.3 再處理 UTF-8。


## 6.2 長度、Index、走訪與區間

### Range-based for：只需要元素

如果不需要 Index，可以直接走訪每個 `char`：

```cpp
for (char ch : text)
{
    process(ch);
}
```

這種寫法適合：

- 計算每個字元的出現次數。
- 判斷是否包含某種字元。
- 將每個字元依序輸出。

### Index-based for：需要位置

如果需要目前位置、前一個元素或後一個元素，使用 Index：

```cpp
for (std::size_t i = 0; i < text.size(); ++i)
{
    process(i, text[i]);
}
```

### Half-open Interval

本章統一使用 Half-open Interval：

```text
[begin, end)
```

它表示：

- 包含 `begin`。
- 不包含 `end`。
- 長度是 `end - begin`。

例如，字串 `"abcde"` 中的 `"bcd"` 可以表示為：

```text
[1, 4)
```

因為包含 Index 1、2、3，但不包含 Index 4。

### 為什麼使用 `[begin, end)`

它有三個直接好處：

1. 長度就是 `end - begin`。
2. 空範圍可以寫成 `[x, x)`。
3. 整個字串可以寫成 `[0, text.size())`。

### `substr` 的第二個參數不是右邊界

```cpp
std::string part = text.substr(position, count);
```

第二個參數是數量 `count`，不是 `end`。

若要取出 `[left, right)`：

```cpp
std::string part = text.substr(left, right - left);
```

例如：

```cpp
std::string text = "abcde";
std::string part = text.substr(1, 3);

// part 是 "bcd"
```

### 常見錯誤：把右邊界當成數量

```cpp
std::string part = text.substr(1, 4);
```

這不是取 `[1, 4)`，而是「從 Index 1 開始取 4 個元素」，結果是 `"bcde"`。

### 先記住

- 只需要元素時用 Range-based for。
- 需要位置時用 Index。
- 本章使用 `[begin, end)`。
- `substr(position, count)` 的第二個參數是數量。


## 6.3 Byte、ASCII 與 UTF-8

### 為什麼前面說「先假設 ASCII」

`std::string` 保存的是一段 `char` 序列。更精確地說，我們通常把其中的資料視為 Bytes，而不是直接視為人類看到的完整字元。

如果內容是 ASCII：

```cpp
std::string text = "abc";
```

通常一個英文字母對應一個 Byte，因此：

```text
text.size() == 3
```

而且每個 Index 剛好對應一個英文字母。

### UTF-8 使用可變長度編碼

UTF-8 中，一個 Unicode Code Point 可能使用 1 到 4 個 Bytes。因此：

```cpp
text.size()
```

回傳的是 Byte 數量，不保證等於：

- Unicode Code Point 數量。
- 使用者看到的字形數量。
- 游標移動一次所跨越的文字單位。

### 三個容易混在一起的層級

#### Byte

`std::string` 可以逐 Index 讀到的儲存單位。

#### Unicode Code Point

Unicode 定義的抽象文字單位，例如某個字母或符號的編號。

#### Grapheme Cluster

使用者通常認為的一個完整可見字形。它可能由多個 Code Points 組成，例如基底字母加上組合符號，或由多個 Code Points 組成的 Emoji。

### 為什麼不能直接逐 Byte 反轉 UTF-8

若一個文字單位由多個 Bytes 組成，逐 Byte 交換位置可能打亂編碼順序，產生無效或錯誤文字。

因此，題目若要求：

> 反轉使用者看到的字元

就必須先確認目標單位是 Code Point 還是 Grapheme Cluster，不能直接假設 `text[i]` 是完整字元。

### 什麼情況可以逐 `char` 處理

可以：

- 題目明確保證只含英文字母。
- 題目明確保證只含 ASCII。
- 需求本來就是處理原始 Byte 序列。

不應直接如此處理：

- 完整 Unicode 大小寫轉換。
- 依使用者可見字形反轉。
- 依 Code Point 計算字數。
- Unicode 正規化。

### 內含 Null Byte

`std::string` 會另外保存長度，所以內容可以包含 `\0`：

```cpp
std::string data{"a\0b", 3};
```

此時：

```cpp
data.size() == 3
```

但如果將 `data.c_str()` 傳給只依 `\0` 判斷結尾的 C 介面，對方可能只讀到 `a`。

### 本章後續的預設

除非小節另有說明，後續演算法案例都假設輸入是 ASCII，或需求本來就是逐 Byte 比較。

這項限制不是附帶細節，而是演算法正確性的一部分。

### 先記住

- `std::string::size()` 回傳 Byte 數量。
- ASCII 常可逐 `char` 處理。
- UTF-8 的一個文字單位可能占用多個 Bytes。
- 看到「字元」一詞時，要先問它指哪一層。


## 6.4 先定義比較規則

### 同一個輸入可能有不同答案

考慮：

```text
A man, a plan, a canal: Panama
```

若逐 Byte 比較、區分大小寫並保留標點，它不是回文。

若規格改成：

- 忽略非英數字元。
- 英文字母不區分大小寫。

它就可以被判定為回文。

差異不在於哪個演算法比較正確，而在於兩者的 Postcondition 不同。

### 寫程式前先回答

- 輸入是否只含 ASCII？
- 是否區分英文大小寫？
- 是否忽略空白？
- 是否忽略標點？
- 是否保留數字？
- 空字串的答案是什麼？
- 輸出位置是 Byte Index，還是其他文字單位的位置？
- 是否允許修改輸入字串？

### 比較規則不代表一定要先修改字串

「忽略非英數字元」描述的是如何比較，不代表一定要先建立修改後的新字串。

可以選擇兩種方向：

#### 方向一：先整理，再執行演算法

```text
原字串
    ↓ 移除不需要的內容、統一大小寫
整理後的新字串
    ↓
回文比較
```

優點：

- 整理與比較分開，容易閱讀。
- 整理後結果可重複使用。
- 每個階段容易單獨測試。

代價：

- 需要建立新字串。
- 通常需要 O(n) 額外空間。

#### 方向二：走訪時直接套用規則

```text
原字串
    ↓ 指標移動時略過不需要的內容
直接比較
```

優點：

- 不必建立完整副本。
- 回文案例可以做到 O(1) 額外空間。

代價：

- 迴圈同時處理略過、轉換與比較。
- 邊界條件較多。

6.8 會完整比較這兩種方式。

### 用詞提醒

本章前半部若提到「整理比較內容」，主要指忽略標點、忽略空白或統一 ASCII 大小寫。Unicode Normalization Form 是另一個更完整的文字規格，不應直接混為一談。

### 先記住

- 比較規則必須先定義。
- 規格描述「比較什麼」，實作再決定「何時整理」。
- 不要還沒確認規格，就先選演算法。


## 6.5 Substring 與 Subsequence

這兩個詞很像，但它們描述不同的候選結構。

### Substring：連續區間

Substring 是原字串中的連續片段。

對：

```text
abcde
```

以下都是 Substring：

```text
abc
bcd
e
空字串
```

`ace` 不是 Substring，因為它在原字串中不連續。

用區間表示時，Substring 就是一段 `[left, right)`。

### Subsequence：保持順序，可以跳過

Subsequence 必須保留原順序，但可以跳過部分元素。

```text
ace
```

是 `abcde` 的 Subsequence，因為可以依序選取 Index 0、2、4。

但：

```text
aec
```

不是 `abcde` 的 Subsequence，因為順序改變了。

### Subset：通常不處理順序

Subset 通常只關心選了哪些元素，不強調原本順序。字串題多半問 Substring 或 Subsequence，不能看到「選一些字元」就忽略順序。

### 用一張圖建立直覺

```text
原字串： a b c d e
Index：  0 1 2 3 4

Substring "bcd"：
          └─────┘       連續

Subsequence "ace"：
          ↑   ↑   ↑     可以跳過，但順序不變
```

### 題目用語如何判斷

常見線索：

- 「連續片段」、「子字串」：通常是 Substring。
- 「刪除若干字元後得到」、「保持相對順序」：通常是 Subsequence。
- 「任選若干元素」、「不考慮順序」：可能是 Subset。

### 問題模型會改變解法

- 最長不重複 Substring：常用 Sliding Window。
- 判斷是否為 Subsequence：常用 Two Pointers。
- 最長共同 Subsequence：常用 Dynamic Programming。
- 最長共同 Substring：也可用 Dynamic Programming，但狀態轉移不同。

先分清問題模型，再選方法。


## 6.6 完整案例：判斷 Subsequence

### 問題

給定 `candidate` 與 `text`，判斷 `candidate` 是否為 `text` 的 Subsequence。

```text
candidate = "ace"
text      = "abcde"
答案      = true
```

### 先用人類方式思考

我們想依序在 `text` 中找到：

1. 先找 `a`。
2. 找到後，再往右找 `c`。
3. 找到後，再往右找 `e`。
4. 全部找到就成功。

重點是不能回頭。

### 需要記住什麼狀態

只需要一個 Index：

```text
matched = 下一個想在 text 中找到的 candidate 位置
```

開始時：

```text
matched = 0
```

表示下一個要找 `candidate[0]`。

### 逐步走一次

```text
candidate = a c e
text      = a b c d e

看到 a：等於 candidate[0]，matched 變成 1
看到 b：不等於 candidate[1]，略過
看到 c：等於 candidate[1]，matched 變成 2
看到 d：不等於 candidate[2]，略過
看到 e：等於 candidate[2]，matched 變成 3

matched == candidate.size()，成功
```

### C++ 解法

```cpp
#include <string_view>

bool isSubsequence(
    std::string_view candidate,
    std::string_view text)
{
    std::size_t matched = 0;

    for (char ch : text)
    {
        if (matched < candidate.size() &&
            candidate[matched] == ch)
        {
            ++matched;
        }
    }

    return matched == candidate.size();
}
```

### 為什麼這樣可行

每次遇到下一個需要的字元，就使用目前最早的位置配對。

選擇較早的位置不會減少後方可用範圍。若故意放棄目前位置，改用更晚的相同字元，只會讓剩餘可搜尋範圍更短。

### State 與 Loop Invariant

第一次閱讀可以先理解程式，再讀這一段。

`matched` 表示：

```text
candidate[0, matched)
```

已經依序出現在目前走訪過的 `text` Prefix 中。

每輪開始前：

1. `candidate[0, matched)` 已完成配對。
2. 這些配對位置保持嚴格遞增。
3. `matched` 不會超過 `candidate.size()`。

### 邊界案例

- `candidate` 為空：回傳 `true`，因為空字串是任何字串的 Subsequence。
- `text` 為空而 `candidate` 非空：回傳 `false`。
- `candidate.size() > text.size()`：一定是 `false`，現有流程已能自然處理。
- 重複字元仍按順序配對，例如 `"aa"` 是 `"aba"` 的 Subsequence。

### 複雜度

- 時間：O(text.size())。
- 額外空間：O(1)。


## 6.7 從最簡單的回文開始

### 什麼是回文

回文表示從左向右與從右向左讀取相同。

```text
level
abba
racecar
```

### 最直接的想法

可以先反轉字串，再比較是否相同。但這需要建立反轉後的結果。

另一種方式是同時比較左右兩端：

```text
l e v e l
↑       ↑
left  right
```

若兩端相同，就向中間移動。

### 為什麼 `right` 不直接設成 `size() - 1`

如果字串為空：

```cpp
text.size() == 0
```

而 `std::size_t` 是 Unsigned Type，計算 `0 - 1` 會產生 Underflow。

因此本章讓 `right` 先表示 Exclusive End：

```cpp
std::size_t right = text.size();
```

真正比較右側元素前，再使用 `--right`。

### C++ 解法

```cpp
#include <string_view>

bool isPalindrome(std::string_view text)
{
    std::size_t left = 0;
    std::size_t right = text.size();

    while (left < right)
    {
        --right;

        if (text[left] != text[right])
        {
            return false;
        }

        ++left;
    }

    return true;
}
```

### 逐步理解 `"abba"`

```text
初始：[0, 4)
比較 text[0] 與 text[3]：a == a

剩下：[1, 3)
比較 text[1] 與 text[2]：b == b

剩下：[2, 2)
沒有尚未比較的內容，成功
```

### 本版本的規格

這個版本：

- 逐 Byte 比較。
- 區分大小寫。
- 不忽略空白。
- 不忽略標點。

因此 `"Aba"` 不是回文，`"a a"` 則是回文。

### Loop Invariant

每輪開始前：

- `[0, left)` 與 `[right, size())` 已完成鏡像配對。
- 已比較的每一組內容都相同。
- `[left, right)` 是尚未完成比較的範圍。

### 複雜度

- 時間：O(n)。
- 額外空間：O(1)。


## 6.8 完整案例：忽略標點與大小寫的回文

### 問題規格

判斷 ASCII 字串在以下規則下是否為回文：

- 忽略非英數字元。
- 英文字母不區分大小寫。
- 數字保留並參與比較。

### 第一步：定義 ASCII 分類

```cpp
bool isAsciiAlphaNumeric(char ch)
{
    const bool digit = ch >= '0' && ch <= '9';
    const bool lower = ch >= 'a' && ch <= 'z';
    const bool upper = ch >= 'A' && ch <= 'Z';

    return digit || lower || upper;
}
```

### 第二步：定義 ASCII 小寫轉換

```cpp
char lowerAscii(char ch)
{
    if (ch >= 'A' && ch <= 'Z')
    {
        return static_cast<char>(ch - 'A' + 'a');
    }

    return ch;
}
```

這裡刻意只處理 ASCII，避免讓函式名稱暗示它能完成所有 Unicode 大小寫轉換。

### 第三步：左右略過不需比較的內容

```cpp
#include <string_view>

bool isNormalizedPalindrome(std::string_view text)
{
    std::size_t left = 0;
    std::size_t right = text.size();

    while (left < right)
    {
        while (left < right &&
               !isAsciiAlphaNumeric(text[left]))
        {
            ++left;
        }

        while (left < right &&
               !isAsciiAlphaNumeric(text[right - 1]))
        {
            --right;
        }

        if (left >= right)
        {
            break;
        }

        if (lowerAscii(text[left]) !=
            lowerAscii(text[right - 1]))
        {
            return false;
        }

        ++left;
        --right;
    }

    return true;
}
```

### 迴圈分成哪幾件事

每一輪依序處理：

1. 左指標略過非英數內容。
2. 右指標略過非英數內容。
3. 確認是否還有需要比較的範圍。
4. 將左右字元轉成相同大小寫後比較。
5. 兩端都向中間移動。

將流程拆開後，程式會比一次閱讀整個 `while` 容易理解。

### 為什麼不一定要先建立整理後的字串

本解法在指標移動時直接套用比較規則，所以不需要建立完整副本，額外空間為 O(1)。

另一種寫法是先建立新字串：

```cpp
#include <string>
#include <string_view>

std::string keepLowercaseAsciiAlphaNumeric(
    std::string_view text)
{
    std::string cleaned;
    cleaned.reserve(text.size());

    for (char ch : text)
    {
        if (isAsciiAlphaNumeric(ch))
        {
            cleaned.push_back(lowerAscii(ch));
        }
    }

    return cleaned;
}
```

接著再對 `cleaned` 呼叫一般回文函式。

兩種方法比較：

- 先建立新字串：流程較容易拆解與測試，需要 O(n) 額外空間。
- 比較時直接略過：不建立完整副本，需要更仔細處理左右邊界。

如果整理後結果還會用於其他工作，建立新字串可能更合理。如果只比較一次，而且空間限制嚴格，直接使用 Two Pointers 較合適。

### `<cctype>` 的安全呼叫方式

也可以使用 `std::isalnum`、`std::tolower` 等函式，但傳入值必須等於 `EOF`，或可表示為 `unsigned char`。直接傳入負值 `char` 可能造成 Undefined Behavior。

安全包裝方式：

```cpp
#include <cctype>

bool isAlphaNumeric(char ch)
{
    const auto value = static_cast<unsigned char>(ch);
    return std::isalnum(value) != 0;
}

char toLower(char ch)
{
    const auto value = static_cast<unsigned char>(ch);
    return static_cast<char>(std::tolower(value));
}
```

這些函式也可能受到目前 Locale 影響。若題目明確限定 ASCII，直接寫出 ASCII 範圍通常更容易看出規格。

### 邊界案例

- 空字串：`true`。
- 單一英數字元：`true`。
- 只含標點：`true`，因為整理後為空字串。
- `"A1a"`：`true`。
- `"ab"`：`false`。
- `"A man, a plan, a canal: Panama"`：`true`。

### Unicode 限制

本案例只定義 ASCII 分類與大小寫轉換。若需求包含完整 Unicode Case Folding、Normalization Form 或 Grapheme Cluster，應使用適合的 Unicode 函式庫，不能直接將這組函式視為完整 Unicode 解法。


## 6.9 字元頻率與 Anagram

### 核心問題

頻率統計是在回答：

```text
每個可能值出現幾次？
```

在選資料結構以前，先問：

> 可能的鍵值範圍是否小而固定？

### 只含小寫英文字母

如果題目保證每個字元都在 `'a'` 到 `'z'`，可以使用 26 格 Array。

```text
'a' -> 0
'b' -> 1
...
'z' -> 25
```

```cpp
#include <array>
#include <string_view>

std::array<std::size_t, 26> countLowercase(
    std::string_view text)
{
    std::array<std::size_t, 26> frequency{};

    for (char ch : text)
    {
        const auto index =
            static_cast<std::size_t>(ch - 'a');
        ++frequency[index];
    }

    return frequency;
}
```

### 為什麼 Precondition 很重要

若 `ch` 不是小寫英文字母：

```cpp
ch - 'a'
```

可能不在 `[0, 26)`，接著存取 Array 就可能越界。

固定大小 Array 的速度與簡潔，建立在「值域假設成立」之上。

### 任意 Byte

若需要統計任意 Byte，可以使用 256 格 Array：

```cpp
#include <array>
#include <string_view>

std::array<std::size_t, 256> countBytes(
    std::string_view text)
{
    std::array<std::size_t, 256> frequency{};

    for (unsigned char byte : text)
    {
        ++frequency[byte];
    }

    return frequency;
}
```

### 何時使用 Hash Map

適合使用固定 Array：

- 值域小。
- 值域固定。
- 可以直接將值映射成 Index。

適合使用 Hash Map：

- Key 範圍很大。
- 實際出現的 Key 很少。
- Key 不是容易映射成小範圍 Index 的型別。

若需要統計解碼後的 Unicode Code Points，應先正確解碼 UTF-8，再依需求選擇 Map 或其他結構。

### 完整案例：Anagram

若兩個小寫 ASCII 字串互為 Anagram，每個字母在兩者中的出現次數必須相同。

```cpp
#include <array>
#include <string_view>

bool areAnagrams(
    std::string_view first,
    std::string_view second)
{
    if (first.size() != second.size())
    {
        return false;
    }

    std::array<int, 26> difference{};

    for (std::size_t i = 0; i < first.size(); ++i)
    {
        const auto firstIndex =
            static_cast<std::size_t>(first[i] - 'a');
        const auto secondIndex =
            static_cast<std::size_t>(second[i] - 'a');

        ++difference[firstIndex];
        --difference[secondIndex];
    }

    for (int value : difference)
    {
        if (value != 0)
        {
            return false;
        }
    }

    return true;
}
```

### 為什麼一加一減

- `first` 出現某字母時，差值加一。
- `second` 出現同一字母時，差值減一。
- 最後全部回到零，代表每個字母數量相同。

### Precondition

兩個輸入都只包含 `'a'` 到 `'z'`。

若規格允許大寫、空白或 Unicode，必須先重新定義比較規則與鍵值範圍。

### 複雜度

- 時間：O(n)。
- 額外空間：O(1)，因為 Array 大小固定為 26。


## 6.10 `substr`：切出內容與複製成本

### `std::string::substr` 做什麼

```cpp
std::string part = text.substr(position, count);
```

它從已知位置開始，建立一個新的 `std::string`。

若複製長度為 `k`，時間與額外空間通常至少與 `k` 成正比。

### 單次使用通常不是問題

```cpp
std::string filename = "report.txt";
std::string extension = filename.substr(7, 3);
```

若只建立一次短字串，這通常很自然。

問題多半發生在迴圈或遞迴中反覆建立越來越大的副本。

### 遞迴中反覆複製

```cpp
bool solve(std::string text)
{
    if (text.empty())
    {
        return true;
    }

    return solve(text.substr(1));
}
```

長度為 `n` 時，各層可能分別複製約：

```text
n - 1, n - 2, ..., 1
```

總複製量可能形成 O(n²)。函式參數按值接收，也會產生額外複製。

### 改傳原字串與 Index

```cpp
bool solve(
    const std::string& text,
    std::size_t position)
{
    if (position == text.size())
    {
        return true;
    }

    return solve(text, position + 1);
}
```

這裡所有遞迴層共用同一個原字串，只傳遞位置。

### 改傳區間

如果子問題是一段連續範圍，可以傳：

```text
[left, right)
```

這能表示：

- 子問題的起點。
- 子問題的終點。
- 空範圍。
- 子問題長度 `right - left`。

### 改用 `string_view`

```cpp
#include <string_view>

bool solve(std::string_view text)
{
    if (text.empty())
    {
        return true;
    }

    return solve(text.substr(1));
}
```

`string_view::substr` 只建立新的 View，不複製底層字元。但它帶來生命週期要求，6.11 會完整說明。

### 先記住

- `std::string::substr` 會建立擁有資料的新字串。
- 少量使用很自然。
- 反覆使用前要估算總複製量。
- Index、區間與 `string_view` 都能表示子問題，但適用條件不同。

### 6.10.1 從共同子串問題理解 `substr` 的角色

> 延伸閱讀：若尚未學過 Dynamic Programming，可以先讀到暴力解法。DP 部分可稍後再回來。

本節透過共同子串問題區分兩件事：

1. 尋找答案的位置與長度。
2. 已知位置與長度後，切出答案內容。

`substr` 只負責第二件事。

#### 固定 Pattern 搜尋

如果已知要找的 Pattern：

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string first = "abcabc";
    std::string second = "abcabcabcabc";
    std::string pattern = "abc";

    if (first.find(pattern) != std::string::npos &&
        second.find(pattern) != std::string::npos)
    {
        std::cout << pattern
                  << " is a common substring\n";
    }
}
```

這裡使用 `find` 搜尋固定 Pattern，不需要改寫 `substr`。

#### 最長共同子串的暴力想法

若 Pattern 未知，題目要求自動找出兩個字串的最長連續共同片段，可以：

1. 枚舉 `first` 的起點。
2. 枚舉 `second` 的起點。
3. 從兩個起點同步向右比較。
4. 記錄目前最長的連續配對長度。

```cpp
#include <string>

std::string longestCommonSubstring(
    const std::string& first,
    const std::string& second)
{
    std::size_t bestBegin = 0;
    std::size_t bestLength = 0;

    for (std::size_t i = 0; i < first.size(); ++i)
    {
        for (std::size_t j = 0; j < second.size(); ++j)
        {
            std::size_t length = 0;

            while (i + length < first.size() &&
                   j + length < second.size() &&
                   first[i + length] == second[j + length])
            {
                ++length;
            }

            if (length > bestLength)
            {
                bestBegin = i;
                bestLength = length;
            }
        }
    }

    return first.substr(bestBegin, bestLength);
}
```

這段程式中：

- 雙迴圈選擇兩個起點。
- `while` 找出從起點開始的共同長度。
- `bestBegin` 與 `bestLength` 記錄答案區間。
- 最後才由 `substr` 切出答案。

#### 為什麼這不是 Subsequence

只要中間出現不同字元，目前的共同長度就停止，因為 Substring 必須連續。

若題目是最長共同 Subsequence，則允許跳過元素，問題模型與狀態轉移都不同。

#### 延伸：Dynamic Programming

定義：

```text
dp[i][j] = 以 first[i - 1] 與 second[j - 1] 結尾的
           最長共同連續後綴長度
```

轉移：

```text
若 first[i - 1] == second[j - 1]
    dp[i][j] = dp[i - 1][j - 1] + 1
否則
    dp[i][j] = 0
```

遇到不同字元時歸零，是因為 Substring 必須連續。

```cpp
#include <string>
#include <vector>

std::string longestCommonSubstringDp(
    const std::string& first,
    const std::string& second)
{
    const std::size_t n = first.size();
    const std::size_t m = second.size();

    std::vector<std::vector<std::size_t>> dp(
        n + 1,
        std::vector<std::size_t>(m + 1, 0));

    std::size_t bestLength = 0;
    std::size_t bestEnd = 0;

    for (std::size_t i = 1; i <= n; ++i)
    {
        for (std::size_t j = 1; j <= m; ++j)
        {
            if (first[i - 1] == second[j - 1])
            {
                dp[i][j] = dp[i - 1][j - 1] + 1;

                if (dp[i][j] > bestLength)
                {
                    bestLength = dp[i][j];
                    bestEnd = i;
                }
            }
        }
    }

    return first.substr(
        bestEnd - bestLength,
        bestLength);
}
```

DP 決定 `bestEnd` 與 `bestLength`，`substr` 仍只負責把已知答案區間建立成新字串。

#### 本節真正要帶走的觀念

- 已知 Pattern：可先考慮 `find`。
- 未知共同區間：需要搜尋方法。
- `substr` 不會替你決定答案在哪裡。
- Substring 與 Subsequence 的連續性不同，不能共用同一套轉移規則。


## 6.11 `string_view`：只看資料，不擁有資料

### 為什麼需要 `string_view`

有時函式只想讀取字串，不需要：

- 修改內容。
- 保存獨立副本。
- 接管資料生命週期。

此時可以接收 `std::string_view`。

### 最重要的直覺

> `string_view` 自己沒有保存字元內容。它只記住「從哪裡開始」與「看多長」。

可以想成：

```text
原始資料： [ a b c d e ]
              ↑─────↑
View：       起點 + 長度
```

### 它通常保存什麼

概念上包含：

- 指向某段字元資料的位置。
- 該範圍的長度。

它不負責延長原始資料的生命週期。

### 安全案例

```cpp
#include <string_view>

std::string_view middle(std::string_view text)
{
    if (text.size() < 2)
    {
        return {};
    }

    return text.substr(1, text.size() - 2);
}
```

回傳 View 是否安全，取決於呼叫端提供的原始資料是否仍然存在。

### 懸空 View

```cpp
std::string_view makeView()
{
    std::string local = "temporary";
    return local;
}
```

函式結束時，`local` 被銷毀。回傳的 View 仍記得舊位置，但該位置已不再代表有效字串資料。

暫時物件也有相同風險：

```cpp
std::string_view view = std::string("temporary");
```

完整運算式結束後，暫時字串被銷毀，`view` 失效。

### 原字串修改也可能讓 View 失效

```cpp
std::string text = "abc";
std::string_view view = text;

text.push_back('d');
```

如果 `push_back` 引發 Reallocation，原本的儲存位置會改變，View 便可能失效。

即使沒有 Reallocation，刪除或重排內容也可能讓 View 所代表的語意不再符合原本預期。

### View 不保證以 Null Terminator 結尾

```cpp
std::string_view part(text.data() + 2, 3);
```

`part.data()` 指向原字串中間，View 結尾處不一定有 `\0`。

因此不能直接將 `part.data()` 當成完整 C String 使用。若 C 介面需要 Null-terminated String，可以建立擁有資料的字串：

```cpp
std::string owned(part);
useCFunction(owned.c_str());
```

### 何時適合使用

適合：

- 函式只在呼叫期間讀取資料。
- 需要切分大量範圍，但不想建立副本。
- 原始資料生命週期清楚且足夠長。

不適合直接保存：

- 原始資料可能很快被銷毀。
- 原字串會被修改或重新配置。
- 結果需要獨立 Ownership。
- 需要穩定的 Null-terminated C String。

### `std::string` 與 `string_view` 的選擇

問自己：

> 這個結果需要擁有自己的資料嗎？

- 需要：使用 `std::string`。
- 不需要，而且原資料一定活得夠久：可以考慮 `string_view`。
- 無法確定生命週期：優先選擇擁有資料的型別。


## 6.12 有效率地建構字串

### 尾端追加

```cpp
std::string result;
result.push_back('a');
result += "bc";
```

字串容量足夠時，尾端追加通常不需要搬移全部既有內容。

### 已知大致長度時使用 `reserve`

```cpp
std::string result;
result.reserve(expectedLength);

for (char ch : input)
{
    if (shouldKeep(ch))
    {
        result.push_back(ch);
    }
}
```

`reserve` 只調整 Capacity，不會改變 `size()`，也不會建立可以用 Index 存取的新元素。

錯誤觀念：

```cpp
std::string result;
result.reserve(10);
result[0] = 'a';  // 錯誤，size() 仍是 0
```

### 預先建立實際大小

若最終長度可以準確計算：

```cpp
std::string result(length, '\0');

for (std::size_t i = 0; i < length; ++i)
{
    result[i] = computeCharacter(i);
}
```

這時 `size()` 已是 `length`，可以合法寫入 `[0, length)`。

### 避免反覆從前端插入

```cpp
result.insert(result.begin(), ch);
```

前端插入可能搬移全部既有內容。重複 `n` 次，總搬移量可能形成 O(n²)。

若要建立反向結果，可以：

- 從原輸入尾端向前走訪，再 `push_back`。
- 先尾端加入，再呼叫 `std::reverse`。
- 預先建立完整大小，再依 Index 寫入。

### `result = result + piece`

```cpp
result = result + piece;
```

這種寫法可能建立暫時字串並複製舊內容。若語意只是追加，通常使用：

```cpp
result += piece;
```

更直接。

### 先記住

- `reserve` 改變 Capacity，不改變 Size。
- `resize` 或建構指定長度才會建立元素。
- 尾端追加通常比反覆前端插入更合適。
- 先估算最終長度，可以減少重新配置。


## 6.13 字串解析與 Split

### 「用逗號切開」還不算完整規格

開始實作以前，至少要回答：

- 連續 Delimiter 是否產生空 Token？
- 開頭 Delimiter 是否產生第一個空 Token？
- 結尾 Delimiter 是否產生最後一個空 Token？
- Token 前後空白是否保留？
- 空輸入代表零個 Token，還是一個空 Token？
- 是否允許引號或 Escape？
- 遇到非法內容時如何回報？

### 本案例的規格

以下 Split：

- 保留空 Token。
- 空輸入得到一個空 Token。
- 結尾 Delimiter 產生最後一個空 Token。
- 不移除 Token 前後空白。
- 不處理引號與 Escape。

### C++ 解法

```cpp
#include <string_view>
#include <vector>

std::vector<std::string_view> split(
    std::string_view text,
    char delimiter)
{
    std::vector<std::string_view> result;
    std::size_t begin = 0;

    while (true)
    {
        const std::size_t end =
            text.find(delimiter, begin);

        if (end == std::string_view::npos)
        {
            result.push_back(text.substr(begin));
            break;
        }

        result.push_back(
            text.substr(begin, end - begin));
        begin = end + 1;
    }

    return result;
}
```

### 逐步理解

對：

```text
a,,b,
```

結果為：

```text
["a", "", "b", ""]
```

流程：

1. 第一個逗號前是 `"a"`。
2. 兩個逗號中間是空範圍，因此得到 `""`。
3. 下一段是 `"b"`。
4. 最後一個逗號後仍有一個空範圍，因此得到 `""`。

### 為什麼最後還要 `push_back`

Token 不只在遇到 Delimiter 時形成。最後一段內容位於最後一個 Delimiter 之後，或整個字串根本沒有 Delimiter。

因此，`find` 回傳 `npos` 時仍要將尾端範圍加入結果。

### `vector<string_view>` 的生命週期

所有 Token 都指向原輸入資料：

```cpp
std::string input = "a,b,c";
auto parts = split(input, ',');
```

使用 `parts` 期間，`input` 必須仍然存在，而且不能發生使底層位置失效的修改。

若 Token 需要獨立保存，應考慮回傳：

```cpp
std::vector<std::string>
```

### 數字解析還要定義什麼

解析整數時還要確認：

- 是否允許前導空白？
- 是否允許 `+` 或 `-`？
- 是否至少需要一個數字？
- 是否允許前導零？
- 遇到非法 Byte 時停止，還是整體失敗？
- 超出目標型別範圍時如何回報？

Overflow 應在乘以 10 與加入下一位數字之前檢查，而不是結果已經溢位後再判斷。


## 6.14 補充：C String

### C String 如何表示結尾

C String 通常以 `char` Array 儲存，並使用 Null Terminator `\0` 表示結尾。

```c
char text[] = "abc";
```

記憶體內容概念上是：

```text
'a' 'b' 'c' '\0'
```

Array 容量是 4，文字長度是 3。

### Capacity 與 Length

必須分清：

- Buffer Capacity：總共配置多少 Bytes。
- String Length：第一個 `\0` 前有多少 Bytes。
- 是否保留一格寫入最後的 `\0`。

若 Capacity 是 `capacity`，最多只能保存 `capacity - 1` 個非 Null Bytes。

### `strlen` 的前置條件與成本

`strlen(text)` 從起點一路尋找第一個 `\0`，時間為 O(n)。

前置條件是後方確實存在可達的 Null Terminator。若沒有，函式可能讀出有效範圍。

避免在不變的字串上重複呼叫：

```c
size_t length = strlen(text);

for (size_t i = 0; i < length; ++i)
{
    process(text[i]);
}
```

### 明確傳入 Pointer 與 Length

若資料可以包含 Null Byte，或本來就不是 C String，應明確傳入資料位置與長度：

```c
void process_bytes(
    const unsigned char data[],
    size_t length);
```

此時不能使用 `strlen` 推測資料大小。

### 安全追加一個字元

```c
#include <stdbool.h>
#include <stddef.h>

bool append_char(
    char buffer[],
    size_t capacity,
    size_t *length,
    char ch)
{
    if (buffer == NULL || length == NULL)
    {
        return false;
    }

    if (*length >= capacity ||
        capacity - *length <= 1)
    {
        return false;
    }

    buffer[*length] = ch;
    ++(*length);
    buffer[*length] = '\0';

    return true;
}
```

成功後同時維持：

- `*length < capacity`。
- `buffer[*length] == '\0'`。
- `[0, *length)` 是有效文字內容。

### C 與 UTF-8

使用 `char*` 不代表每個 `char` 是一個 Unicode 字元。UTF-8 在 C 中仍是多 Byte 編碼，逐 `char` 反轉或截斷仍可能破壞文字。


## 6.15 字串題的固定分析流程

遇到新題目時，可以依序問以下問題。

### 第一步：資料單位是什麼

- ASCII？
- 任意 Byte？
- UTF-8 Code Point？
- Grapheme Cluster？

若題目保證只含小寫英文字母，就不需要一開始處理完整 Unicode。

### 第二步：比較規則是什麼

- 是否區分大小寫？
- 是否忽略空白？
- 是否忽略標點？
- 數字是否參與比較？
- 空字串如何定義？

### 第三步：候選結構是什麼

- 連續區間：Substring。
- 保持順序、可跳過：Subsequence。
- 不重視順序：可能是 Set 或 Multiset 類型問題。

### 第四步：需要保存什麼狀態

- 回文：左右 Index。
- Subsequence：下一個待配對位置。
- 頻率統計：Array 或 Map。
- Sliding Window：左右邊界與 Window 內狀態。

### 第五步：區間如何表示

優先考慮：

```text
[begin, end)
```

並確認 API 接受的是右邊界，還是數量。

### 第六步：是否正在複製資料

檢查：

- 迴圈中是否反覆呼叫 `std::string::substr`？
- 遞迴參數是否按值傳遞整個字串？
- 是否反覆使用 `result = result + piece`？
- 是否可以改傳 Index、區間或 View？

### 第七步：Ownership 與生命週期

若使用 `string_view`：

- 原字串是否活得夠久？
- 原字串是否會修改？
- 是否可能 Reallocation？
- 結果是否需要獨立保存？

### 第八步：建立邊界測試

至少測試：

- 空字串。
- 單一元素。
- 全部相同。
- 完全不匹配。
- 重複字元。
- 開頭與結尾邊界。
- 非 ASCII 輸入是否符合目前 Precondition。
- Delimiter 在開頭、結尾與連續出現。


## 6.16 常見問題與判讀

### `size()` 比看到的字數大

可能原因：輸入是 UTF-8，而 `size()` 計算 Bytes。

先確認需求要計算：

- Byte。
- Code Point。
- Grapheme Cluster。

### 反轉後出現亂碼

可能原因：逐 Byte 反轉多 Byte UTF-8 序列。

先確認是否需要 UTF-8 解碼，甚至需要依 Grapheme Cluster 處理。

### `substr` 取出的範圍太長

可能原因：把右邊界當成第二個參數。

若目標是 `[left, right)`：

```cpp
text.substr(left, right - left)
```

### Frequency Array 越界

可能原因：輸入不符合值域假設。

若使用 26 格 Array，先確認每個輸入都在 `'a'` 到 `'z'`。

### `isdigit` 或 `tolower` 在部分輸入異常

可能原因：直接傳入負值 `char`。

先轉成：

```cpp
static_cast<unsigned char>(ch)
```

### 遞迴輸入不大，但執行很慢

可能原因：每層建立新的 Substring，或按值複製整個字串。

考慮改傳：

- `const std::string&` 加 Index。
- `[left, right)`。
- 生命週期安全的 `string_view`。

### `string_view` 內容突然改變或失效

檢查：

- 原字串是否已被銷毀？
- 是否來自暫時物件？
- 原字串是否修改或 Reallocation？

### Split 漏掉最後一個 Token

可能原因：只在遇到 Delimiter 時加入 Token，沒有在 `npos` 時處理尾端範圍。

### 連續 Delimiter 的結果不符合預期

可能原因：空 Token 規格沒有先定義。

先決定：

- 保留空 Token。
- 忽略空 Token。
- 空輸入代表零個還是一個 Token。

### 回文答案與預期不同

先不要急著改迴圈，先確認：

- 是否區分大小寫？
- 是否忽略空格與標點？
- 是逐 Byte 比較，還是更高層的 Unicode 文字單位？


## 6.17 本章檢查表

### 基本模型

- 我知道 `std::string` 可以依 Index 存取 `char`。
- 我知道合法 Index 是 `[0, size())`。
- 我知道空字串不能存取 `text[0]`。
- 我知道 `size()` 回傳 `std::size_t`。

### 編碼

- 我知道 `std::string::size()` 回傳 Byte 數量。
- 我不會將 Byte 數直接稱為 Unicode 字元數。
- 我能區分 Byte、Code Point 與 Grapheme Cluster。
- 我知道逐 Byte 反轉可能破壞 UTF-8。

### 區間與問題模型

- 我能使用 `[begin, end)` 表示區間。
- 我知道 `substr(position, count)` 的第二個參數是數量。
- 我能區分 Substring、Subsequence 與 Subset。
- 我知道 `substr` 是切片工具，不是搜尋方法。

### 回文與分類

- 我會先定義大小寫、空白、標點與數字規則。
- 我能使用左右指標判斷回文。
- 我知道如何避免空字串上的 Unsigned Underflow。
- 我會在呼叫 `<cctype>` 函式前轉成 `unsigned char`。
- 我知道 ASCII 解法不等於完整 Unicode 解法。

### 頻率

- 我知道固定 Frequency Array 需要明確值域。
- 我能依需求選擇 26 格 Array、256 格 Array 或 Map。
- 我知道 Anagram 可以使用頻率差值判斷。

### 複製與生命週期

- 我知道 `std::string::substr` 通常建立副本。
- 我會檢查迴圈與遞迴中的總複製成本。
- 我能使用 Index、區間或 `string_view` 表示子問題。
- 我知道 `string_view` 不擁有資料。
- 我不會回傳指向區域 `std::string` 的 View。
- 我知道 Reallocation 可能讓 View 失效。
- 我知道 View 結尾不保證有 Null Terminator。

### 建構、解析與 C String

- 我能區分 `reserve`、`resize` 與實際 Size。
- 我會避免反覆從字串前端插入。
- 我會先定義 Split 的空 Token 規格。
- 我知道 `vector<string_view>` 依賴原輸入生命週期。
- 我知道 C String 必須有 Null Terminator。
- 我知道 Buffer Capacity 必須保留結尾空間。
- 我知道 `strlen` 每次都可能重新走訪字串。


## 6.18 本章重點

1. 先把 `std::string` 理解成依序存放 `char` 的容器，再逐步加入編碼觀念。
2. 對 ASCII 題目，逐 `char` 處理通常合理；對 UTF-8，Byte 不一定是完整文字單位。
3. 字串演算法開始前，先定義比較規則與資料單位。
4. Substring 必須連續；Subsequence 可以跳過元素，但順序不能改變。
5. 判斷 Subsequence 時，只需要記錄下一個待配對位置。
6. 回文可以使用左右指標，Half-open Interval 能安全表示空範圍。
7. 忽略標點與大小寫可以先建立整理後字串，也可以在比較時直接套用規則。
8. Frequency Array 的正確性依賴明確的字元值域。
9. `std::string::substr` 負責依已知位置與長度建立新字串，不負責搜尋答案。
10. 反覆建立 Substring 可能累積大量複製成本。
11. `string_view` 不擁有資料，使用它時必須追蹤原資料生命週期。
12. `reserve` 只保留 Capacity，不會建立可用元素。
13. 字串解析必須先定義空 Token、Delimiter 與錯誤處理規則。
14. C String 以 `\0` 表示結尾，必須同時管理 Capacity、Length 與 Null Terminator。
15. 看不懂某個進階細節時，可以先回到三個問題：處理單位是什麼、候選是否連續、資料由誰擁有。
