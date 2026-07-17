## 第 7 章　Linked List

### 適用範圍

本章介紹 Linked List 的資料模型，以及 Node、Pointer 與鏈結更新如何共同決定演算法是否正確。

Linked List 和 Array 都能保存一串元素，但兩者的記憶體模型不同。Array 的元素依序位於連續空間，可以按 Index 直接定位；Linked List 的 Node 不必相鄰，而是透過 `next` 保存下一個 Node 的位置。這項差異會直接影響：

- 為什麼 Linked List 不支援 O(1) 的任意 Index 存取。
- 為什麼已知前一個 Node 時，可以在 O(1) 內修改局部鏈結。
- 為什麼尋找插入或刪除位置仍可能需要 O(n)。
- 為什麼 Pointer 更新順序會造成資料遺失、Cycle 或 Memory Leak。
- 為什麼 Head 經常需要額外的邊界處理。
- 為什麼演算法正確性與記憶體 Ownership 必須分開分析。

本章會建立一套固定流程：

1. 先畫出每個 Node 與 `next` 的方向。
2. 明確區分 Node Value 與 Node Identity。
3. 修改鏈結前，先保存之後仍需要的 Pointer。
4. 使用 Dummy Node 統一 Head 可能改變的情況。
5. 使用 Invariant 描述已處理區段與尚未處理區段。
6. 每輪確認所有必要 Pointer 都有前進。
7. 使用快慢指標前，先定義速度、停止條件與偶數長度語意。
8. 最後另外確認 Node 的配置、移轉與釋放責任。

### 適用讀者

- 容易在改寫 `next` 後遺失剩餘鏈結的讀者。
- 需要理解 Linked List 與 Array 成本差異的讀者。
- 經常為 Head 撰寫多組特殊分支的讀者。
- 會寫反轉程式，但無法說明 Pointer State 的讀者。
- 在 Merge、刪除或分割 List 時遇到無窮迴圈的讀者。
- 容易用 Node Value 代替 Pointer Identity 判斷 Cycle 的讀者。
- 需要區分演算法鏈結與 C++ Ownership 的讀者。
- 同時使用 C++ 與 C，希望建立一致 Pointer 推理方式的讀者。

### 快速導覽

