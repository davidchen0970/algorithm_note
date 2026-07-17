# 第 6 章　String 與字元處理

## 適用範圍

本章介紹 `std::string`、Byte 與字元、Substring、Subsequence、Palindrome、Frequency、字串建構成本、`string_view` 與常見解析問題。

## 適用讀者

- 需要處理字元走訪、回文、頻率與連續字串問題的讀者。
- 容易把 Byte 數當成 Unicode 字元數的讀者。

## 快速導覽

- [String 的資料模型](#61-string-的資料模型)
- [Substring 和 Subsequence](#62-substring-和-subsequence)
- [回文](#63-回文)
- [頻率統計](#64-頻率統計)
- [複製與生命週期](#65-複製與生命週期)

## 6.1 String 的資料模型

`std::string` 是 Byte 序列。ASCII 常是一個 Byte 對應一個字元；UTF-8 的一個可見字元可能使用多個 Byte。因此 `size()` 回傳 Byte 數，不保證是人類可見字元數。

使用 `std::isdigit` 等函式時，應轉成 `unsigned char`：

```cpp
bool isDigit(char ch)
{
    return std::isdigit(static_cast<unsigned char>(ch)) != 0;
}
```

## 6.2 Substring 和 Subsequence

- Substring：連續區間。
- Subsequence：保留順序但可跳過元素。

`"ace"` 是 `"abcde"` 的 Subsequence，不是 Substring。

## 6.3 回文

```cpp
bool isPalindrome(const std::string& text)
{
    int left = 0;
    int right = static_cast<int>(text.size()) - 1;
    while (left < right)
    {
        if (text[left] != text[right]) return false;
        ++left;
        --right;
    }
    return true;
}
```

若要求忽略大小寫或非英數字元，需先明確定義 Normalize 規則。

## 6.4 頻率統計

輸入保證小寫英文字母時：

```cpp
std::array<int, 26> frequency{};
for (char ch : text)
{
    ++frequency[ch - 'a'];
}
```

前置條件不成立時，`ch - 'a'` 可能越界。字元範圍不固定時使用 Hash Map。

## 6.5 複製與生命週期

`substr` 通常建立新字串，大量使用可能增加時間與空間。`std::string_view` 可建立不擁有資料的 View，但原字串銷毀或修改後，View 可能失效。

```cpp
std::string_view middle(std::string_view text)
{
    if (text.size() < 2) return {};
    return text.substr(1, text.size() - 2);
}
```

不可回傳指向區域 `std::string` 的 View。

## 6.6 字串建構

已知大致長度時可先 `reserve`，再使用 `push_back` 或 `+=`。反覆在前端插入通常需要搬移資料，可能形成 `O(n²)`。

## 6.7 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| Unicode 長度錯誤 | 把 Byte 當字元 | 編碼與需求 |
| Frequency 越界 | 字元範圍假設錯 | 是否只含小寫英文 |
| 遞迴意外變慢 | 每層建立 Substring | 改傳 Index 或 View |
| View 內容異常 | 原資料已失效 | Ownership 與生命週期 |
| 回文邏輯錯 | Normalize 規則不明 | 大小寫與過濾條件 |

## 6.8 本章檢查表

- [ ] 能區分 Byte、Substring 與 Subsequence。
- [ ] 知道固定 Frequency Array 的前置條件。
- [ ] 了解 `substr` 的複製成本。
- [ ] 了解 `string_view` 不擁有資料。
- [ ] 能為回文題明確定義比較規則。

## 6.9 本章重點

1. `std::string` 是 Byte 容器。
2. Substring 必須連續，Subsequence 不必。
3. 字元範圍決定 Frequency 的資料結構。
4. `string_view` 可避免複製，但必須管理生命週期。
5. 字串題首先要明確定義編碼與正規化語意。
