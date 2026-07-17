# 第 11 章　Enumeration 與 Brute Force

## 適用範圍

本章介紹元素、Pair、區間、Subset、Permutation 的完整枚舉，說明搜尋空間、搜尋樹、剪枝、去重與直接解法作為 Oracle 的用途。

## 適用讀者

- 不知道如何建立第一個正確解法的讀者。
- 需要理解 Backtracking 前置概念的讀者。

## 快速導覽

- [為什麼先枚舉](#111-為什麼先枚舉)
- [常見候選空間](#112-常見候選空間)
- [Subset 和 Permutation](#113-subset-和-permutation)
- [剪枝](#114-剪枝)
- [作為測試 Oracle](#115-作為測試-oracle)

## 11.1 為什麼先枚舉

完整枚舉直接反映問題定義，適合：

- 驗證題意。
- 建立候選空間。
- 找出重複工作。
- 作為最佳化版對拍基準。

## 11.2 常見候選空間

### Pair

```cpp
for (int i = 0; i < n; ++i)
    for (int j = i + 1; j < n; ++j)
        check(i, j);
```

共有 `n(n-1)/2` 組，時間 `O(n²)`。

### 連續區間

非空區間數為 `n(n+1)/2`。若每次區間再重新求和，可能由 `O(n²)` 增為 `O(n³)`。

### Subset

`n` 個元素共有 `2^n` 個 Subset：

```cpp
for (long long mask = 0; mask < (1LL << n); ++mask)
{
    for (int i = 0; i < n; ++i)
        if (mask & (1LL << i)) select(i);
}
```

## 11.3 Subset 和 Permutation

- Subset：每個元素選或不選，不關心順序。
- Combination：選固定數量，通常不關心順序。
- Permutation：順序不同視為不同答案。

重複輸入值需要明確區分按 Value 去重或按 Index 區分。

## 11.4 剪枝

剪枝表示某個部分狀態不可能產生合法或更佳答案，因此停止整個 Subtree。每項剪枝都需說清楚前置條件。

例如所有數非負時，Current Sum 已超過 Target，後續加入只會更大。若允許負數，這項剪枝不成立。

## 11.5 作為測試 Oracle

將輸入限制在小範圍，以完整枚舉得到可靠答案，再和 Greedy、DP、Two Pointers 或 Hash 解法比較。

## 11.6 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 重複 Pair | 同時枚舉 `(i,j)` 和 `(j,i)` | `j=i+1` |
| Subset 數溢位 | `1 << n` 使用 int | `1LL` 與 n 範圍 |
| 剪枝漏解 | 前置條件不成立 | 負數、排序、界線 |
| 重複答案 | 去重層級錯誤 | 同層或跨層 |
| 暴力版也錯 | 候選空間不完整 | 是否涵蓋所有可能 |

## 11.7 本章檢查表

- [ ] 能計算 Pair、Subset、Permutation 規模。
- [ ] 能區分 Combination 與 Permutation。
- [ ] 能為剪枝說明排除整棵 Subtree 的理由。
- [ ] 會保留小型直接解法作為 Oracle。

## 11.8 本章重點

1. 完整枚舉是理解問題與驗證最佳化的重要工具。
2. 候選空間決定基礎複雜度。
3. 剪枝必須有正確性依據。
4. 去重需明確定義 Value 與 Index 語意。
