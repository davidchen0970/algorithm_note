# 第 15 章　Binary Search

## 適用範圍

本章介紹 Exact Search、Lower Bound、Upper Bound、First True、Last True、Binary Search on Answer，以及 Closed 與 Half-open Interval 的差異。

## 適用讀者

- 會背 `left`、`right`、`mid`，但不理解更新理由的讀者。
- 常遇到死迴圈或邊界差一的讀者。

## 快速導覽

- [必要條件](#151-必要條件)
- [半開區間](#152-半開區間)
- [Lower Bound](#153-lower-bound)
- [Upper Bound](#154-upper-bound)
- [答案搜尋](#155-binary-search-on-answer)
- [Debug](#156-debug-方法)

## 15.1 必要條件

1. 有序候選空間。
2. 比較或 Predicate 具有單調性。
3. 每次更新後候選範圍嚴格縮小。

Binary Search 的核心是安全排除一半，不是單純計算 Mid。

## 15.2 半開區間

搜尋 `[left,right)`：

```cpp
int binarySearch(const std::vector<int>& nums, int target)
{
    int left = 0;
    int right = static_cast<int>(nums.size());

    while (left < right)
    {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target)
            left = mid + 1;
        else
            right = mid;
    }

    if (left < static_cast<int>(nums.size()) && nums[left] == target)
        return left;
    return -1;
}
```

這先找第一個 `>= target` 的位置，再確認是否相等。

## 15.3 Lower Bound

```cpp
int lowerBound(const std::vector<int>& nums, int target)
{
    int left = 0;
    int right = static_cast<int>(nums.size());
    while (left < right)
    {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) left = mid + 1;
        else right = mid;
    }
    return left;
}
```

結束時 `[0,left)` 都小於 Target，`left` 是第一個可能不小於 Target 的位置，也可能等於 `n`。

## 15.4 Upper Bound

```cpp
int upperBound(const std::vector<int>& nums, int target)
{
    int left = 0;
    int right = static_cast<int>(nums.size());
    while (left < right)
    {
        int mid = left + (right - left) / 2;
        if (nums[mid] <= target) left = mid + 1;
        else right = mid;
    }
    return left;
}
```

`upperBound - lowerBound` 是 Target 出現次數。

## 15.5 Binary Search on Answer

若 `feasible(x)` 呈現：

```text
false false false true true true
```

可找第一個 True：

```cpp
template <class Predicate>
long long firstTrue(long long left, long long right, Predicate predicate)
{
    while (left < right)
    {
        long long mid = left + (right - left) / 2;
        if (predicate(mid)) right = mid;
        else left = mid + 1;
    }
    return left;
}
```

呼叫端必須確保搜尋範圍涵蓋答案，並定義沒有可行答案時的政策。

## 15.6 Debug 方法

逐輪記錄：

```text
left | mid | right | predicate(mid) | new interval | exclusion reason
```

檢查：

- Mid 在目前區間內。
- 新區間嚴格變小。
- 被排除部分不可能含答案。
- 結束位置滿足 Postcondition。

### 15.6.1 逐步追蹤

`[1,3,3,5]` 找第一個 `>=3`：

```text
left=0 right=4 mid=2 value=3 → right=2
left=0 right=2 mid=1 value=3 → right=1
left=0 right=1 mid=0 value=1 → left=1
結束 left=1
```

## 15.7 Closed 與 Half-open 不可混用

Closed `[left,right]` 常使用 `left <= right` 和 `right = mid - 1`；Half-open `[left,right)` 使用 `left < right` 和 `right = mid`。每一行都必須由區間定義推導，不能混搭模板。

## 15.8 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 兩元素死迴圈 | 更新後區間未縮小 | `left=mid` 等寫法 |
| 找到錯的重複值 | Exact Search 不保證邊界 | Lower/Upper Bound |
| 回傳 n 被當成合法 Index | 未檢查不存在 | Position 和 Element 分開 |
| 答案搜尋錯誤 | Predicate 不單調 | Truth Table |
| 大範圍溢位 | Mid 算法或上界倍增 | `left+(right-left)/2` |

## 15.9 本章檢查表

- [ ] 先定義搜尋區間與 Predicate。
- [ ] 能說明每次排除哪一半。
- [ ] 不混用 Closed 與 Half-open 更新規則。
- [ ] 能處理答案不存在與回傳 `n`。
- [ ] 能用兩元素案例檢查停機。
- [ ] 能證明答案 Predicate 單調。

## 15.10 本章重點

1. Binary Search 依賴有序候選和單調性。
2. 區間定義決定迴圈與更新規則。
3. Lower Bound 與 Upper Bound 是邊界搜尋的核心。
4. Answer Search 先定義 Predicate，再搜尋分界。
5. 每次更新都必須嚴格縮小候選區間。
