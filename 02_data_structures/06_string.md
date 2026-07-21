## 第 6 章　String 與字元處理

### 適用範圍

本章介紹字串的資料模型，以及字串題中經常混在一起的 Byte、字元、編碼、區間與生命週期問題。

`std::string` 的介面看起來和 Array 相似，可以取得長度、以 Index 存取、走訪與建立子字串。但字串多了一層編碼語意。若輸入只包含 ASCII，一個可見字元通常對應一個 Byte；若使用 UTF-8，一個可見字元可能由多個 Bytes 組成。因此，在開始寫迴圈前，需要先確認演算法處理的是：

- Byte。
- ASCII 字元。
- Unicode Code Point。
- 使用者看到的完整字形。

本章也會說明：

- Substring、Subsequence 與 Subset 的差異。
- 如何使用 Two Pointers 判斷回文。
- 如何根據字元範圍選擇 Frequency Array 或 Hash Map。
- `std::isdigit`、`std::tolower` 等函式的安全呼叫方式。
- `substr` 的複製成本。
- `std::string_view` 的非擁有特性與生命週期風險。
- 如何避免反覆串接造成不必要的資料搬移。
- 如何使用 Index、Delimiter 與 Token 規格進行字串解析。

本章會建立一套固定流程：

1. 先確認輸入編碼與比較單位。
2. 明確定義空字串、大小寫與忽略字元規則。
3. 區分連續 Substring 與可跳過元素的 Subsequence。
4. 使用 Half-open Interval 表示字串區間。
5. 根據字元範圍選擇頻率資料結構。
6. 建立新字串前，評估複製與串接成本。
7. 使用 `string_view` 前，確認原始資料的 Ownership 與生命週期。
8. 從空輸入、單一 Byte、多 Byte 文字與 Delimiter 邊界建立測試。

### 適用讀者

- 需要處理字元走訪、回文、頻率與連續字串問題的讀者。
- 容易把 Byte 數量當成 Unicode 字元數量的讀者。
- 常混淆 Substring、Subsequence 與 Subset 的讀者。
- 在使用 `std::isdigit` 或 `std::tolower` 時遇到不穩定結果的讀者。
- 遞迴或迴圈中大量呼叫 `substr`，但不清楚複製成本的讀者。
- 使用 `std::string_view` 後遇到內容異常或懸空 View 的讀者。
- 需要建立 Token、Delimiter 與空欄位解析規格的讀者。
- 同時使用 C++ 與 C，希望理解 `std::string` 和 C String 差異的讀者。

### 快速導覽

