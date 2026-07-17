# 第 14 章　Prefix Sum 與 Difference Array

## 適用範圍

本章介紹一維與二維 Prefix Sum、區間查詢、Prefix Hash、Difference Array、靜態查詢與動態更新的差異。

## 適用讀者

- 面對大量區間和查詢仍逐段重算的讀者。
- 容易混淆 Inclusive 與 Half-open Interval 的讀者。

## 快速導覽

- [Prefix 定義](#141-prefix-定義)
- [區間查詢](#142-區間查詢)
- [Prefix 加 Hash](#143-prefix-加-hash)
- [二維 Prefix](#144-二維-prefix)
- [Difference Array](#145-difference-array)

## 14.1 Prefix 定義

令 `prefix[i]` 表示前 `i` 個元素之和：

```text
prefix[0] = 0
prefix[i+1] = prefix[i] + nums[i]
```

```cpp
std::vector<long long> buildPrefixSum(const std::vector<int>& nums)
{
    std::vector<long long> prefix(nums.size() + 1, 0);
    for (std::size_t i = 0; i < nums.size(); ++i)
        prefix[i + 1] = prefix[i] + nums[i];
    return prefix;
}
```

## 14.2 區間查詢

半開區間 `[left,right)`：

```text
sum = prefix[right] - prefix[left]
```

多一個 `prefix[0]` 讓從 0 開始和空區間自然處理。

## 14.3 Prefix 加 Hash

計算 Sum 等於 `k` 的 Subarray 數量：

```text
prefix[right] - prefix[left] = k
prefix[left] = prefix[right] - k
```

```cpp
long long countSubarraysWithSum(
    const std::vector<int>& nums, long long k)
{
    std::unordered_map<long long, long long> frequency;
    frequency[0] = 1;

    long long prefix = 0;
    long long answer = 0;
    for (int value : nums)
    {
        prefix += value;
        auto it = frequency.find(prefix - k);
        if (it != frequency.end()) answer += it->second;
        ++frequency[prefix];
    }
    return answer;
}
```

先查再增加目前 Prefix，避免把空長度區間誤計入。

## 14.4 二維 Prefix

矩形和使用 Inclusion-Exclusion：

```text
右下 Prefix - 上方 - 左方 + 左上重複扣除區
```

統一使用半開矩形 `[top,bottom) × [left,right)` 可降低邊界特例。

## 14.5 Difference Array

對 Inclusive 區間 `[left,right]` 加值：

```text
diff[left] += value
diff[right+1] -= value
```

最後對 `diff` 做 Prefix Sum 還原。若更新和查詢交錯，靜態 Difference Array 不足，可能需要 Fenwick Tree 或 Segment Tree。

## 14.6 Prefix 的一般化

- Sum：兩個 Prefix 相減。
- XOR：兩個 Prefix 再 XOR。
- Frequency Vector：逐欄相減。
- Min/Max：通常無法只用兩個 Prefix 組合任意區間答案。

## 14.7 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 區間差一格 | Inclusive/Half-open 混用 | 定義 `[l,r)` |
| 從 0 開始區間漏算 | 缺少空 Prefix | `prefix[0]=0` |
| Sum 溢位 | Prefix 使用 int | 改 `long long` |
| Prefix Hash 多算 | 查詢與插入順序錯 | 先查再插 |
| 更新後查詢錯 | 使用靜態 Prefix | 是否需動態結構 |

## 14.8 本章檢查表

- [ ] 能從 Prefix 定義推導區間公式。
- [ ] 能一致使用半開區間。
- [ ] 知道 Prefix Hash 為何加入 `frequency[0]=1`。
- [ ] 能區分靜態查詢和動態更新。
- [ ] 會檢查 Prefix 型別範圍。

## 14.9 本章重點

1. Prefix 將重複區間計算轉成預處理加常數查詢。
2. 半開區間能自然表示空區間。
3. Prefix Hash 可處理含負數的區間和計數。
4. Difference Array 適合批次區間更新後一次還原。
