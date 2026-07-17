# 第 5 章　Array 與 Dynamic Array

## 適用範圍

本章介紹連續儲存、Index、走訪、插入刪除、`std::vector` 的 Size、Capacity、Reallocation、Iterator Invalidation、二維資料與常見 Array 模式。

## 適用讀者

- 需要建立後續 Two Pointers、Sliding Window、Prefix Sum 基礎的讀者。
- 常混淆 `reserve`、`resize` 或遇到越界問題的讀者。

## 快速導覽

- [核心模型](#51-核心模型)
- [操作成本](#52-操作成本)
- [stdvector](#53-stdvector)
- [In-place 和 ReadWrite Pointer](#54-in-place-和-readwrite-pointer)
- [二維資料](#55-二維資料)
- [常見問題與判讀](#56-常見問題與判讀)

## 5.1 核心模型

Array 將相同型別元素放在連續位置，因此可由 Base Address 和 Index 計算第 `i` 個元素位置，隨機存取為 `O(1)`。

有效範圍通常寫成半開區間 `[0, size)`。

## 5.2 操作成本

| 動作 | 常見成本 | 原因 |
|---|---:|---|
| 按 Index 讀寫 | `O(1)` | 直接定位 |
| 未排序查找 | `O(n)` | 最差需完整走訪 |
| 尾端追加 | 攤銷 `O(1)` | 偶爾擴容 |
| 中間插入或刪除 | `O(n)` | 後方元素搬移 |

## 5.3 `std::vector`

- `size()`：目前元素數。
- `capacity()`：Reallocate 前可容納數量。
- `reserve(n)`：保留容量，不建立元素。
- `resize(n)`：改變元素數量。

```cpp
std::vector<int> values;
values.reserve(100);
values.push_back(7);
```

`reserve(100)` 後 `size()` 仍為 0，不能存取 `values[0]`。

### 5.3.1 Iterator Invalidation

Vector Reallocate 後，原有 Pointer、Reference 與 Iterator 可能失效。中間 Insert/Erase 也會影響該位置之後的 Iterator。

## 5.4 In-place 和 Read/Write Pointer

移除排序 Array 的重複值：

```cpp
int removeDuplicates(std::vector<int>& nums)
{
    if (nums.empty()) return 0;

    int write = 1;
    for (int read = 1; read < static_cast<int>(nums.size()); ++read)
    {
        if (nums[read] != nums[write - 1])
        {
            nums[write] = nums[read];
            ++write;
        }
    }
    return write;
}
```

Invariant：`[0, write)` 是已處理範圍的不重複結果。前置條件是 Array 已排序。

## 5.5 二維資料

`vector<vector<int>>` 每列各自配置，整體不保證一塊連續記憶體。真正連續可用：

```cpp
std::vector<int> matrix(rows * cols);
int& value = matrix[row * cols + col];
```

需分別檢查 Row 與 Column 邊界。

## 5.6 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 最後一次越界 | 使用 `<= size()` | 有效範圍 `[0,size)` |
| Reserve 後存取失敗 | 混淆 Size/Capacity | 是否先 `resize` 或 Push |
| Iterator 突然失效 | Vector Reallocate | 修改容器後是否沿用 Iterator |
| 倒序無法結束 | Unsigned Underflow | Index 型別與停止條件 |
| 二維走訪很慢 | 存取順序不連續 | Row-major 方向 |

## 5.7 本章檢查表

- [ ] 能說明連續儲存和 `O(1)` Index 的關係。
- [ ] 能區分 Size 與 Capacity。
- [ ] 知道哪些動作使 Iterator 失效。
- [ ] 能寫出 Read/Write Pointer invariant。
- [ ] 能安全處理空 Array 和倒序。

## 5.8 本章重點

1. Array 的核心優勢是連續儲存與隨機存取。
2. 中間 Insert/Erase 需要搬移元素。
3. `reserve` 不會建立元素。
4. Vector 修改可能讓 Pointer 與 Iterator 失效。
5. Array 是多數基礎演算法模式的共同載體。