- [String 到底保存什麼](#61-string-到底保存什麼)：區分 Byte、編碼與可見字形。
- [第一步：先定義字串規格](#62-第一步先定義字串規格)：確認編碼、大小寫與正規化規則。
- [第二步：安全走訪 Byte 與字元分類](#63-第二步安全走訪-byte-與字元分類)：正確使用 `<cctype>`。
- [第三步：區分 Substring 與 Subsequence](#64-第三步區分-substring-與-subsequence)：建立連續區間與選取模型。
- [完整案例：判斷 Subsequence](#65-完整案例判斷-subsequence)：使用雙 Index 與 Invariant。
- [第四步：使用 Two Pointers 判斷回文](#66-第四步使用-two-pointers-判斷回文)：處理比較規則與邊界。
- [完整案例：忽略非英數字元的回文](#67-完整案例忽略非英數字元的回文)：整合 Normalize 與 Two Pointers。
- [第五步：進行頻率統計](#68-第五步進行頻率統計)：依值域選擇 Array 或 Hash Map。
- [第六步：理解 Substring 的複製成本](#69-第六步理解-substring-的複製成本)：比較 `substr`、Index 與 View。
- [第七步：管理 string_view 的生命週期](#610-第七步管理-string_view-的生命週期)：區分 View 與 Ownership。
- [第八步：有效率地建構字串](#611-第八步有效率地建構字串)：使用 `reserve`、`push_back` 與尾端追加。
- [第九步：建立清楚的解析規格](#612-第九步建立清楚的解析規格)：處理 Delimiter、空 Token 與尾端資料。
- [C 語言中的字串](#613-c-語言中的字串)：補充 Null Terminator、長度與緩衝區容量。
- [建立自己的字串分析表](#614-建立自己的字串分析表)：形成固定檢查流程。
- [常見問題與判讀](#615-常見問題與判讀)：整理常見錯誤與檢查方向。
- [本章檢查表](#616-本章檢查表)：確認必要觀念是否完整。
- [本章重點](#617-本章重點)：回顧核心方法。

### 6.1 String 到底保存什麼

`std::string` 可以視為一段連續的 `char` 序列。它保存的是 Bytes，而不是抽象的「人類可見字元」集合。

```cpp
std::string text = "abc";
```

若內容只使用 ASCII，可以直觀地畫成：

```text
Index：0   1   2
Byte： 'a' 'b' 'c'
```

此時 `text.size()` 為 3，每個 Index 也剛好對應一個英文字母。

#### UTF-8 需要分開理解

UTF-8 使用可變長度編碼。一個 Unicode Code Point 可能使用一到四個 Bytes。因此：

```cpp
text.size()
```

回傳的是 Byte 數量，不保證等於：

- Unicode Code Point 數量。
- 使用者看到的字形數量。
- 游標移動一次所跨越的文字單位。

例如，某些帶有組合符號的文字可能由多個 Code Points 組成一個可見字形。Emoji 也可能由多個 Code Points 組成。若題目要求「反轉使用者看到的字元」，直接交換 `std::string` 的 Bytes 可能破壞 UTF-8 序列。

#### 先確認處理層級

<table>
<tr><th>需求</th><th>可能的處理單位</th><th>注意事項</th></tr>
<tr><td>只含小寫英文字母的演算法題</td><td>ASCII Byte</td><td>可使用 `ch - 'a'`，但要有明確 Precondition</td></tr>
<tr><td>檢查通訊資料或檔案前綴</td><td>Byte</td><td>不一定需要解讀成自然語言</td></tr>
<tr><td>計算 Unicode Code Point</td><td>解碼後的 Code Point</td><td>不能只使用 `size()`</td></tr>
<tr><td>依使用者可見字形截斷</td><td>Grapheme Cluster</td><td>通常需要 Unicode 函式庫</td></tr>
</table>

演算法題若明確保證只含英文字母或 ASCII，可直接按 Byte 處理。若規格只寫「字串」而沒有說明編碼，應先把這項假設列出。

#### 內含 Null Byte

`std::string` 可以保存 `\0` Byte，因為它另外記錄長度：

```cpp
std::string data{"a\0b", 3};
```

此時 `data.size() == 3`。但若把 `data.c_str()` 傳給只依 Null Terminator 判斷長度的 C 介面，對方可能只看到第一個 `a`。跨 C 與 C++ 介面時，需要確認資料是否允許內含 Null Byte。

### 6.2 第一步：先定義字串規格

字串題常因為比較規則不明而產生不同答案。開始寫程式前，至少確認以下內容：

- 輸入使用哪種編碼。
- 是否只含 ASCII。
- 是否區分英文大小寫。
- 是否忽略空白。
- 是否忽略標點符號。
- 是否保留數字。
- 空字串的答案是什麼。
- 輸出要求 Byte Index 還是字元位置。
- 是否允許修改原字串。

#### 同一個輸入可能有不同答案

輸入：

```text
A man, a plan, a canal: Panama
```

若逐 Byte、區分大小寫並保留標點，它不是回文。

若規格要求：

- 忽略非英數字元。
- 英文字母不區分大小寫。

則它可被判定為回文。

這不是哪一個演算法比較正確，而是 Postcondition 不同。Normalize 規則必須先定義。

#### Normalize 不一定要建立新字串

一種作法是先建立正規化結果：

```text
amanaplanacanalpanama
```

再判斷回文。另一種作法是在 Two Pointers 移動時略過不需要比較的 Bytes，並在比較當下轉成相同大小寫。後者可避免建立完整副本，但控制流程較複雜。

選擇哪一種方式，應考慮：

- 可讀性。
- 是否允許額外 O(n) 空間。
- Normalize 結果是否會重複使用。
- 輸入是否限定為 ASCII。

#### 區間仍建議使用 Half-open Interval

字串區間可寫成 `[begin, end)`：

```text
長度 = end - begin
```

`std::string::substr(position, count)` 的第二個參數是數量，不是右邊界。若要複製 `[left, right)`：

```cpp
std::string part = text.substr(left, right - left);
```

常見錯誤是把 `right` 直接當成 `count`，導致範圍過長。

### 6.3 第二步：安全走訪 Byte 與字元分類

若規格限定 ASCII，可像 Array 一樣走訪：

```cpp
for (char ch : text)
{
    process(ch);
}
```

若需要 Index：

```cpp
for (std::size_t i = 0; i < text.size(); ++i)
{
    process(text[i]);
}
```

#### `<cctype>` 的參數限制

`std::isdigit`、`std::isalpha`、`std::isalnum`、`std::tolower` 等函式，接受的值必須可表示為 `unsigned char`，或等於 `EOF`。若直接傳入負值的 `char`，行為可能未定義。

安全包裝方式：

```cpp
#include <cctype>

bool isDigit(char ch)
{
    const auto value = static_cast<unsigned char>(ch);
    return std::isdigit(value) != 0;
}
```

轉成小寫：

```cpp
char toLowerAscii(char ch)
{
    const auto value = static_cast<unsigned char>(ch);
    return static_cast<char>(std::tolower(value));
}
```

這些函式也可能受目前 Locale 影響。若題目只處理 ASCII，可以直接寫出 ASCII 範圍條件，讓規格更明確：

```cpp
bool isAsciiDigit(char ch)
{
    return ch >= '0' && ch <= '9';
}
```

#### `char` 不保證是 Signed 或 Unsigned

`char` 是否視為 Signed 由執行環境決定。這也是呼叫 `<cctype>` 函式前先轉成 `unsigned char` 的原因之一。

#### Byte Index 不一定是字元 Index

對 UTF-8 文字，`text[i]` 可能只取到某個多 Byte 編碼的一部分。若演算法以 `i + 1` 表示「下一個 Unicode 字元」，邏輯就不成立。此時需要使用 UTF-8 解碼器或適當的 Unicode 函式庫，而不是逐 Byte 套用 ASCII 分類函式。

### 6.4 第三步：區分 Substring 與 Subsequence

#### Substring

Substring 是原字串中的連續區間。

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

若使用 `[left, right)` 表示，Substring 就是原字串的一段連續 Index 範圍。

#### Subsequence

Subsequence 保留原本順序，但可以跳過元素。

```text
ace
```

是 `abcde` 的 Subsequence，因為可以依序選取 Index 0、2、4。

但 `ace` 不是 Substring，因為它不連續。

#### Subset

Subset 通常不保留原本順序概念，只關心選了哪些元素。字串題大多使用 Substring 或 Subsequence，不應只看到「選一些字元」就忽略順序。

<table>
<tr><th>結構</th><th>要求連續</th><th>要求保留順序</th><th>常見表示</th></tr>
<tr><td>Substring</td><td>是</td><td>是</td><td>區間 `[left, right)`</td></tr>
<tr><td>Subsequence</td><td>否</td><td>是</td><td>依序選取 Index</td></tr>
<tr><td>Subset</td><td>否</td><td>通常不強調</td><td>每個元素選或不選</td></tr>
</table>

#### 問題模型會影響演算法

- 最長不重複 Substring 常使用 Sliding Window。
- 判斷某字串是否為 Subsequence 常使用 Two Pointers。
- 列舉所有 Subsequence 常使用遞迴或 Backtracking。

三者名稱相似，但候選空間不同。

### 6.5 完整案例：判斷 Subsequence

#### 問題規格

給定 `candidate` 與 `text`，判斷 `candidate` 是否為 `text` 的 Subsequence。

```text
candidate = "ace"
text = "abcde"
答案 = true
```

#### Precondition

本案例按 `std::string` 的 Byte 比較，適用於題目已保證 ASCII，或需求本來就是比較 Byte 序列。

#### Postcondition

回傳 `true`，若且唯若存在嚴格遞增的 Index：

```text
i0 < i1 < ... < ik
```

使 `candidate[j] == text[ij]`。

#### C++ 解法

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

#### State 意義

`matched` 表示：

> 已經依序配對完成 `candidate[0, matched)`。

#### Loop Invariant

每輪開始前：

1. `candidate[0, matched)` 是已走訪 `text` Prefix 的 Subsequence。
2. `matched` 是使用目前已走訪 Prefix 能依序配對出的最長 Candidate Prefix 長度。
3. `0 <= matched <= candidate.size()`。

#### Initialization

開始時 `matched == 0`。空字串是任何字串的 Subsequence，因此 `candidate[0, 0)` 已成功配對，Invariant 成立。

#### Maintenance

讀取目前 `text` Byte：

- 若它等於下一個待配對 Byte，增加 `matched`。
- 若不同，略過目前 Byte，不改變已配對結果。

因為 Subsequence 允許跳過元素，而配對時只向前移動，順序不會被破壞。

#### Termination

完整走訪 `text` 後：

- 若 `matched == candidate.size()`，全部 Candidate Bytes 都已依序配對，回傳 `true`。
- 否則仍有 Candidate Byte 未找到，回傳 `false`。

#### 為什麼貪心配對最早位置可行

當下一個 Candidate Byte 可在目前位置配對時，選擇最早可用位置，不會減少後方可使用的空間。若改選更晚的相同 Byte，只會留下更短的後綴。因此，立即配對目前最早位置是安全的。

#### 邊界案例

<table>
<tr><th>candidate</th><th>text</th><th>答案</th><th>目的</th></tr>
<tr><td>空字串</td><td>任何字串</td><td>true</td><td>空序列是任何序列的 Subsequence</td></tr>
<tr><td>"a"</td><td>空字串</td><td>false</td><td>沒有可配對位置</td></tr>
<tr><td>"abc"</td><td>"abc"</td><td>true</td><td>完全相同</td></tr>
<tr><td>"ace"</td><td>"abcde"</td><td>true</td><td>跳過中間 Byte</td></tr>
<tr><td>"aec"</td><td>"abcde"</td><td>false</td><td>順序不符</td></tr>
<tr><td>"aaa"</td><td>"aa"</td><td>false</td><td>重複次數不足</td></tr>
</table>

#### 複雜度

- 時間複雜度：O(text.size())。
- 額外空間複雜度：O(1)。

使用 `string_view` 只是避免複製參數，不會改變核心演算法。

### 6.6 第四步：使用 Two Pointers 判斷回文

回文表示由左向右與由右向左讀取相同。

最直接的方法是將字串反轉後比較，但這需要建立副本。Two Pointers 可以從左右兩端向中間移動：

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

這裡將 `right` 設為 Exclusive End，避免空字串時直接計算 `size() - 1` 造成 Unsigned Underflow。

#### Invariant

每輪開始前：

> 區間 `[0, left)` 與 `[right, size)` 已完成鏡像配對，而且每一組都相同。

本輪先將 `right` 移到下一個尚未比較的位置，再比較 `text[left]` 與 `text[right]`。

#### 終止條件

當左右未比較區間長度小於 2 時，不再需要比較。奇數長度字串中央 Byte 不影響回文結果。

#### 本版本的語意

本版本：

- 逐 Byte 比較。
- 區分大小寫。
- 不忽略空白或標點。

因此它適合 ASCII 或 Byte 序列回文。若需求是 Unicode 可見字形回文，需要先使用適當方式切分文字單位。

### 6.7 完整案例：忽略非英數字元的回文

#### 問題規格

判斷 ASCII 字串在忽略非英數字元，且英文字母不區分大小寫後，是否為回文。

#### Precondition

- 輸入按 ASCII 規則分類。
- 非 ASCII Byte 不視為本案例定義中的英數字元。

#### 輔助函式

```cpp
bool isAsciiAlphaNumeric(char ch)
{
    const bool digit = ch >= '0' && ch <= '9';
    const bool lower = ch >= 'a' && ch <= 'z';
    const bool upper = ch >= 'A' && ch <= 'Z';
    return digit || lower || upper;
}

char lowerAscii(char ch)
{
    if (ch >= 'A' && ch <= 'Z')
    {
        return static_cast<char>(ch - 'A' + 'a');
    }
    return ch;
}
```

#### C++ 解法

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

#### 區間模型

尚未處理範圍是：

```text
[left, right)
```

左側檢查 `text[left]`，右側檢查 `text[right - 1]`。使用 Exclusive End 可以安全表示空字串。

#### Loop Invariant

每輪主要比較開始前：

1. `[0, left)` 與 `[right, size)` 中所有需要比較的英數字元，已完成鏡像配對。
2. 已配對內容在忽略大小寫後相同。
3. `[left, right)` 是尚未處理範圍。

略過非英數 Byte 不會改變 Normalize 後的字串，因此 Invariant 仍成立。

#### 邊界案例

<table>
<tr><th>輸入</th><th>答案</th><th>目的</th></tr>
<tr><td>空字串</td><td>true</td><td>Normalize 後仍為空</td></tr>
<tr><td>"a"</td><td>true</td><td>單一英文字母</td></tr>
<tr><td>".,,"</td><td>true</td><td>Normalize 後為空</td></tr>
<tr><td>"A1a"</td><td>true</td><td>大小寫與數字</td></tr>
<tr><td>"ab"</td><td>false</td><td>第一組比較即失敗</td></tr>
<tr><td>"A man, a plan, a canal: Panama"</td><td>true</td><td>空白、標點與大小寫</td></tr>
</table>

#### 不應直接套用到任意 Unicode 文字

此案例的分類與小寫轉換只定義 ASCII。若產品需求涉及完整 Unicode 大小寫折疊、正規化或 Grapheme Cluster，比較規則會更複雜，應使用適合的 Unicode 函式庫並明確指定 Normalization Form。

### 6.8 第五步：進行頻率統計

頻率統計的核心問題是：

> 每個可能鍵值的範圍是否小而固定？

#### 只含小寫英文字母

若 Precondition 保證每個 Byte 都在 `'a'` 到 `'z'`：

```cpp
#include <array>
#include <string_view>

std::array<int, 26> countLowercase(
    std::string_view text)
{
    std::array<int, 26> frequency{};

    for (char ch : text)
    {
        ++frequency[static_cast<std::size_t>(ch - 'a')];
    }

    return frequency;
}
```

映射關係是：

```text
'a' -> 0
'b' -> 1
...
'z' -> 25
```

#### Precondition 不成立的風險

若 `ch` 是 `'A'`、數字或其他 Byte：

```cpp
ch - 'a'
```

可能不在 `[0, 26)`，進而造成越界。因此，固定 Frequency Array 的正確性依賴明確字元範圍。

#### 範圍不固定時使用 Map

若需統計任意 Byte：

```cpp
#include <array>

std::array<std::size_t, 256> frequency{};
for (unsigned char byte : text)
{
    ++frequency[byte];
}
```

若要統計解碼後的 Unicode Code Point，則可使用以 Code Point 為 Key 的 Map，但前提是先正確解碼 UTF-8。

#### Array 與 Hash Map 的選擇

<table>
<tr><th>已知條件</th><th>適合結構</th><th>原因</th></tr>
<tr><td>只含 26 個小寫英文字母</td><td>固定 Array</td><td>範圍小、映射直接</td></tr>
<tr><td>任意 Byte</td><td>大小 256 的 Array</td><td>Byte 值域固定</td></tr>
<tr><td>稀疏且範圍大的鍵值</td><td>Hash Map</td><td>不需配置完整值域</td></tr>
<tr><td>需要依鍵排序輸出</td><td>Ordered Map 或排序後輸出</td><td>Hash Map 不保證排序順序</td></tr>
</table>

#### 計數型別

若字串長度可能大於 `int` 可表示範圍，頻率可使用 `std::size_t`。選擇型別時應以最大輸入長度為基礎，而不是固定使用 `int`。

#### 完整案例：檢查 Anagram

若兩個 ASCII 小寫字串互為 Anagram，它們的每個字母出現次數相同：

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
        ++difference[first[i] - 'a'];
        --difference[second[i] - 'a'];
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

Precondition 是兩個輸入都只包含 `'a'` 到 `'z'`。Postcondition 是：回傳 `true`，若且唯若每個小寫字母在兩個輸入中的頻率相同。

### 6.9 第六步：理解 Substring 的複製成本

`std::string::substr` 通常建立一個新的 `std::string`，並複製指定內容：

```cpp
std::string part = text.substr(position, count);
```

若複製長度為 `k`，時間與額外空間通常至少和 `k` 成正比。

#### 遞迴中反覆建立 Substring

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

假設長度為 `n`，每層分別複製約 `n - 1`、`n - 2`、直到 1 個 Byte。總複製量可能形成：

```text
(n - 1) + (n - 2) + ... + 1 = O(n²)
```

另外，函式參數按值接收也會產生複製。

#### 改傳 Index

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

原字串由 `const` Reference 共用，每層只傳遞 Index。

#### 改傳 Half-open Interval

若遞迴處理一段範圍，可傳：

```text
[left, right)
```

這樣能避免每層建立新字串，也能清楚表示空範圍。

#### 使用 `string_view`

```cpp
bool solve(std::string_view text)
{
    if (text.empty())
    {
        return true;
    }

    return solve(text.substr(1));
}
```

`string_view::substr` 建立新 View，不複製底層字元。但 View 不擁有資料，原始字串必須在所有遞迴呼叫期間保持有效。

#### 6.9.1 名詞對照與實戰：從定義到程式碼

看到 `substr` 時，可以先把問題拆成三層：

1. 定義層：題目要的是連續的 Substring，還是可跳過元素的 Subsequence。
2. 工具層：`std::string::substr` 只負責從已知位置切出一段連續內容。
3. 搜尋層：若還不知道答案在哪裡，需要另外用 `find`、雙迴圈、Sliding Window、DP 或其他字串搜尋方法。

`substr` 的角色比較像「切片工具」，不是「搜尋工具」。例如：

```cpp
std::string text = "abcde";
std::string part = text.substr(1, 3); // "bcd"
```

這裡的第二個參數是 Count，也就是要取幾個 Bytes。若用 Half-open Interval 表示同一段範圍，`"bcd"` 是 `[1, 4)`，因此應寫成：

```cpp
std::string part = text.substr(1, 4 - 1);
```

若寫成 `text.substr(1, 4)`，意思會變成「從 Index 1 開始取 4 個 Bytes」，結果是 `"bcde"`。

##### 找共同子串不是改寫 substr

若題目給兩個字串，要求找出共同的連續片段，真正要處理的是「搜尋兩個字串中相同的連續區間」。最後可以用 `substr` 把答案切出來，但答案的位置與長度要先由演算法決定。

例如：

```cpp
std::string s1 = "abcabc";
std::string s2 = "abcabcabcabc";
```

若只是判斷固定 Pattern `"abc"` 是否同時出現在兩個字串中，可以使用 `find`：

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string s1 = "abcabc";
    std::string s2 = "abcabcabcabc";
    std::string pattern = "abc";

    if (s1.find(pattern) != std::string::npos &&
        s2.find(pattern) != std::string::npos)
    {
        std::cout << pattern << " is a common substring
";
    }
}
```

若要自動找出最長共同 Substring，可以先用雙迴圈建立概念：

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
                // bestBegin 記錄 first 中該段連續區間的起始位置。
                // 因為本次配對都以目前的 i 為基準。
                bestBegin = i;
                bestLength = length;
            }
        }
    }

    return first.substr(bestBegin, bestLength);
}
```

這段流程中，雙迴圈負責選擇兩個起點，`while` 負責同步往右比對。一旦遇到不同 Byte，本次連續片段就結束。`substr` 只在最後使用，用來切出已經找到的區間。

##### 最長共同子串的 DP 狀態

若輸入較長，暴力比對最壞情況可能需要 `O(n * m * min(n, m))` 時間。DP 可將時間降為 `O(n * m)`。

定義：

```text
dp[i][j] = first[0, i) 與 second[0, j) 中，
           以 first[i - 1] 和 second[j - 1] 結尾的最長共同後綴長度
```

轉移規則：

- 若 `first[i - 1] == second[j - 1]`，則 `dp[i][j] = dp[i - 1][j - 1] + 1`。
- 若不同，則 `dp[i][j] = 0`。因為 Substring 必須連續，中間斷開後不能沿用前面的長度。

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

    return first.substr(bestEnd - bestLength, bestLength);
}
```

這個 DP 狀態只從左上角轉移。左上角代表兩邊前一個位置也必須配對成功，因此它自然保留了 Substring 的連續性。

<table>
<tr><th>題目用語</th><th>通常代表</th><th>常見方法</th></tr>
<tr><td>連續片段、Substring、子字串</td><td>`[left, right)` 連續區間</td><td>Sliding Window、雙迴圈、DP</td></tr>
<tr><td>保持順序、可刪除部分字元、Subsequence</td><td>可跳過元素，但順序不能改</td><td>Two Pointers、DP</td></tr>
<tr><td>某個 Pattern 是否出現</td><td>固定字串搜尋</td><td>`find`、KMP、Rolling Hash</td></tr>
<tr><td>最長共同子串</td><td>兩字串中的最長連續共同片段</td><td>DP、Suffix Array、Suffix Automaton</td></tr>
</table>

### 6.10 第七步：管理 string_view 的生命週期

`std::string_view` 保存的是：

- 指向某段字元資料的位置。
- 該範圍的長度。

它不擁有資料，也不負責延長原字串生命週期。

#### 安全案例

```cpp
std::string_view middle(std::string_view text)
{
    if (text.size() < 2)
    {
        return {};
    }

    return text.substr(1, text.size() - 2);
}
```

函式回傳的 View 仍指向呼叫端提供的資料。它是否安全，取決於呼叫端原始資料是否繼續存在且未被不相容地修改。

#### 懸空 View

```cpp
std::string_view makeView()
{
    std::string local = "temporary";
    return local;
}
```

函式結束時 `local` 被銷毀，回傳的 View 指向已失效資料。

類似風險也可能發生於暫時物件：

```cpp
std::string_view view = std::string("temporary");
```

完整運算式結束後，暫時字串被銷毀，`view` 失效。

#### 原字串修改也可能使 View 失效

```cpp
std::string text = "abc";
std::string_view view = text;

text.push_back('d');
```

若 `push_back` 造成 Reallocation，View 原本指向的位置便失效。即使沒有 Reallocation，刪除或重排內容也可能讓 View 的語意不再符合原本預期。

#### View 不保證 Null-terminated

`string_view` 可指向原字串中間：

```cpp
std::string_view part(text.data() + 2, 3);
```

`part.data()` 後方不一定在 View 結尾處有 `\0`。因此不能只把 `part.data()` 傳給預期 C String 的函式。若對方需要 Null-terminated 字串，應建立擁有資料的 `std::string`：

```cpp
std::string owned(part);
useCFunction(owned.c_str());
```

#### 是否適合使用 `string_view`

適合：

- 函式只在呼叫期間讀取字串。
- 切分大量子區間但不需保存副本。
- 原始資料生命週期清楚且足夠長。

不適合直接保存：

- 原始資料可能很快被銷毀。
- 原字串會修改或重新配置。
- 需要獨立 Ownership。
- 需要傳給依賴 Null Terminator 的介面。

### 6.11 第八步：有效率地建構字串

#### 尾端追加

```cpp
std::string result;
result.push_back('a');
result += "bc";
```

和 `std::vector` 相似，字串尾端追加在容量足夠時通常不需搬移全部內容；容量不足時可能重新配置。

#### 已知長度時先 `reserve`

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

`reserve` 不會改變 `size()`，也不會建立可用字元。它只是降低成長過程中反覆配置的可能性。

#### 反覆在前端插入

```cpp
result.insert(result.begin(), ch);
```

每次前端插入都可能搬移現有全部內容。若執行 `n` 次，總搬移量可能形成 O(n²)。

如果目標是反向結果，可考慮：

- 先尾端追加，再 `std::reverse`。
- 從原輸入後方向前走訪並尾端追加。
- 預先建立固定大小結果，再按 Index 填入。

#### 反覆使用 `result = result + piece`

此寫法可能建立新的暫時字串並複製舊內容：

```cpp
result = result + piece;
```

若只是追加，通常較適合：

```cpp
result += piece;
```

仍需根據實際流程與編譯器最佳化判斷，但語意上 `+=` 更直接表達在原結果尾端加入內容。

#### 預先建立大小

若最終長度可以準確計算：

```cpp
std::string result(length, '\0');

for (std::size_t i = 0; i < length; ++i)
{
    result[i] = computeCharacter(i);
}
```

這裡使用的是建立 `length` 個元素，不是只保留 Capacity。因此可以合法按 Index 寫入 `[0, length)`。

### 6.12 第九步：建立清楚的解析規格

字串解析不能只寫「用逗號切開」。還要定義：

- 連續 Delimiter 是否產生空 Token。
- 開頭 Delimiter 是否產生第一個空 Token。
- 結尾 Delimiter 是否產生最後一個空 Token。
- Token 前後空白是否保留。
- 空輸入是零個 Token，還是一個空 Token。
- 是否允許 Escape 或引號。
- 數字前是否允許正負號。
- 遇到非法內容時回傳什麼。

#### 簡單切分案例

以下函式保留空 Token，並將結果存成 `string_view`：

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
        const std::size_t end = text.find(delimiter, begin);

        if (end == std::string_view::npos)
        {
            result.push_back(text.substr(begin));
            break;
        }

        result.push_back(text.substr(begin, end - begin));
        begin = end + 1;
    }

    return result;
}
```

此規格下：

```text
"a,,b," -> ["a", "", "b", ""]
""       -> [""]
```

若希望空輸入得到零個 Token，或忽略空 Token，需要另外修改 Postcondition 與分支。

#### View 的生命週期再次出現

回傳的 `vector<string_view>` 全部指向輸入資料。因此：

```cpp
std::string input = "a,b,c";
auto parts = split(input, ',');
```

只要使用 `parts`，`input` 就必須仍然存在，而且不能發生使底層位置失效的修改。若 Token 需要獨立保存，應回傳 `vector<string>`。

#### 數字解析的規格

解析整數時需要確認：

- 是否允許前導空白。
- 是否允許 `+` 或 `-`。
- 是否至少要有一個數字。
- 是否允許前導 0。
- 超出目標型別範圍如何回報。
- 解析到非法 Byte 時停止，還是整體失敗。

Overflow 應在乘以 10 與加入下一位數字之前檢查，而不是等結果已超出型別範圍後再判斷。

### 6.13 C 語言中的字串

C String 通常以 `char` Array 儲存，並使用 Null Terminator `\0` 表示結尾：

```c
char text[] = "abc";
```

記憶體內容為：

```text
'a' 'b' 'c' '\0'
```

Array 容量是 4，文字長度是 3。

#### 容量與文字長度

在 C 中尤其需要區分：

- Buffer Capacity：配置了多少 Bytes。
- String Length：第一個 `\0` 前有多少 Bytes。
- 寫入後是否仍保留 Null Terminator 空間。

若 Buffer 容量為 `capacity`，最多只能保存 `capacity - 1` 個非 Null Bytes，最後一格要留給 `\0`。

#### `strlen` 的成本與前置條件

`strlen(text)` 從起點走訪到第一個 `\0`，時間為 O(n)。Precondition 是傳入位置後方存在可達的 Null Terminator。若 Buffer 沒有終止 Byte，函式可能讀出有效範圍。

在迴圈條件中反覆呼叫 `strlen`，可能重複掃描：

```c
for (size_t i = 0; i < strlen(text); ++i)
{
    process(text[i]);
}
```

若字串不會在迴圈中改變，可先保存長度：

```c
size_t length = strlen(text);
for (size_t i = 0; i < length; ++i)
{
    process(text[i]);
}
```

#### 明確傳入長度

若資料可以內含 Null Byte，或本來就不是 C String，應傳入 Pointer 與 Length：

```c
void process_bytes(
    const unsigned char data[],
    size_t length);
```

此時不能使用 `strlen` 判斷資料大小。

#### 安全建構介面

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

成功時必須同時維持：

- `*length < capacity`。
- `buffer[*length] == '\0'`。
- `[0, *length)` 是有效文字內容。

#### C 與 UTF-8

使用 `char*` 並不代表每個 `char` 是一個 Unicode 字元。UTF-8 在 C 中同樣是多 Byte 編碼，逐 `char` 反轉或截斷仍可能破壞編碼。

### 6.14 建立自己的字串分析表

<table>
<tr><th>欄位</th><th>要回答的問題</th></tr>
<tr><td>編碼</td><td>ASCII、UTF-8、其他編碼，還是純 Byte 資料？</td></tr>
<tr><td>比較單位</td><td>Byte、Code Point 還是可見字形？</td></tr>
<tr><td>Normalize</td><td>是否忽略大小寫、空白、標點或其他內容？</td></tr>
<tr><td>區間</td><td>使用 `[left, right)` 還是位置加長度？</td></tr>
<tr><td>候選結構</td><td>Substring、Subsequence 還是其他選取方式？</td></tr>
<tr><td>字元範圍</td><td>只含小寫英文、ASCII、任意 Byte 或 Unicode？</td></tr>
<tr><td>頻率結構</td><td>固定 Array、256 格 Array、Hash Map 或 Ordered Map？</td></tr>
<tr><td>Ownership</td><td>結果需要擁有資料，還是 View 即可？</td></tr>
<tr><td>生命週期</td><td>原始字串會活多久？是否會修改或 Reallocation？</td></tr>
<tr><td>複製成本</td><td>是否在迴圈或遞迴中反覆建立 Substring？</td></tr>
<tr><td>建構方式</td><td>尾端追加、預先建立大小，還是反覆前端插入？</td></tr>
<tr><td>解析規則</td><td>空 Token、連續 Delimiter 與尾端 Delimiter 如何處理？</td></tr>
<tr><td>錯誤表示</td><td>非法輸入、Overflow 或缺少 Token 時如何回報？</td></tr>
<tr><td>邊界案例</td><td>空字串、單一 Byte、多 Byte 文字與內含 Null 如何處理？</td></tr>
</table>

### 6.15 常見問題與判讀

<table>
<tr><th>現象</th><th>可能原因</th><th>第一輪檢查</th></tr>
<tr><td>Unicode 長度比預期大</td><td>把 Byte 數當成可見字元數</td><td>確認編碼與計數單位</td></tr>
<tr><td>反轉後出現亂碼</td><td>逐 Byte 反轉 UTF-8</td><td>是否需要依 Code Point 或 Grapheme Cluster 處理</td></tr>
<tr><td>Frequency Array 越界</td><td>字元範圍假設不成立</td><td>是否真的只含 `'a'` 到 `'z'`</td></tr>
<tr><td>`isdigit` 在部分輸入異常</td><td>直接傳入負值 `char`</td><td>先轉成 `unsigned char`</td></tr>
<tr><td>Substring 範圍太長</td><td>把右邊界當成 `substr` 的 Count</td><td>Count 是否應為 `right - left`</td></tr>
<tr><td>遞迴輸入不大但執行很慢</td><td>每層複製 Substring</td><td>改傳 Index、區間或 `string_view`</td></tr>
<tr><td>View 內容突然改變或失效</td><td>原資料被銷毀、修改或重新配置</td><td>檢查 Ownership 與生命週期</td></tr>
<tr><td>C 函式讀到 View 範圍之外</td><td>`string_view` 不保證 Null-terminated</td><td>是否需要建立 `std::string` 副本</td></tr>
<tr><td>回文答案和預期不同</td><td>Normalize 規則不一致</td><td>大小寫、標點、空白與編碼如何定義</td></tr>
<tr><td>Subsequence 判斷漏解</td><td>當成連續 Substring</td><td>是否允許跳過元素但保持順序</td></tr>
<tr><td>建構長字串愈來愈慢</td><td>反覆前端插入或建立完整暫時字串</td><td>改成尾端追加並視需要 `reserve`</td></tr>
<tr><td>Split 漏掉最後一個 Token</td><td>只在遇到 Delimiter 時輸出</td><td>迴圈結束後是否處理尾端範圍</td></tr>
<tr><td>連續 Delimiter 結果不一致</td><td>空 Token 規格未定義</td><td>明確決定保留或忽略空 Token</td></tr>
<tr><td>C String 偶爾讀取越界</td><td>缺少 Null Terminator</td><td>Buffer 是否保留一格並在結尾寫入 `\0`</td></tr>
<tr><td>含 Null Byte 的資料被截短</td><td>傳給依賴 C String 的介面</td><td>改用 Pointer 加 Length 介面</td></tr>
</table>



#### 針對 `substr` 與共同子串的判讀

- 若已經知道 `[left, right)`，可以用 `text.substr(left, right - left)` 切出結果。
- 若尚未知道共同片段在哪裡，應先寫搜尋邏輯，再使用 `substr` 輸出答案。
- 若只判斷固定 Pattern 是否存在，可先考慮 `find`。
- 若要最長共同 Substring，可使用雙迴圈或 DP。
- 若要最長共同 Subsequence，問題模型不同，不能沿用同一個 DP 轉移式。

<table>
<tr><th>問題</th><th>判讀方向</th><th>檢查重點</th></tr>
<tr><td>`substr(1, 4)` 為什麼得到 `bcde`？</td><td>第二個參數是 Count</td><td>若目標是 `[1, 4)`，應傳 `4 - 1`</td></tr>
<tr><td>兩個字串要找共同片段</td><td>這是搜尋問題</td><td>`substr` 只適合在已知區間後切出答案</td></tr>
<tr><td>`abc` 和 `abcabc` 都是共同子串，該回傳誰？</td><td>看 Postcondition</td><td>若要求最長，應回傳較長且連續的那段</td></tr>
<tr><td>DP 遇到不同字元時為什麼歸零？</td><td>Substring 必須連續</td><td>斷開後不能延續前一段共同長度</td></tr>
<tr><td>能不能用 `string_view` 取代所有 `substr`？</td><td>只能在生命週期安全時使用</td><td>確認原資料是否仍存在且位置穩定</td></tr>
</table>

### 6.16 本章檢查表

- 我知道 `std::string` 的 `size()` 回傳 Byte 數量。
- 我不會在未確認編碼時，把 Byte 數直接稱為 Unicode 字元數。
- 我能區分 Byte、Unicode Code Point 與 Grapheme Cluster。
- 我會先定義大小寫、空白、標點與其他 Normalize 規則。
- 我能區分 Substring、Subsequence 與 Subset。
- 我知道 `substr(position, count)` 的第二個參數是長度，不是右邊界。
- 我知道 `substr` 是切片工具，不負責搜尋兩個字串的共同片段。
- 我能區分固定 Pattern 搜尋、最長共同子串與最長共同子序列。
- 我能用雙 Index 與 Invariant 說明 Subsequence 判斷。
- 我能使用不發生 Unsigned Underflow 的回文區間。
- 我知道 ASCII 回文方法不能直接代表完整 Unicode 文字處理。
- 我會在呼叫 `<cctype>` 函式前轉成 `unsigned char`。
- 我知道固定 Frequency Array 需要明確的字元範圍 Precondition。
- 我能依值域選擇 26 格 Array、256 格 Array 或 Map。
- 我會根據最大輸入長度選擇頻率計數型別。
- 我了解 `std::string::substr` 通常會建立副本。
- 我能使用 Index、Half-open Interval 或 `string_view` 減少重複複製。
- 我知道 `string_view` 不擁有資料，也不延長原資料生命週期。
- 我不會回傳指向區域 `std::string` 的 View。
- 我知道原字串 Reallocation 可能讓既有 View 失效。
- 我知道 `string_view::data()` 不保證在 View 結尾處有 Null Terminator。
- 我能區分 `reserve` 與實際建立字元。
- 我會避免在迴圈中反覆從字串前端插入。
- 我會為 Split 明確定義空 Token 與尾端 Delimiter 行為。
- 我知道回傳 `vector<string_view>` 時，輸入資料必須持續有效。
- 我知道 C String 需要 Null Terminator，且 Capacity 必須保留結尾空間。
- 我知道 `strlen` 需要可達的 Null Terminator，而且每次呼叫都可能重新走訪。

### 6.17 本章重點

- `std::string` 是連續 Byte 容器，不是 Unicode 可見字形容器。
- ASCII 常可用一個 Byte 表示一個字元，但 UTF-8 使用可變數量 Bytes。
- 字串題開始前，應先確認編碼、比較單位與 Normalize 規則。
- Substring 必須連續；Subsequence 可跳過元素，但必須保持原順序。
- `std::string::substr` 適合在已知位置與長度後切出結果；共同子串問題仍需要搜尋方法先決定區間。
- Two Pointers 適合由左右兩端比較回文，Half-open Interval 可安全處理空字串。
- `<cctype>` 函式的 `char` 輸入應先轉成 `unsigned char`。
- 頻率資料結構由字元值域決定，固定 Array 需要明確 Precondition。
- `std::string::substr` 通常會複製內容，反覆使用可能增加時間與空間成本。
- Index 與 Half-open Interval 可以在不複製字串的情況下表示子問題。
- `std::string_view` 可建立非擁有 View，但原資料必須保持有效且位置穩定。
- View 不保證 Null-terminated，需要 C String 時應建立擁有資料的字串。
- 已知大致結果長度時可先 `reserve`，再從尾端使用 `push_back` 或 `+=`。
- 反覆前端插入需要搬移既有內容，可能形成 O(n²)。
- 解析前應定義 Delimiter、空 Token、空白、非法輸入與 Overflow 規則。
- C String 使用 `\0` 表示結尾，必須同時管理 Buffer Capacity 與目前 Length。
- 字串演算法的正確性不只取決於迴圈，也取決於編碼、Ownership 與生命週期假設。
