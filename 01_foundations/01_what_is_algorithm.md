# 第 1 章　演算法是什麼

## 適用範圍

本章說明演算法的定義、五項核心特性、問題規格、演算法和程式的差異，以及從直接解法推導最佳化的方法。

## 適用讀者

- 初次系統化學習演算法的讀者。
- 能閱讀基本 C++，但不確定如何分析題目的讀者。
- 想從記憶答案改為依規格與瓶頸推導解法的讀者。

## 快速導覽

- [演算法的定義](#11-演算法的定義)
- [問題規格](#12-問題輸入與輸出)
- [演算法和程式](#13-演算法和程式)
- [直接解法](#14-從直接解法開始)
- [Two Sum 推導](#15-two-sum-完整推導)
- [常見問題](#16-常見問題與判讀)

## 1.1 演算法的定義

演算法是一組用來解決特定問題的有限且明確步驟。一個演算法應具備：

- **輸入**：接受零個或多個初始資料。
- **輸出**：產生至少一個與問題相關的結果。
- **明確性**：每個步驟都有清楚且唯一的解讀。
- **有限性**：對合法輸入會在有限步驟後結束。
- **可行性**：每個步驟都能由實際計算完成。

有限性和效率不同。指數演算法可能終止，但對大型輸入仍不可行。

## 1.2 問題、輸入與輸出

設計前先確認輸入型別、數量、值域、排序、重複條件、輸出形式、無答案政策，以及是否允許修改輸入。

```cpp
std::optional<int> maximum(const std::vector<int>& nums)
{
    if (nums.empty()) return std::nullopt;
    int answer = nums[0];
    for (std::size_t i = 1; i < nums.size(); ++i)
        answer = std::max(answer, nums[i]);
    return answer;
}
```

空 Array 沒有最大值，所以使用 `std::optional`。合法值 `0`、`-1` 或 `INT_MIN` 不應被任意當成無答案標記。

## 1.3 演算法和程式

演算法是抽象解題步驟；程式是特定語言、容器與型別下的實作。演算法正確不代表程式必然正確，Index 越界、Overflow、Pointer 失效與錯誤 Comparator 都可能破壞結果。

一個解法需同時評估：

- 正確性。
- 時間與額外空間。
- 可讀性。
- 可驗證性。

## 1.4 從直接解法開始

直接解法完整檢查候選，適合確認題意、建立基準、找出瓶頸並作為對拍 Oracle。

```cpp
std::optional<std::pair<int,int>> twoSumBruteForce(
    const std::vector<int>& nums, int target)
{
    for (int i = 0; i < static_cast<int>(nums.size()); ++i)
        for (int j = i + 1; j < static_cast<int>(nums.size()); ++j)
            if (static_cast<long long>(nums[i]) + nums[j] == target)
                return std::pair{i, j};
    return std::nullopt;
}
```

它檢查所有不同 Index Pair，時間 `O(n²)`，額外空間 `O(1)`。

設計流程：

```text
定義輸入輸出與限制
→ 建立小案例
→ 確認候選空間
→ 寫直接解法
→ 找出重複工作
→ 選擇資料結構
→ 重新證明與測試
```

## 1.5 Two Sum 完整推導

直接解法固定 `i` 後，反覆搜尋 `target - nums[i]`。可用 Hash Table 保存 `Value → 先前 Index`。

```cpp
std::optional<std::pair<int,int>> twoSumHash(
    const std::vector<int>& nums, int target)
{
    std::unordered_map<long long,int> indexByValue;
    indexByValue.reserve(nums.size());

    for (int i = 0; i < static_cast<int>(nums.size()); ++i)
    {
        long long value = nums[i];
        long long needed = static_cast<long long>(target) - value;
        auto it = indexByValue.find(needed);
        if (it != indexByValue.end()) return std::pair{it->second, i};
        indexByValue[value] = i;
    }
    return std::nullopt;
}
```

先查再插可避免目前元素和自己配對。Invariant：每輪開始時，Map 只保存目前 Index 之前的元素。平均時間 `O(n)`，額外空間 `O(n)`。

| i | value | needed | 查詢前 Map | 結果 |
|---:|---:|---:|---|---|
| 0 | 2 | 7 | 空 | 插入 2 → 0 |
| 1 | 7 | 2 | 2 → 0 | 回傳 0、1 |

## 1.6 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 回傳同一 Index | 先插再查 | 改為先查再插 |
| 大數答案錯 | Integer Overflow | Sum 與 Complement 型別 |
| 排序後 Index 錯 | 遺失原位置 | 保存 `(value,index)` |
| Sample 通過但隨機失敗 | 邊界或規格誤解 | 與直接解法對拍 |

## 1.7 本章檢查表

- [ ] 能說明五項核心特性。
- [ ] 能整理輸入、輸出、限制與無答案政策。
- [ ] 能區分演算法和 C++ 實作。
- [ ] 能先寫直接解法並描述候選空間。
- [ ] 能指出重複工作並選擇工具。
- [ ] 能寫出 Precondition、Postcondition 與 Invariant。

## 1.8 本章重點

1. 演算法由輸入、輸出、明確性、有限性與可行性構成。
2. 問題規格決定合法資料與答案語意。
3. 直接解法是推導與驗證最佳化的基礎。
4. 最佳化應對應明確瓶頸。
5. 邊界、Overflow 與多答案驗證屬於解法的一部分。
