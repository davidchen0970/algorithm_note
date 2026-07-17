# 第 13 章　Sliding Window

## 適用範圍

本章介紹固定與可變長度 Window、Expand、Shrink、Window State、Validity、答案更新時機，以及含負數時的限制。

## 適用讀者

- 會背模板，但不清楚 Left 為何移動的讀者。
- 容易混淆最長與最短區間更新時機的讀者。

## 快速導覽

- [四個必要組件](#131-四個必要組件)
- [固定長度 Window](#132-固定長度-window)
- [可變長度 Window](#133-可變長度-window)
- [最長無重複字串](#134-最長無重複字串)
- [不適用情境](#135-不適用情境)

## 13.1 四個必要組件

1. Window 邊界，例如 `[left,right]`。
2. Window State，例如 Sum 或 Frequency。
3. Validity 條件。
4. 答案更新時機。

## 13.2 固定長度 Window

```cpp
std::optional<long long> maximumWindowSum(
    const std::vector<int>& nums, int k)
{
    if (k <= 0 || k > static_cast<int>(nums.size()))
        return std::nullopt;

    long long current = 0;
    for (int i = 0; i < k; ++i) current += nums[i];

    long long answer = current;
    for (int right = k; right < static_cast<int>(nums.size()); ++right)
    {
        current += nums[right];
        current -= nums[right - k];
        answer = std::max(answer, current);
    }
    return answer;
}
```

每個元素加入與移除各一次，時間 `O(n)`。

## 13.3 可變長度 Window

全為正數時，求 Sum 至少 Target 的最短區間：

```cpp
int minimumLengthAtLeastTarget(
    const std::vector<int>& nums, long long target)
{
    int left = 0;
    int answer = static_cast<int>(nums.size()) + 1;
    long long sum = 0;

    for (int right = 0; right < static_cast<int>(nums.size()); ++right)
    {
        sum += nums[right];
        while (sum >= target)
        {
            answer = std::min(answer, right - left + 1);
            sum -= nums[left++];
        }
    }
    return answer == static_cast<int>(nums.size()) + 1 ? 0 : answer;
}
```

正數條件讓 Expand 不會降低 Sum，Shrink 不會提高 Sum，因此可單調調整。

## 13.4 最長無重複字串

```cpp
int longestUniqueSubstring(const std::string& text)
{
    std::array<int, 256> frequency{};
    int left = 0;
    int answer = 0;

    for (int right = 0; right < static_cast<int>(text.size()); ++right)
    {
        unsigned char ch = static_cast<unsigned char>(text[right]);
        ++frequency[ch];
        while (frequency[ch] > 1)
        {
            --frequency[static_cast<unsigned char>(text[left])];
            ++left;
        }
        answer = std::max(answer, right - left + 1);
    }
    return answer;
}
```

此版本按 Byte 處理，不是完整 Unicode Grapheme 解法。

## 13.5 不適用情境

含負數的最短 Sum Window 可能失效，因為加入元素可能讓 Sum 下降，移除左端可能讓 Sum 上升。此時可能需要 Prefix Sum 與 Monotonic Deque。

## 13.6 最長與最短的更新時機

- 最長合法 Window：通常先收縮到合法，再更新最大長度。
- 最短滿足條件 Window：條件成立時，在 `while` 內先記錄，再繼續收縮。

## 13.7 複雜度

Right 移動 `n` 次，Left 最多也移動 `n` 次。所有 `while` 加總仍為 `O(n)`。

## 13.8 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 結果差一 | Window Length 少 `+1` | 邊界定義 |
| 含負數漏解 | 單調性不存在 | 換方法 |
| State 越來越錯 | Shrink 未撤銷狀態 | 元素離開更新 |
| 求最短卻得到較長 | 答案更新時機錯 | 是否在 `while` 內 |

## 13.9 本章檢查表

- [ ] 已定義 Boundary、State、Validity 與答案時機。
- [ ] 能證明 Left 不需回頭。
- [ ] 知道含負數時哪些 Sum 問題失效。
- [ ] 能說明總時間為 `O(n)`。

## 13.10 本章重點

1. Window 需同時定義邊界、狀態、合法條件和答案時機。
2. State 必須支援元素進出時的增量更新。
3. 單調性決定 Left 是否能只向前。
4. 最長與最短問題的收縮及更新位置不同。
