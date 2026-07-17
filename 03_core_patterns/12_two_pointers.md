# 第 12 章　Two Pointers

## 適用範圍

本章介紹相向 Pointer、同方向 Read/Write Pointer、排序 Pair、Partition 與快慢指標，重點是每次移動如何安全排除候選。

## 適用讀者

- 看到兩個 Index 便稱為 Two Pointers，但無法說明成立條件的讀者。
- 需要處理排序 Pair、去重與 Partition 的讀者。

## 快速導覽

- [共同本質](#121-共同本質)
- [有序 Two Sum](#122-有序-two-sum)
- [ReadWrite Pointer](#123-readwrite-pointer)
- [Partition](#124-partition)
- [不適用情境](#125-不適用情境)

## 12.1 共同本質

Two Pointers 的核心不是程式中有兩個 Index，而是 Pointer 移動後能安全排除一批候選，且不需回頭。

## 12.2 有序 Two Sum

```cpp
bool hasPairWithSum(const std::vector<int>& nums, int target)
{
    int left = 0;
    int right = static_cast<int>(nums.size()) - 1;

    while (left < right)
    {
        long long sum = static_cast<long long>(nums[left]) + nums[right];
        if (sum == target) return true;
        if (sum < target) ++left;
        else --right;
    }
    return false;
}
```

前置條件：Array 已排序。

- Sum 太小：Right 已是目前最大搭配，更小的 Left 不可能達標，所以增加 Left。
- Sum 太大：Left 已是目前最小搭配，更大的 Right 只會更大，所以減少 Right。

## 12.3 Read/Write Pointer

處理 Array Prefix 時：

```text
[0, write)    已整理結果
[write, read) 已處理但不需保留
[read, n)     尚未處理
```

這個區域 Invariant 適合移除、壓縮與 Partition。

## 12.4 Partition

Partition 將元素依 Predicate 分區。Quick Sort 的 Pivot Partition、把偶數移到前方，都屬於此模型。重點是先定義每段區域代表什麼，再決定 Swap 和 Pointer 更新。

## 12.5 不適用情境

- 無序資料卻利用 Value 大小排除候選。
- Pointer 移動後無法證明略過位置不可能是答案。
- 排序會破壞原 Index 或順序需求。
- Window State 因負數等條件無法單調維護。

## 12.6 Two Pointers 和 Sliding Window

Sliding Window 是同方向 Pointer 的一類，額外維護連續區間 State 與 Validity。一般 Two Pointers 也可處理排序 Pair、Linked List 與 Partition，不一定存在 Window。

## 12.7 Debug 表

```text
left | right | left value | right value | condition | movement | exclusion reason
```

若無法填寫 Exclusion Reason，需重新檢查方法是否成立。

## 12.8 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 漏掉 Pair | 移動方向錯誤 | 排序與排除理由 |
| 原 Index 錯誤 | 排序未保存 Index | Pair `(value,index)` |
| 重複答案 | 重複值跳過時機錯 | 找到答案後再跳過 |
| 無法終止 | Pointer 未嚴格前進 | 每輪更新 |

## 12.9 本章檢查表

- [ ] 能說明每次移動排除哪些候選。
- [ ] 能寫出區域 Invariant。
- [ ] 已確認排序是否允許。
- [ ] 能分辨 Two Pointers 與 Sliding Window。

## 12.10 本章重點

1. Pointer 移動必須有安全排除候選的理由。
2. 排序常提供需要的單調性。
3. Read/Write Pointer 管理已整理與未處理區域。
4. 區間與順序語意決定是否可使用此模式。