- [Linked List 到底保存什麼](#71-linked-list-到底保存什麼)：建立 Node、鏈結與 Head 模型。
- [第一步：比較 Linked List 與 Array 成本](#72-第一步比較-linked-list-與-array-成本)：從定位與搬移理解複雜度。
- [第二步：建立安全的 Pointer 更新順序](#73-第二步建立安全的-pointer-更新順序)：避免遺失剩餘鏈結。
- [第三步：使用 Dummy Node](#74-第三步使用-dummy-node)：統一 Head 修改與空 List 邊界。
- [完整案例：移除指定值](#75-完整案例移除指定值)：使用前驅 Node 與 Invariant。
- [第四步：反轉 Linked List](#76-第四步反轉-linked-list)：拆分已反轉與尚未處理區段。
- [完整案例：合併兩條排序 List](#77-完整案例合併兩條排序-list)：使用 Dummy、Tail 與兩個輸入 Pointer。
- [第五步：使用快慢指標](#78-第五步使用快慢指標)：處理中點與速度差。
- [完整案例：Cycle Detection](#79-完整案例-cycle-detection)：使用 Pointer Identity 判斷相遇。
- [第六步：定義 Ownership 與釋放責任](#710-第六步定義-ownership-與釋放責任)：分開處理鏈結正確性與資源管理。
- [第七步：系統化 Debug Pointer](#711-第七步系統化-debug-pointer)：記錄第一個錯誤鏈結狀態。
- [C 語言中的 Linked List](#712-c-語言中的-linked-list)：補充配置、釋放與輸出參數。
- [建立自己的 Linked List 分析表](#713-建立自己的-linked-list-分析表)：形成固定檢查流程。
- [常見問題與判讀](#714-常見問題與判讀)：整理常見失敗現象。
- [本章檢查表](#715-本章檢查表)：確認必要觀念是否完整。
- [本章重點](#716-本章重點)：回顧核心方法。

### 7.1 Linked List 到底保存什麼

Singly Linked List 的每個 Node 通常保存兩項資訊：

- `value`：目前 Node 的資料。
- `next`：下一個 Node 的位置。

```cpp
struct ListNode
{
    int value;
    ListNode* next;
};
```

一條 List 可以畫成：

```text
head
 ↓
[4 | next] → [7 | next] → [2 | null]
```

`head` 不是整條 List 本身，而是第一個 Node 的 Pointer。最後一個 Node 的 `next == nullptr`，表示鏈結結束。

#### 空 List

空 List 通常表示為：

```cpp
ListNode* head = nullptr;
```

因此，任何讀取 `head->value` 或 `head->next` 的程式，都需要先確認 `head != nullptr`，或由 Precondition 保證非空。

#### Node 不需要連續配置

`next` 保存的是下一個 Node 的位置，因此 Node 可以位於不同記憶體區域。這表示：

- 不能由 `head + i` 找到第 `i` 個 Node。
- 必須從 Head 逐一沿著 `next` 前進。
- 已知某 Node 時，可以直接存取它的下一個 Node。

#### Value 與 Identity 不同

兩個 Node 可以有相同 Value，但仍是不同 Node：

```text
位址 A：[5 | next]
位址 B：[5 | next]
```

判斷數值相同：

```cpp
a->value == b->value
```

判斷是否為同一個 Node：

```cpp
a == b
```

Cycle Detection 必須比較 Pointer Identity。不同 Node 即使 Value 相同，也不代表快慢指標相遇。

#### Head 可能改變

以下動作可能產生新的 Head：

- 刪除第一個 Node。
- 在開頭插入 Node。
- 反轉整條 List。
- 合併兩條 List。

因此，修改函式通常需要回傳新的 Head，或接收能修改 Head 的介面。

### 7.2 第一步：比較 Linked List 與 Array 成本

Linked List 的成本不能只看「插入為 O(1)」。需要先問：是否已經持有正確位置？

<table>
<tr><th>動作</th><th>Linked List 常見成本</th><th>原因</th></tr>
<tr><td>讀取 Head</td><td>O(1)</td><td>已有 Head Pointer</td></tr>
<tr><td>讀取第 i 個 Node</td><td>O(i)</td><td>必須沿 `next` 逐一前進</td></tr>
<tr><td>查找某個 Value</td><td>O(n)</td><td>最差走訪全部 Node</td></tr>
<tr><td>已知前驅時插入</td><td>O(1)</td><td>只需修改固定數量鏈結</td></tr>
<tr><td>已知前驅時刪除</td><td>O(1)</td><td>只需略過目標 Node</td></tr>
<tr><td>尋找位置後插入</td><td>O(n)</td><td>尋找位置本身需要走訪</td></tr>
<tr><td>反轉整條 List</td><td>O(n)</td><td>每個 `next` 都需要改寫</td></tr>
</table>

#### 插入為何可以是 O(1)

已知 `previous` 與要插入的 `node`：

```text
修改前：previous → next
修改後：previous → node → next
```

需要兩次鏈結更新：

```cpp
node->next = previous->next;
previous->next = node;
```

但若題目只給 Index，仍需先走到正確前驅，因此完整動作通常是 O(n)。

#### 刪除為何需要前驅

Singly Linked List 的 Node 只知道下一個位置，不知道前一個位置。若要刪除 `current->next`：

```cpp
current->next = current->next->next;
```

因此，走訪時保存「目標前一個 Node」常比只保存目標本身更方便。

#### Linked List 不一定比 Array 快

即使局部插入不需搬移大量元素，Linked List 仍可能有：

- 尋找位置成本。
- 每個 Node 額外保存 Pointer 的空間。
- 個別配置與釋放成本。
- 存取位置分散造成的實際效能差異。

資料結構選擇應以主要動作、資料大小與介面需求為基礎，不能只比較單一複雜度欄位。

### 7.3 第二步：建立安全的 Pointer 更新順序

Linked List 程式的核心通常不是比較條件，而是鏈結改寫順序。

#### 改寫前先問三個問題

每次準備修改 `next` 前，先確認：

1. 修改後，如何找到尚未處理的剩餘 List？
2. 哪個 Pointer 仍代表已處理結果？
3. 是否可能讓某個 Node 暫時或永久無法到達？

#### 遺失剩餘鏈結

反轉時若直接寫：

```cpp
current->next = previous;
current = current->next;
```

第二行會沿著剛改寫的方向回到 `previous`，原本 `current` 後方的 List 失去入口。

正確順序是先保存：

```cpp
ListNode* next = current->next;
current->next = previous;
previous = current;
current = next;
```

#### 局部圖比變數名稱更重要

更新前：

```text
previous ← current → next → remaining
```

更新後：

```text
previous ← current    next → remaining
           ↑
       新 previous
```

每輪都應能說明：

- 哪一段已完成。
- 哪一段尚未處理。
- 下一輪從哪裡開始。

#### Pointer Assignment 不會複製 Node

```cpp
ListNode* a = head;
ListNode* b = a;
```

這不會建立兩個 Node。`a` 與 `b` 都指向同一個 Node。透過其中一個 Pointer 修改 Node，另一個 Pointer 看到的是相同物件。

### 7.4 第三步：使用 Dummy Node

Dummy Node 是暫時放在真正 Head 前方的輔助 Node：

```text
dummy → head → ...
```

它的主要用途是讓第一個真實 Node 也有前驅，從而統一 Head 與中間位置的處理方式。

```cpp
ListNode dummy{0, head};
```

`dummy.value` 通常沒有演算法意義，重要的是 `dummy.next` 指向目前 Head。

#### 沒有 Dummy 時

刪除目標值通常需要分開處理：

- Head 是否為目標。
- Head 後方是否有目標。
- 連續多個 Head 是否都要刪除。

#### 使用 Dummy 後

所有刪除都可描述為：

> 檢查 `current->next`，若它是目標，就讓 `current->next` 略過該 Node。

即使目標是原本的 Head，它仍只是 Dummy 後方的一般 Node。

#### Dummy 的生命週期

若 Dummy 是函式區域變數，可以安全回傳：

```cpp
return dummy.next;
```

但不能回傳：

```cpp
return &dummy;
```

函式結束後，區域 Dummy 已不存在。回傳的應是真實 List Head，而不是 Dummy 自身的位址。

### 7.5 完整案例：移除指定值

#### 問題規格

移除 Linked List 中所有 `value == target` 的 Node，並回傳新的 Head。

```text
輸入：1 → 2 → 2 → 3，target = 2
輸出：1 → 3
```

#### Ownership 約定

先明確區分兩種介面：

- 只重新鏈結，不負責釋放被移除 Node。
- 函式擁有 Node，移除時也負責 `delete`。

以下第一個版本只示範鏈結演算法，不釋放 Node。

#### C++ 鏈結版本

```cpp
ListNode* removeValue(
    ListNode* head,
    int target)
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

#### 為什麼檢查 `current->next`

`current` 代表目前保留結果的最後一個 Node。若下一個 Node 要刪除，必須由前驅修改鏈結：

```cpp
current->next = current->next->next;
```

#### Loop Invariant

每輪開始前：

1. Dummy 到 `current` 的所有真實 Node 都不等於 `target`。
2. 這一段恰好是原 List 已處理部分中應保留的 Node，且相對順序不變。
3. `current->next` 是下一個尚待判斷的 Node。

#### 刪除後不能立即前進

若 `current->next` 被移除，新的 `current->next` 尚未檢查，因此 `current` 應保持不動。

例如：

```text
1 → 2 → 2 → 3
```

刪除第一個 2 後，`current->next` 又是 2。若同時令 `current = current->next`，就可能漏掉連續目標。

#### 終止性

每輪都會發生其中一種情況：

- 刪除 `current->next`，未處理 Node 數量減少 1。
- `current` 向下一個 Node 前進。

所以演算法會在有限步驟後到達 List 結尾。

#### 負責釋放的版本

若函式確實擁有所有 Node，可以保存被刪除位置再釋放：

```cpp
ListNode* removeValueAndDelete(
    ListNode* head,
    int target)
{
    ListNode dummy{0, head};
    ListNode* current = &dummy;

    while (current->next != nullptr)
    {
        if (current->next->value == target)
        {
            ListNode* removed = current->next;
            current->next = removed->next;
            delete removed;
        }
        else
        {
            current = current->next;
        }
    }

    return dummy.next;
}
```

只有在 Ownership 約定允許時才能 `delete`。若 Node 由其他物件、Pool 或測試框架管理，擅自釋放可能造成 Double Free 或懸空 Pointer。

#### 邊界案例

<table>
<tr><th>輸入</th><th>Target</th><th>輸出</th><th>目的</th></tr>
<tr><td>空 List</td><td>1</td><td>空 List</td><td>Dummy 的空輸入行為</td></tr>
<tr><td>1</td><td>1</td><td>空 List</td><td>刪除唯一 Node</td></tr>
<tr><td>1 → 2</td><td>1</td><td>2</td><td>Head 改變</td></tr>
<tr><td>1 → 2 → 2</td><td>2</td><td>1</td><td>連續與尾端刪除</td></tr>
<tr><td>2 → 2 → 2</td><td>2</td><td>空 List</td><td>全部刪除</td></tr>
<tr><td>1 → 3</td><td>2</td><td>1 → 3</td><td>沒有目標</td></tr>
</table>

### 7.6 第四步：反轉 Linked List

#### 問題規格

將：

```text
1 → 2 → 3 → null
```

改成：

```text
3 → 2 → 1 → null
```

所有原 Node 都保留，只改變 `next` 方向並回傳新的 Head。

#### C++ 解法

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

#### 三個 Pointer 的角色

- `previous`：已反轉 Prefix 的 Head。
- `current`：下一個要處理的 Node。
- `next`：改寫前保存的剩餘 List Head。

#### Loop Invariant

每輪開始前：

1. `previous` 指向已反轉的原始 Prefix。
2. `current` 指向尚未處理 Suffix。
3. 已反轉區段和未處理區段合起來，恰好包含全部原始 Node，沒有遺失或重複。
4. 已反轉區段中的鏈結方向已完成，未處理區段仍保持原方向。

#### Initialization

開始時：

```text
previous = null
current = head
```

已反轉 Prefix 為空，全部 Node 都在未處理 Suffix，Invariant 成立。

#### Maintenance

每輪：

1. 保存 `current->next`。
2. 讓 `current->next` 指向 `previous`。
3. 將 `previous` 移到 `current`。
4. 將 `current` 移到保存的 `next`。

這會把未處理區段的第一個 Node 移到已反轉區段前方。

#### Termination

當 `current == nullptr` 時，未處理 Suffix 為空。由 Invariant 可得：

- `previous` 包含所有原始 Node。
- 全部鏈結方向已反轉。

因此新的 Head 是 `previous`。

#### 逐輪追蹤

```text
初始：previous = null，current = 1 → 2 → 3
第 1 輪：previous = 1 → null，current = 2 → 3
第 2 輪：previous = 2 → 1，current = 3
第 3 輪：previous = 3 → 2 → 1，current = null
```

#### 常見錯誤

- 未先保存 `current->next`，導致剩餘 List 遺失。
- 最後回傳 `head`，但原 Head 已變成 Tail。
- 忘記讓原 Head 的 `next` 最終指向 `nullptr`。
- 只改變區域 Pointer，沒有正確改寫 Node 的 `next`。

### 7.7 完整案例：合併兩條排序 List

#### 問題規格

給定兩條依非遞減順序排列的 Linked List，重新鏈結成一條排序 List。

```text
a：1 → 4 → 7
b：2 → 3 → 8
結果：1 → 2 → 3 → 4 → 7 → 8
```

#### Precondition

- `a` 與 `b` 各自已排序。
- 本版本假設兩條輸入 List 不共享 Node。
- 函式重新使用原 Node，不配置新資料 Node。

#### C++ 解法

```cpp
ListNode* mergeSortedLists(
    ListNode* a,
    ListNode* b)
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

#### State 意義

- `dummy.next`：合併結果的 Head。
- `tail`：已完成結果的最後一個 Node。
- `a`、`b`：兩條尚未合併 Suffix 的 Head。

#### Loop Invariant

每輪開始前：

1. `dummy.next` 到 `tail` 已排序。
2. 已完成區段恰好包含兩條輸入中已取出的 Node。
3. `a` 與 `b` 分別指向尚未處理區段。
4. 若尚未處理 Node 存在，其 Value 都不小於已完成區段最後一個 Value。

#### 為什麼取較小 Head

因為 `a` 與 `b` 各自排序，兩條 List 尚未處理部分的最小值必定位於 `a` 或 `b`。選擇較小者附加到 `tail`，不會遺漏更小候選。

#### 每輪必須前進

選擇 `a` 時必須：

```cpp
a = a->next;
```

選擇 `b` 時必須：

```cpp
b = b->next;
```

之後 `tail` 也必須移到新加入 Node。若其中一個 Pointer 未前進，可能重複接上相同 Node，形成 Cycle 或無窮迴圈。

#### 接上剩餘區段

當其中一條 List 為空時，另一條剩餘區段本身已排序，而且所有值都不小於已完成 Tail，因此可以一次接上，不必逐 Node 處理。

#### 相同值的選擇

程式使用：

```cpp
if (a->value <= b->value)
```

相等時先取 `a`。若需要保留來源間的穩定順序，這項規則應明確記錄。若題目只要求排序，先取哪一邊通常都能得到合法結果。

#### 複雜度

若兩條 List 長度分別是 `m` 與 `n`：

- 時間複雜度：O(m + n)。
- 額外演算法空間：O(1)。

Dummy Node 位於 Stack，不屬於輸出 List，也不會讓額外空間隨輸入增加。

### 7.8 第五步：使用快慢指標

快慢指標是在同一條鏈結上，以不同速度前進的兩個 Pointer。

常見用途：

- 找中點。
- Cycle Detection。
- 找倒數第 k 個 Node。
- 將 List 分成前後兩段。

#### 找中點

```cpp
ListNode* middleNode(ListNode* head)
{
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast != nullptr &&
           fast->next != nullptr)
    {
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;
}
```

每輪 Slow 前進一步，Fast 前進兩步。Fast 到達結尾時，Slow 約走完一半。

#### 偶數長度必須定義中點

對：

```text
1 → 2 → 3 → 4
```

中間有 2 與 3。上述版本回傳右中點 3。

若題目要求左中點 2，初值或停止條件需要調整。不能只寫「找中點」而不定義偶數長度答案。

#### Dereference 順序

在執行：

```cpp
fast->next->next
```

前必須確認：

```cpp
fast != nullptr && fast->next != nullptr
```

C++ 的 `&&` 由左至右進行 Short-circuit。若第一個條件為 false，不會讀取 `fast->next`。

### 7.9 完整案例：Cycle Detection

#### 問題規格

判斷從 Head 沿 `next` 前進時，是否會再次到達先前經過的同一個 Node。

```text
1 → 2 → 3 → 4
        ↑       ↓
        └───────┘
```

#### C++ 解法

```cpp
bool hasCycle(ListNode* head)
{
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast != nullptr &&
           fast->next != nullptr)
    {
        slow = slow->next;
        fast = fast->next->next;

        if (slow == fast)
        {
            return true;
        }
    }

    return false;
}
```

#### 為什麼無 Cycle 時會結束

若 List 無 Cycle，沿 `next` 前進最終會到達 `nullptr`。Fast 每輪前進兩步，因此最後會出現：

- `fast == nullptr`，或
- `fast->next == nullptr`。

此時可判定沒有 Cycle。

#### 為什麼有 Cycle 時會相遇

當 Slow 與 Fast 都進入 Cycle 後，可只看它們在 Cycle 中的相對位置。每輪：

- Slow 前進 1。
- Fast 前進 2。
- Fast 相對 Slow 多前進 1。

Cycle 長度有限，因此相對距離會在有限輪數內變成 0，兩個 Pointer 指向同一個 Node。

#### 必須比較 Pointer Identity

錯誤方式：

```cpp
slow->value == fast->value
```

非 Cycle List 也可能有重複值。Cycle 的定義是再次到達同一個 Node，不是看到相同 Value。

正確方式：

```cpp
slow == fast
```

#### 為什麼不在移動前立即比較

Slow 與 Fast 都從 `head` 開始。若進入迴圈前直接比較，它們在尚未移動時就相等，所有非空 List 都會被誤判為有 Cycle。

可以選擇不同初值或使用 `do-while`，但比較時間點必須和初值保持一致。

#### 邊界案例

<table>
<tr><th>輸入</th><th>答案</th><th>目的</th></tr>
<tr><td>空 List</td><td>false</td><td>沒有 Node</td></tr>
<tr><td>單一 Node，next 為 null</td><td>false</td><td>最小無 Cycle</td></tr>
<tr><td>單一 Node，next 指向自己</td><td>true</td><td>Cycle 長度 1</td></tr>
<tr><td>兩個相同 Value 的不同 Node</td><td>false</td><td>區分 Value 與 Identity</td></tr>
<tr><td>Tail 指回 Head</td><td>true</td><td>整條 List 為 Cycle</td></tr>
<tr><td>Tail 指回中間 Node</td><td>true</td><td>前綴加 Cycle</td></tr>
</table>

### 7.10 第六步：定義 Ownership 與釋放責任

Pointer 演算法正確，不代表記憶體管理已正確。需要另外回答：

- Node 由誰建立？
- Node 由誰釋放？
- 函式是否取得 Ownership？
- 被移除 Node 是否立即銷毀？
- 輸入 List 是否允許共享 Node？
- 回傳後舊 Head 是否仍可使用？

#### Raw Pointer 不表示 Ownership

```cpp
ListNode* head
```

只表示可透過 Pointer 存取 Node，沒有自動說明誰負責 `delete`。Ownership 必須由介面文件、類別設計或 Smart Pointer 型別表達。

#### 常見風險

- 移除鏈結後沒有釋放，造成 Memory Leak。
- 同一 Node 被多個 Owner 重複釋放。
- 釋放 Node 後仍使用 Pointer，造成 Use-after-free。
- List 具有 Cycle，單純沿 `next` 釋放會無法結束。
- 兩條 List 共享 Tail，分別釋放會造成 Double Free。

#### `unique_ptr` 的取捨

若每個 Node 唯一擁有下一個 Node，可以考慮：

```cpp
#include <memory>

struct OwnedListNode
{
    int value;
    std::unique_ptr<OwnedListNode> next;
};
```

這能表達單一 Ownership，List 銷毀時會遞迴釋放後方 Node。但鏈結轉移需要使用 `std::move`，演算法寫法和 Raw Pointer 版本不同。

此外，具有 Cycle 的結構不能直接用單向 `unique_ptr` 形成循環 Ownership。資料模型與 Ownership 模型必須相容。

#### 演算法題與正式專案

演算法題常由平台建立與回收 Node，函式只需重新鏈結。正式專案則必須在介面中明確說明 Ownership。不要把平台題目的記憶體假設直接帶入其他系統。

### 7.11 第七步：系統化 Debug Pointer

Linked List 錯誤適合以「第一個錯誤鏈結狀態」定位，而不是只看最後輸出。

#### 每輪記錄 Pointer

反轉時記錄：

```text
iteration
previous 的位址與 Value
current 的位址與 Value
next 的位址與 Value
current->next 改寫前指向
current->next 改寫後指向
```

Merge 時記錄：

```text
a
b
tail
tail->next
本輪選擇哪一側
選擇後哪個輸入 Pointer 前進
```

#### 使用 Node 編號而非只看 Value

若 List 有重複值，只印出 Value 會無法區分 Node。可以在測試資料中為每個 Node 暫時編號：

```text
A(value=2) → B(value=2) → C(value=3)
```

這有助於判斷：

- 是否重複接上同一 Node。
- 是否遺失某個 Node。
- 是否意外形成 Cycle。

#### 檢查可達 Node 集合

對重新鏈結演算法，可在小型測試中驗證：

- 輸出可達 Node 數量是否正確。
- 每個預期保留 Node 是否恰好出現一次。
- Tail 的 `next` 是否為 `nullptr`，除非題目允許 Cycle。
- 走訪步數是否超過預期 Node 數，若超過可能形成 Cycle。

#### 最小失敗案例

Pointer Bug 通常能縮小到很短的 List：

- 空 List。
- 一個 Node。
- 兩個 Node。
- 三個 Node。
- 連續兩個待刪除 Node。
- 兩條各有一個 Node 的 Merge。
- Cycle 長度 1 或 2。

短 List 可以完整畫出每輪鏈結，通常比大型輸入更容易定位。

### 7.12 C 語言中的 Linked List

C 的 Node 結構和 Raw Pointer C++ 版本相近：

```c
struct ListNode
{
    int value;
    struct ListNode *next;
};
```

#### 配置 Node

```c
#include <stdlib.h>

struct ListNode *create_node(int value)
{
    struct ListNode *node =
        malloc(sizeof(struct ListNode));

    if (node == NULL)
    {
        return NULL;
    }

    node->value = value;
    node->next = NULL;
    return node;
}
```

呼叫端需要處理配置失敗，並明確決定成功配置 Node 的 Ownership。

#### 釋放無 Cycle List

```c
void destroy_list(struct ListNode *head)
{
    while (head != NULL)
    {
        struct ListNode *next = head->next;
        free(head);
        head = next;
    }
}
```

釋放目前 Node 前必須先保存 `next`，因為 `free(head)` 後不能再讀取 `head->next`。

此函式的 Precondition 是 List 無 Cycle，而且每個 Node 可由此函式安全釋放一次。

#### 修改 Head 的介面

C 可以回傳新 Head：

```c
struct ListNode *reverse_list(
    struct ListNode *head);
```

也可以接收 Pointer to Pointer：

```c
void reverse_list(
    struct ListNode **head);
```

第二種方式直接修改呼叫端的 Head，但需要明確檢查 `head != NULL`。兩者演算法核心相同，介面責任不同。

### 7.13 建立自己的 Linked List 分析表

<table>
<tr><th>欄位</th><th>要回答的問題</th></tr>
<tr><td>List 類型</td><td>Singly、Doubly，還是 Circular Linked List？</td></tr>
<tr><td>Head</td><td>是否可能為空？演算法會不會產生新 Head？</td></tr>
<tr><td>Tail</td><td>是否已知 Tail？結果是否要求 Tail 指向 null？</td></tr>
<tr><td>Node Identity</td><td>需要比較 Value，還是比較是否為同一個 Node？</td></tr>
<tr><td>排序條件</td><td>輸入是否已排序？排序是必要 Precondition 嗎？</td></tr>
<tr><td>Pointer State</td><td>每個 Pointer 代表前驅、目前、下一個，還是結果 Tail？</td></tr>
<tr><td>已處理區段</td><td>已完成的 Prefix 或結果 List 保存什麼？</td></tr>
<tr><td>未處理區段</td><td>哪個 Pointer 是剩餘 List 的入口？</td></tr>
<tr><td>更新順序</td><td>改寫 `next` 前需要保存哪些位置？</td></tr>
<tr><td>前進條件</td><td>每輪哪個 Pointer 或未處理 Node 數量會前進？</td></tr>
<tr><td>Dummy Node</td><td>Head 是否可能插入、刪除或替換？</td></tr>
<tr><td>共享結構</td><td>兩條輸入 List 是否可能共享 Node 或 Tail？</td></tr>
<tr><td>Cycle</td><td>輸入是否保證無 Cycle？走訪如何終止？</td></tr>
<tr><td>Ownership</td><td>誰建立、持有與釋放 Node？</td></tr>
<tr><td>邊界案例</td><td>空、一個 Node、兩個 Node、全部刪除與 Cycle 長度 1 如何處理？</td></tr>
</table>

### 7.14 常見問題與判讀

<table>
<tr><th>現象</th><th>可能原因</th><th>第一輪檢查</th></tr>
<tr><td>後半 List 消失</td><td>改寫 `next` 前未保存剩餘入口</td><td>先保存 `current->next`</td></tr>
<tr><td>反轉後只剩第一個 Node</td><td>沿著已改寫的 `next` 前進</td><td>確認 `current = next` 使用暫存位置</td></tr>
<tr><td>回傳結果仍從舊 Head 開始</td><td>Head 已改變但回傳舊值</td><td>反轉後應回傳 `previous`</td></tr>
<tr><td>刪除 Head 時失敗</td><td>沒有更新 Head 或缺少 Dummy</td><td>確認回傳 `dummy.next`</td></tr>
<tr><td>連續目標只刪除一個</td><td>刪除後仍讓前驅前進</td><td>新 `current->next` 是否尚未檢查</td></tr>
<tr><td>Merge 無窮迴圈</td><td>選中來源的 Pointer 未前進</td><td>每輪確認 `a` 或 `b` 至少一個前進</td></tr>
<tr><td>Merge 產生 Cycle</td><td>重複接上同一 Node 或輸入共享 Node</td><td>檢查 Node Identity 與輸入 Precondition</td></tr>
<tr><td>Cycle Detection 誤判重複值</td><td>比較 Value 而非 Pointer</td><td>應使用 `slow == fast`</td></tr>
<tr><td>所有非空 List 都被判定有 Cycle</td><td>快慢指標尚未移動便比較</td><td>檢查初值與比較時間點</td></tr>
<tr><td>找中點在偶數長度與預期不同</td><td>左中點、右中點未定義</td><td>同步初值、停止條件與 Postcondition</td></tr>
<tr><td>偶爾 Null Dereference</td><td>先讀取 `fast->next` 才檢查 `fast`</td><td>條件順序是否安全</td></tr>
<tr><td>Memory Leak</td><td>移除鏈結後沒有依 Ownership 釋放</td><td>確認被移除 Node 的 Owner</td></tr>
<tr><td>Double Free</td><td>共享 Node 被多次視為獨立 Ownership</td><td>檢查 List 是否共享 Tail</td></tr>
<tr><td>Use-after-free</td><td>釋放後仍讀取 Node</td><td>釋放前保存必要 `next`，釋放後停止使用舊 Pointer</td></tr>
<tr><td>釋放 List 無法結束</td><td>輸入包含 Cycle</td><td>釋放函式是否以無 Cycle 為 Precondition</td></tr>
</table>

### 7.15 本章檢查表

- 我能說明 Node Value、`next` 與 Head 的角色。
- 我知道 Linked List Node 不需要位於連續記憶體。
- 我能區分 Node Value 相同與 Pointer Identity 相同。
- 我知道 Linked List 不支援 O(1) 的任意 Index 存取。
- 我能說明已知前驅時插入與刪除為何可在 O(1) 完成。
- 我不會忽略尋找前驅位置所需的 O(n) 成本。
- 我會在改寫 `next` 前保存之後仍需要的 Pointer。
- 我能畫出更新前與更新後的局部鏈結。
- 我知道 Pointer Assignment 不會複製 Node。
- 我能使用 Dummy Node 統一 Head 修改情況。
- 我不會回傳區域 Dummy 自身的位址。
- 我能為移除指定值寫出已保留區段的 Invariant。
- 我知道刪除後為何不能無條件讓前驅前進。
- 我能說明反轉中的 `previous`、`current` 與 `next`。
- 我能證明反轉過程沒有遺失或重複 Node。
- 我能使用 Dummy 與 Tail 合併兩條排序 List。
- 我會確認 Merge 每輪都有輸入 Pointer 前進。
- 我知道偶數長度的中點必須明確定義。
- 我會在讀取 `fast->next->next` 前完成 Null 檢查。
- 我能說明快慢指標在 Cycle 中為何一定相遇。
- 我會使用 Pointer Identity，而不是 Value，判斷相遇。
- 我會將鏈結演算法正確性與 Ownership 分開分析。
- 我知道 Raw Pointer 本身不表示誰負責釋放。
- 我會為配置、移除與銷毀 Node 明確定義責任。
- 我能以空 List、一個 Node、兩個 Node 與短 Cycle 建立測試。
- 我會記錄第一個錯誤 Pointer State，而不只觀察最終輸出。

### 7.16 本章重點

- Linked List 由分散配置的 Node 與 Pointer 鏈結構成，Head 是第一個 Node 的入口。
- Node Value 相同不代表是同一個 Node；Cycle Detection 需要比較 Pointer Identity。
- Linked List 不支援 O(1) 任意 Index 存取，尋找位置通常需要沿 `next` 走訪。
- 已知前驅時，局部插入或刪除只需修改固定數量鏈結。
- Pointer 更新前必須保存之後仍需使用的剩餘 List 入口。
- Dummy Node 讓原 Head 也擁有前驅，可統一插入、刪除與合併邊界。
- 移除 Node 後，是否讓前驅前進取決於新的下一個 Node 是否已檢查。
- 反轉演算法將 List 分成已反轉 Prefix 與未處理 Suffix，`next` 必須在改寫前保存。
- 合併排序 List 時，結果 Tail 與被選中的輸入 Pointer都必須前進。
- 快慢指標可用於中點與 Cycle，但初值、速度與停止條件必須和答案定義一致。
- 偶數長度的左中點與右中點是不同 Postcondition。
- 有 Cycle 時，Fast 相對 Slow 每輪多前進一步，因此會在有限 Cycle 中相遇。
- 演算法重新鏈結正確，不代表 Allocation、Ownership 與 Deallocation 已正確。
- Raw Pointer 不表達 Ownership，正式介面必須說明誰建立與釋放 Node。
- Debug Linked List 時，應記錄 Node Identity、Pointer 更新前後與第一個被破壞的鏈結。
- 空 List、單一 Node、兩個 Node、連續刪除與短 Cycle 是最重要的最小測試案例。
