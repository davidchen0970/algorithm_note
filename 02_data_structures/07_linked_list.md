# 第 7 章　Linked List

## 適用範圍

本章介紹 Singly Linked List、Node、插入刪除、Dummy Node、反轉、合併、快慢指標、Cycle Detection 與 C++ Ownership。

## 適用讀者

- 容易在 Pointer 更新時遺失剩餘鏈結的讀者。
- 需要理解 Linked List 與 Array 成本差異的讀者。

## 快速導覽

- [核心模型](#71-核心模型)
- [Dummy Node](#72-dummy-node)
- [反轉](#73-反轉-linked-list)
- [合併排序 List](#74-合併排序-list)
- [快慢指標](#75-快慢指標)

## 7.1 核心模型

```cpp
struct ListNode
{
    int value;
    ListNode* next;
};
```

Linked List 不支援按 Index 的 `O(1)` 存取。若已持有前一個 Node，插入刪除可為 `O(1)`，但尋找位置仍可能 `O(n)`。

## 7.2 Dummy Node

Dummy Node 統一 Head 被修改的情況：

```cpp
ListNode* removeValue(ListNode* head, int target)
{
    ListNode dummy{0, head};
    ListNode* current = &dummy;

    while (current->next != nullptr)
    {
        if (current->next->value == target)
        {
            current->next = current->next->next;
        }
        else
        {
            current = current->next;
        }
    }
    return dummy.next;
}
```

此範例未處理被移除 Node 的釋放，實際專案需明確定義 Ownership。

## 7.3 反轉 Linked List

```cpp
ListNode* reverseList(ListNode* head)
{
    ListNode* previous = nullptr;
    ListNode* current = head;

    while (current != nullptr)
    {
        ListNode* next = current->next;
        current->next = previous;
        previous = current;
        current = next;
    }
    return previous;
}
```

更新前必須先保存 `next`。Invariant：`previous` 指向已反轉 Prefix，`current` 指向尚未處理部分。

## 7.4 合併排序 List

```cpp
ListNode* mergeSortedLists(ListNode* a, ListNode* b)
{
    ListNode dummy{0, nullptr};
    ListNode* tail = &dummy;

    while (a != nullptr && b != nullptr)
    {
        if (a->value <= b->value)
        {
            tail->next = a;
            a = a->next;
        }
        else
        {
            tail->next = b;
            b = b->next;
        }
        tail = tail->next;
    }
    tail->next = (a != nullptr) ? a : b;
    return dummy.next;
}
```

## 7.5 快慢指標

Cycle Detection：

```cpp
bool hasCycle(ListNode* head)
{
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast != nullptr && fast->next != nullptr)
    {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

偶數長度時「中點」需明確定義為左中點或右中點，初值與停止條件會影響結果。

## 7.6 Pointer Debug 方法

每輪畫出：

```text
previous → current → next → remaining
```

記錄改寫前後每個 Pointer 指向。Linked List Bug 通常出現在更新順序，而不是比較條件。

## 7.7 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 後半 List 消失 | 改 `next` 前未保存 | 暫存 Next |
| Head 刪除錯誤 | 缺少 Dummy Node 或特例 | 回傳的新 Head |
| Cycle 判斷錯誤 | 比較 Value 而非 Node | Pointer Identity |
| Merge 無限迴圈 | 某條 List Pointer 未前進 | 每輪更新 |
| Memory Leak | Ownership 未處理 | Delete 或 Smart Pointer |

## 7.8 本章檢查表

- [ ] 能說明存取與插入成本。
- [ ] 會在改寫 `next` 前保存剩餘鏈結。
- [ ] 能使用 Dummy Node 消除 Head 特例。
- [ ] 能寫出反轉的區域 Invariant。
- [ ] 能區分 Pointer Identity 和 Node Value。

## 7.9 本章重點

1. Linked List 以鏈結換取局部插入刪除彈性。
2. Pointer 更新順序是實作核心。
3. Dummy Node 統一 Head 邊界。
4. 快慢指標可處理中點與 Cycle。
5. 演算法鏈結和 C++ Ownership 必須分開考慮。
