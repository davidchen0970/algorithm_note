# 第 10 章　Hash Table、Set 與 Map

## 適用範圍

本章介紹 Key、Value、Hash Function、Collision、Bucket、Load Factor、Rehash、`unordered_set`、`unordered_map`、自訂 Key 與排序容器比較。

## 適用讀者

- 需要處理 Membership、Frequency 與 Value-to-State 映射的讀者。
- 容易把平均 `O(1)` 當成最差保證的讀者。

## 快速導覽

- [核心模型](#101-核心模型)
- [Set 和 Map](#102-set-和-map)
- [Frequency 和 State](#103-frequency-和-state)
- [Collision 與 Rehash](#104-collision-與-rehash)
- [Hash 和排序的選擇](#105-hash-和排序的選擇)

## 10.1 核心模型

Hash Function 將 Key 映射到 Hash Value，再定位 Bucket。不同 Key 可能 Collision，因此容器還要使用 Equality 判斷是否為同一 Key。

## 10.2 Set 和 Map

- `unordered_set<Key>`：保存是否存在。
- `unordered_map<Key, Value>`：保存 Key 對應的狀態。

`operator[]` 在 Key 不存在時會插入預設值。若只想查詢，不應無意間使用它。

## 10.3 Frequency 和 State

```cpp
std::unordered_map<int, int> countFrequency(const std::vector<int>& nums)
{
    std::unordered_map<int, int> frequency;
    for (int value : nums) ++frequency[value];
    return frequency;
}
```

Map Value 需依問題定義：

- Frequency。
- First/Last Index。
- 最佳結果。
- Parent 或距離。

相同 Key 不代表 Value 語意相同。

## 10.4 Collision 與 Rehash

Load Factor 約為元素數除以 Bucket 數。負載過高時容器可能 Rehash，搬移 Bucket 並使 Iterator 失效。已知規模時可 `reserve` 降低 Rehash 次數。

自訂 Key 必須滿足：若 `a == b`，則 `hash(a) == hash(b)`。

## 10.5 Hash 和排序的選擇

| 需求 | 常見選擇 |
|---|---|
| 快速存在性與 Frequency | Hash Table |
| 需要 Key 排序與範圍 | `std::map` 或排序 |
| 需要最小/最大 Key | Ordered Map |
| 需要穩定最差界線 | Tree-based Map 或排序 |

Hash Table 平均查找 `O(1)`，最差取決於 Collision 與實作；`std::map` 操作 `O(log n)` 並保持排序。

## 10.6 Two Sum 狀態

```text
Key   = Value
Value = 先前 Index
```

先查再插，避免同一 Index 配對自己。這個 State 定義比「用了 Hash Map」更重要。

## 10.7 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 查詢後容器多出 Key | 使用 `operator[]` | 改 `find`/`contains` |
| Iterator 突然失效 | Rehash | 插入期間是否持有 Iterator |
| 輸出順序不固定 | Hash 不排序 | 是否需要 Ordered Container |
| 自訂 Key 找不到 | Hash/Equality 不一致 | 相等 Key 的 Hash |
| 複雜度描述錯 | 混淆平均與最差 | 容器保證 |

## 10.8 本章檢查表

- [ ] 能區分 Set 與 Map。
- [ ] 能明確說出 Key 和 Value 的語意。
- [ ] 知道 `operator[]` 可能插入。
- [ ] 知道 Rehash 與 Iterator 失效。
- [ ] 能比較 Hash Table 和 Ordered Map。

## 10.9 本章重點

1. Hash Table 適合快速 Membership 與 Mapping。
2. Collision 由 Equality 進一步確認 Key。
3. Value 應保存問題真正需要的狀態。
4. Rehash 影響 Iterator 與實際成本。
5. Hash 不提供排序語意。
