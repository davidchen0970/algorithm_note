## 第 23 章　Tree 基礎

### 適用範圍

本章介紹 Tree 的基本資料模型，包括 Node、Edge、Root、Parent、Child、Leaf、Depth、Height、Level、Subtree，以及 Binary Tree 的常見類型與 Array、Pointer 表示方式。

Tree 題目的困難通常不是走訪語法，而是名詞與狀態沒有先定義清楚，例如：空 Tree 的 Height 是多少、Root 的 Depth 從 0 還是 1 開始、Height 以 Edge 數還是 Node 數計算、題目給的是一般 Tree、Binary Tree 或 Binary Search Tree，以及遞迴函式回傳的是目前 Node 的答案還是整棵 Subtree 的摘要。原始章節也列出這些 Tree 題常見混淆點。citeturn43search1

本章會建立一套固定流程：

1. 先定義 Tree 類型與空 Tree 行為。
2. 明確區分 Node Identity 與 Node Value。
3. 定義 Root、Parent、Child、Sibling 與 Leaf。
4. 統一 Depth、Level 與 Height 的計算方式。
5. 將 Tree 問題拆成「目前 Node」與「Child Subtree」。
6. 為遞迴寫出 Base Case、Subproblem 與 Combine。
7. 分別計算 Node 走訪成本與 Call Stack 空間。
8. 以空 Tree、單一 Node、鏈狀 Tree、完整 Tree 與重複值測試。

```mermaid
flowchart TD
    A["Tree 題目"] --> B["確認 Tree 類型與表示方式"]
    B --> C["定義空 Tree、Depth、Height"]
    C --> D["定義遞迴函式回傳語意"]
    D --> E["處理 Base Case"]
    E --> F["取得 Child Subtree 結果"]
    F --> G["Combine 成目前 Node 答案"]
    G --> H["分析時間與 Stack 空間"]
```

### 適用讀者

- 第一次系統化學習 Tree 的讀者。
- 會寫遞迴，但不清楚函式回傳值語意的讀者。
- 容易混淆 Depth、Height 與 Level 的讀者。
- 把 Binary Tree 和 Binary Search Tree 視為相同結構的讀者。
- 需要理解 Pointer Tree 與 Array Tree 表示差異的讀者。
- 想為後續 DFS、BFS、BST、Heap 與 Tree DP 建立基礎的讀者。

### 快速導覽

- [23.1 Tree 到底是什麼](#231-tree-到底是什麼)
- [23.2 Node、Edge 與 Root](#232-nodeedge-與-root)
- [23.3 Parent、Child、Sibling 與 Leaf](#233-parentchildsibling-與-leaf)
- [23.4 Depth、Level 與 Height](#234-depthlevel-與-height)
- [23.5 Subtree 與遞迴模型](#235-subtree-與遞迴模型)
- [23.6 Binary Tree 的常見類型](#236-binary-tree-的常見類型)
- [23.7 Pointer 表示](#237-pointer-表示)
- [23.8 Array 表示](#238-array-表示)
- [23.9 完整案例：計算 Node 數量](#239-完整案例計算-node-數量)
- [23.10 完整案例：計算 Tree Height](#2310-完整案例計算-tree-height)
- [23.11 DFS 與 BFS 的基本差異](#2311-dfs-與-bfs-的基本差異)
- [23.12 Tree 正確性與複雜度](#2312-tree-正確性與複雜度)
- [23.13 Ownership 與生命週期](#2313-ownership-與生命週期)
- [23.14 C 語言中的 Tree](#2314-c-語言中的-tree)
- [23.15 系統化 Debug](#2315-系統化-debug)
- [23.16 建立自己的 Tree 分析表](#2316-建立自己的-tree-分析表)
- [23.17 常見問題與判讀](#2317-常見問題與判讀)
- [23.18 練習題方向](#2318-練習題方向)
- [23.19 本章檢查表](#2319-本章檢查表)
- [23.20 本章重點](#2320-本章重點)

### 23.1 Tree 到底是什麼

Tree 是由 Node 與 Edge 組成的階層結構。Rooted Tree 會指定一個 Root，其他 Node 可依 Root 方向形成 Parent 與 Child 關係。

```mermaid
flowchart TD
    A["Root A"] --> B["Node B"]
    A --> C["Node C"]
    B --> D["Node D"]
    B --> E["Node E"]
    C --> F["Node F"]
```

在一般有限 Tree 中：

- 任意兩個 Node 之間有唯一簡單 Path。
- 沒有 Cycle。
- 若有 n 個 Node，通常有 n - 1 條 Edge。

如果結構中存在 Cycle，或某個 Child 同時由多個 Parent 指向，它可能是 Graph 或 DAG，而不是本章假設的 Tree。原始章節也以這些條件區分 Tree 與 Graph / DAG。citeturn43search1

#### Tree 的核心直覺

Tree 的特性是「分支」與「不重複路徑」。從 Root 往下走，每個 Child Subtree 都是一個更小的 Tree。這讓 Tree 題非常適合遞迴：處理目前 Node，並把左右或多個 Child 當成較小的子問題。

#### 什麼不是 Tree

以下情況要特別小心：

| 情況 | 可能結構 |
|---|---|
| 某個 Node 有兩個 Parent | DAG 或 Graph |
| 存在 A -> B -> C -> A | Graph，有 Cycle |
| 沒有指定 Root | 可能是 Unrooted Tree，需要先選 Root |
| Edge 有方向且形成依賴 | 可能是 DAG |
| Node 可被共享 | 可能不是 Tree Ownership 結構 |

### 23.2 Node、Edge 與 Root

#### Node

Node 保存資料與連接關係。資料可以是 Value、Key、狀態或物件。

例如 Binary Tree Node 常包含：

```cpp
struct TreeNode
{
    int value;
    TreeNode* left;
    TreeNode* right;
};
```

#### Edge

Edge 連接兩個 Node。在 Rooted Tree 中，通常由 Parent 指向 Child。

#### Root

Root 是沒有 Parent 的起點。非空 Rooted Tree 恰好有一個 Root。

```mermaid
flowchart TD
    R["Root\n沒有 Parent"] --> A["Child A"]
    R --> B["Child B"]
    A --> C["Descendant C"]
```

空 Tree 沒有 Root，通常使用 `nullptr` 或空容器表示。原始章節也指出，空 Tree 沒有 Root，常用 `nullptr` 或空容器表示。citeturn43search1

#### Node Identity 與 Value

兩個 Node 可以保存相同 Value，但仍是不同 Node：

```mermaid
flowchart LR
    A["Node A\nvalue 5"] --> B["Node B\nvalue 5"]
```

比較 Value 與比較 Node Identity 是不同問題。判斷 Tree 結構、Ancestor 或共享 Node 時，通常需要 Identity。原始章節也提醒，相同 Value 不代表相同 Node。citeturn43search1

##### 例子

若題目問：

```text
兩棵 Tree 是否相同？
```

通常要比較結構與 Value。

若題目問：

```text
某個 Node 是否為另一個 Node 的 Ancestor？
```

通常要比較 Node Identity，而不是 Value。因為 Tree 中可能有多個 Value 相同的節點。

### 23.3 Parent、Child、Sibling 與 Leaf

若 Edge 由 A 指向 B：

- A 是 B 的 Parent。
- B 是 A 的 Child。

具有相同 Parent 的 Node 稱為 Sibling。沒有 Child 的 Node 稱為 Leaf。

```mermaid
flowchart TD
    P["Parent"] --> A["Child A\nSibling"]
    P --> B["Child B\nSibling"]
    A --> L["Leaf"]
```

#### Ancestor 與 Descendant

沿 Parent 方向可到達的 Node 是 Ancestor；沿 Child 方向可到達的 Node 是 Descendant。

Node 是否視為自己的 Ancestor，需依題目定義，常見做法是另外區分 Proper Ancestor。原始章節也提到，是否把自己算作 Ancestor 需依題目定義。citeturn43search1

#### Degree

Node 的 Degree 可表示 Child 數量。Binary Tree 中每個 Node 最多有兩個 Child。

#### Leaf 的常見用法

Leaf 常作為 Base Case 或答案來源，例如：

- 計算 Root-to-Leaf Path Sum。
- 找最小深度。
- 收集所有 Leaf Value。
- 判斷 Tree 是否 Balanced。

判斷 Leaf：

```cpp
bool isLeaf(const TreeNode* node)
{
    return node != nullptr &&
           node->left == nullptr &&
           node->right == nullptr;
}
```

### 23.4 Depth、Level 與 Height

本章採用 Edge 數定義：

- Root Depth = 0。
- Child Depth = Parent Depth + 1。
- Node Height = 從該 Node 到最深 Leaf 的最大 Edge 數。
- Leaf Height = 0。
- 非空 Tree Height = Root Height。
- 空 Tree Height = -1。

```mermaid
flowchart TD
    A["Root\nDepth 0\nHeight 2"] --> B["Depth 1\nHeight 1"]
    A --> C["Depth 1\nHeight 0"]
    B --> D["Leaf\nDepth 2\nHeight 0"]
```

原始章節也採用 Edge 數定義，並設定空 Tree Height = -1、Leaf Height = 0。citeturn43search1

#### Depth 與 Height 的方向

```text
Depth：從 Root 往目前 Node
Height：從目前 Node 往最深 Leaf
```

Depth 通常由上往下傳遞；Height 通常由下往上組合 Child 結果。

#### Level

Level 有時等同 Depth，有時從 1 開始。題目只寫 Level 時，應先確認定義，不要自行假設。

例如兩種常見定義：

| 名稱 | Root 值 | Child 值 |
|---|---:|---:|
| Depth | 0 | 1 |
| Level，1-based | 1 | 2 |

#### Height 用 Node 數定義時

有些教材以 Node 數定義 Height，這時：

- 空 Tree Height = 0。
- Leaf Height = 1。

兩種定義都可使用，但程式、公式與測試必須保持一致。原始章節也提醒，不可混用 Edge 數與 Node 數定義。citeturn43search1

### 23.5 Subtree 與遞迴模型

以某個 Node 為 Root，包含它與所有 Descendant 的結構稱為 Subtree。

```mermaid
flowchart TD
    A["整棵 Tree Root"] --> B["Subtree Root B"]
    A --> C["另一個 Subtree"]
    B --> D["Node D"]
    B --> E["Node E"]
```

Tree 遞迴的核心模型是：

> 假設每個 Child Subtree 的答案已正確取得，目前 Node 如何組合成自己的答案？

典型結構：

```cpp
Result solve(TreeNode* node)
{
    if (node == nullptr)
    {
        return baseResult;
    }

    Result left = solve(node->left);
    Result right = solve(node->right);

    return combine(node, left, right);
}
```

需要明確定義：

- `Result` 代表什麼。
- 空 Subtree 回傳什麼 Identity 或 Sentinel。
- `combine` 如何使用目前 Node 與 Child Result。

原始章節也以同樣模型說明 Tree 遞迴，並強調要定義 `Result`、Base Case 與 Combine。citeturn43search1

#### 常見 Result 設計

| 問題 | Result 可能代表 |
|---|---|
| 計算 Node 數 | Subtree 的 Node 數量 |
| 計算高度 | Subtree 的 Height |
| 判斷是否 Balanced | Height + 是否 Balanced |
| 最大 Path Sum | 往 Parent 可延伸的最大值 + 全域答案 |
| Tree DP | 多個狀態，例如選或不選目前 Node |

### 23.6 Binary Tree 的常見類型

#### Binary Tree

每個 Node 最多有 Left Child 與 Right Child。沒有排序保證。

#### Full Binary Tree

每個 Node 的 Child 數是 0 或 2。

#### Complete Binary Tree

除了最後一層外，其餘層填滿；最後一層由左至右填入。Heap 常使用此形狀。

#### Perfect Binary Tree

所有內部 Node 都有兩個 Child，而且所有 Leaf 位於相同 Depth。

#### Balanced Binary Tree

通常表示 Tree Height 沒有過度偏斜，但精確條件依資料結構而異。例如 AVL Tree 對每個 Node 的左右 Subtree Height 差有明確限制。

#### Binary Search Tree

除 Binary Tree 結構外，還具備 Key 排序規則。重複 Key 放置政策必須另行定義。

```mermaid
flowchart TD
    B["Binary Tree"] --> F["Full"]
    B --> C["Complete"]
    B --> P["Perfect"]
    B --> A["Balanced"]
    B --> S["Binary Search Tree\n額外具備 Key 順序"]
```

這些名詞描述不同性質，不是互斥分類。一棵 Tree 可以同時是 Full、Complete 與 Perfect。原始章節也特別提醒 Binary Tree 本身沒有排序保證，BST 才有 Key 排序規則。citeturn43search1

#### Binary Tree 不等於 BST

若題目只說 Binary Tree，就不能假設：

```text
left value < node value < right value
```

除非題目明確說它是 Binary Search Tree。

### 23.7 Pointer 表示

Pointer 表示常見於 Binary Tree：

```cpp
struct TreeNode
{
    int value;
    TreeNode* left;
    TreeNode* right;
};
```

```mermaid
flowchart TD
    A["TreeNode A"] -->|"left"| B["TreeNode B"]
    A -->|"right"| C["TreeNode C"]
    B -->|"left"| N1["nullptr"]
    B -->|"right"| N2["nullptr"]
```

Pointer 表示適合：

- Tree 形狀不完整。
- 經常新增或移除 Node。
- Node 還包含其他 State。

缺點包括：

- 每個 Node 需要 Child Pointer。
- Node 可能分散配置。
- Ownership 與釋放責任需另行管理。

原始章節也列出 Pointer 表示的適用情境與缺點。citeturn43search1

#### nullptr 的角色

`nullptr` 通常代表空 Subtree。Tree 遞迴常在一進入函式時先處理：

```cpp
if (node == nullptr)
{
    return baseResult;
}
```

不能先讀 `node->value` 再檢查 `nullptr`。

### 23.8 Array 表示

Complete Binary Tree 常用 Array 儲存。0-based Index 下：

```text
left child = 2 * i + 1
right child = 2 * i + 2
parent = (i - 1) / 2，僅適用 i > 0
```

```mermaid
flowchart TD
    I0["Index 0"] --> I1["Index 1"]
    I0 --> I2["Index 2"]
    I1 --> I3["Index 3"]
    I1 --> I4["Index 4"]
    I2 --> I5["Index 5"]
    I2 --> I6["Index 6"]
```

Array 表示適合 Complete Tree，定位 Parent 與 Child 為 O(1)。若 Tree 高度很大但非常稀疏，保留大量空位置會浪費空間。

計算 `2 * i + 1` 前也要考慮 Index 型別 Overflow，並檢查結果小於 Array Size。原始章節也有相同提醒。citeturn43search1

#### 1-based Index 對照

有些教材或 Heap 實作使用 1-based Index：

```text
left child = 2 * i
right child = 2 * i + 1
parent = i / 2
```

使用哪一種都可以，但程式中要一致。

### 23.9 完整案例：計算 Node 數量

#### Postcondition

回傳以 `node` 為 Root 的 Subtree Node 數量。

```cpp
#include <cstddef>

std::size_t countNodes(const TreeNode* node)
{
    if (node == nullptr)
    {
        return 0;
    }

    return 1 + countNodes(node->left) + countNodes(node->right);
}
```

#### Base Case

空 Subtree 沒有 Node，回傳 0。

#### 遞迴步驟

假設左右 Child 函式分別正確回傳左右 Subtree Node 數量。目前答案為：

```text
目前 Node 1 個 + Left Subtree + Right Subtree
```

```mermaid
flowchart TD
    A["目前 Node"] --> L["計算 Left Subtree 數量"]
    A --> R["計算 Right Subtree 數量"]
    L --> S["1 + left + right"]
    R --> S
```

#### 複雜度

每個 Node 恰好處理一次：

- 時間 O(n)。
- Call Stack 取決於形狀，最差 O(h)。
- 平衡 Tree 的 h 約為 O(log n)，鏈狀 Tree 的 h 可達 O(n)。

原始章節也用同樣案例說明 Node 數量計算與 O(n)、O(h) 分析。citeturn43search1

### 23.10 完整案例：計算 Tree Height

本章採 Edge 數定義，空 Tree Height = -1，Leaf Height = 0。

```cpp
#include <algorithm>

int treeHeight(const TreeNode* node)
{
    if (node == nullptr)
    {
        return -1;
    }

    return 1 + std::max(
        treeHeight(node->left),
        treeHeight(node->right));
}
```

#### 為何空 Tree 回傳 -1

Leaf 的兩個 Child 都是空 Subtree：

```text
1 + max(-1, -1) = 0
```

因此 Leaf Height 自然為 0。

```mermaid
flowchart TD
    L["Leaf"] --> N1["空 Left\nHeight -1"]
    L --> N2["空 Right\nHeight -1"]
    N1 --> H["Leaf Height = 1 + max(-1, -1) = 0"]
    N2 --> H
```

若採 Node 數定義，Base Case 可改為 0，Leaf Height 會是 1。不可只改註解而不改公式與測試。原始章節也用這個例子說明 Base Case 與 Height 定義必須一致。citeturn43search1

#### 常見錯誤

| 錯誤 | 原因 |
|---|---|
| Leaf Height 算成 1 | 混用 Node 數與 Edge 數定義 |
| 空 Tree Crash | 沒有先處理 `node == nullptr` |
| 高度差一 | Base Case 與公式不一致 |

### 23.11 DFS 與 BFS 的基本差異

#### DFS

DFS 優先深入某個 Child Subtree。可用遞迴或顯式 Stack。

常見順序：

- Preorder：Node、Left、Right。
- Inorder：Left、Node、Right。
- Postorder：Left、Right、Node。

#### BFS

BFS 使用 Queue 依 Depth Layer 走訪。

```mermaid
flowchart TD
    A["Depth 0"] --> B["Depth 1"]
    A --> C["Depth 1"]
    B --> D["Depth 2"]
    B --> E["Depth 2"]
    C --> F["Depth 2"]
```

選擇依問題需求：

- 需要 Subtree 結果往上組合：常用 Postorder DFS。
- 需要按層處理或找最淺答案：常用 BFS。
- 需要所有走訪順序：依輸出規格選擇。

原始章節也以這三個方向說明如何選 DFS 或 BFS。citeturn43search1

#### 常見 Tree Traversal 程式骨架

Preorder：

```cpp
void preorder(const TreeNode* node)
{
    if (node == nullptr)
    {
        return;
    }

    visit(node);
    preorder(node->left);
    preorder(node->right);
}
```

Postorder：

```cpp
void postorder(const TreeNode* node)
{
    if (node == nullptr)
    {
        return;
    }

    postorder(node->left);
    postorder(node->right);
    visit(node);
}
```

BFS：

```cpp
#include <queue>

void bfs(const TreeNode* root)
{
    if (root == nullptr)
    {
        return;
    }

    std::queue<const TreeNode*> pending;
    pending.push(root);

    while (!pending.empty())
    {
        const TreeNode* node = pending.front();
        pending.pop();

        visit(node);

        if (node->left != nullptr)
        {
            pending.push(node->left);
        }

        if (node->right != nullptr)
        {
            pending.push(node->right);
        }
    }
}
```

### 23.12 Tree 正確性與複雜度

Tree 遞迴通常使用結構歸納：

1. 空 Tree 或 Leaf 的 Base Case 正確。
2. 假設 Child Subtree 的遞迴結果正確。
3. 證明目前 Node 的 Combine 會產生正確答案。
4. 每次呼叫進入嚴格較小的 Subtree，所以會終止。

```mermaid
flowchart TD
    A["目前 Subtree"] --> L["較小 Left Subtree"]
    A --> R["較小 Right Subtree"]
    L --> C["Combine"]
    R --> C
    C --> O["目前 Subtree 正確答案"]
```

原始章節也以結構歸納說明 Tree 遞迴正確性，並提醒每次呼叫進入嚴格較小的 Subtree，所以會終止。citeturn43search1

#### 複雜度計算

不要只看到兩次遞迴呼叫就判定 O(2^h)。如果 Left 與 Right Subtree 不重疊，每個 Node 只被處理一次，時間通常是 O(n)。原始章節也特別提醒這點。citeturn43search1

空間需包括：

- 明確建立的 Stack、Queue 或結果容器。
- 遞迴 Call Stack O(h)。

其中 h 是 Tree Height。平衡 Tree 中 h 約 O(log n)，鏈狀 Tree 中 h 可達 O(n)。

### 23.13 Ownership 與生命週期

Raw Pointer Tree 不會自動表達 Ownership：

- 誰配置 Node？
- 誰釋放 Node？
- Child 是否可能共享？
- 結構是否保證無 Cycle？

若每個 Node 唯一擁有 Child，可使用：

```cpp
#include <memory>

struct OwnedTreeNode
{
    int value;
    std::unique_ptr<OwnedTreeNode> left;
    std::unique_ptr<OwnedTreeNode> right;
};
```

```mermaid
flowchart TD
    O["Root Owner"] --> L["unique Left Child"]
    O --> R["unique Right Child"]
    L --> LL["unique Descendant"]
```

若使用 Parent Pointer，它通常不應再擁有 Parent，否則可能形成 Ownership Cycle。演算法結構與 Ownership 結構需要分開設計。原始章節也有相同提醒。citeturn43search1

#### Parent Pointer 常見設計

```cpp
struct Node
{
    int value;
    std::unique_ptr<Node> left;
    std::unique_ptr<Node> right;
    Node* parent = nullptr; // non-owning
};
```

`parent` 只是觀察用，不負責釋放 Parent。

### 23.14 C 語言中的 Tree

```c
#include <stdlib.h>

struct TreeNode
{
    int value;
    struct TreeNode *left;
    struct TreeNode *right;
};

struct TreeNode *create_node(int value)
{
    struct TreeNode *node = malloc(sizeof(*node));

    if (node == NULL)
    {
        return NULL;
    }

    node->value = value;
    node->left = NULL;
    node->right = NULL;
    return node;
}
```

釋放無共享、無 Cycle Tree 可使用 Postorder：

```c
void destroy_tree(struct TreeNode *node)
{
    if (node == NULL)
    {
        return;
    }

    destroy_tree(node->left);
    destroy_tree(node->right);
    free(node);
}
```

必須先釋放 Child，再釋放目前 Node。若先 `free(node)`，就不能再安全讀取 Child Pointer。原始章節也明確提醒這個釋放順序。citeturn43search1

#### C 中的錯誤風險

- 忘記初始化 left / right。
- 忘記釋放 Node，造成 Memory Leak。
- 釋放後仍使用 Pointer。
- 多個 Parent 指向同一 Child，導致 Double Free。

### 23.15 系統化 Debug

建議為每次遞迴記錄：

```text
Node Identity
Node Value
Depth
Left Result
Right Result
Combined Result
返回位置
```

```mermaid
flowchart TD
    A["Tree 結果錯誤"] --> B["先確認 Tree 結構與 Root"]
    B --> C["確認空 Tree Base Case"]
    C --> D{"Child Result 語意正確嗎"}
    D -->|否| E["縮小到最小 Subtree"]
    D -->|是| F{"Combine 公式正確嗎"}
    F -->|否| G["修正目前 Node 組合"]
    F -->|是| H["檢查 Depth Height 定義與邊界"]
```

原始章節也建議 Debug 時記錄 Node Identity、Node Value、Depth、Child Result 與 Combined Result，而不是只看最終輸出。citeturn43search1

#### 重要測試

- 空 Tree。
- 單一 Root。
- 只有 Left Child。
- 只有 Right Child。
- 完全鏈狀 Tree。
- Perfect Tree。
- 左右 Height 不同。
- Node Value 全部相同。
- 極深 Tree 的 Stack 深度。

### 23.16 建立自己的 Tree 分析表

| 欄位 | 要回答的問題 |
|---|---|
| Tree 類型 | 一般 Tree、Binary Tree、BST、Complete Tree？ |
| Root | 空 Tree 如何表示？ |
| Node Identity | 相同 Value 的 Node 是否區分？ |
| Child | Child 數量固定或可變？ |
| Depth | Root 從 0 還是 1 開始？ |
| Height | 以 Edge 或 Node 數計算？空 Tree 是多少？ |
| Subproblem | 遞迴函式對一棵 Subtree 回傳什麼？ |
| Base Case | 空 Subtree 或 Leaf 回傳什麼？ |
| Combine | 如何由 Child Result 組合目前答案？ |
| Traversal | Preorder、Inorder、Postorder 或 BFS？ |
| Representation | Pointer、Array、Parent Array 或 Edge List？ |
| Ownership | 誰建立與釋放 Node？ |
| 複雜度 | 每個 Node 處理幾次？最大 Height 多少？ |
| 邊界 | 空、單一 Node、鏈狀與重複值如何處理？ |

原始章節也提供同樣方向的 Tree 分析表，協助在寫程式前固定名詞與狀態。citeturn43search1

### 23.17 常見問題與判讀

| 現象 | 可能原因 | 第一輪檢查 |
|---|---|---|
| Height 差一 | Edge 與 Node 數定義混用 | 空 Tree 與 Leaf Height 是多少 |
| Root Depth 不一致 | Level 從 1、Depth 從 0 混用 | 先固定名詞定義 |
| 把 Binary Tree 當 BST | 誤以為 Left 一定較小 | 題目是否明確提供排序性 |
| 遞迴答案重複或漏算 | Subtree Result 語意不清 | 函式對目前 Node 回傳什麼 |
| 空 Tree Crash | 先讀取 Node 再檢查 Null | Base Case 應先處理 `nullptr` |
| 複雜度誤判 O(2^h) | 只看兩次遞迴呼叫 | Left、Right Subtree 是否重疊 |
| 深 Tree Stack Overflow | Height 接近 n | 改用顯式 Stack 或限制深度 |
| Array Child Index 越界 | 未檢查 `2*i+1` | 先檢查型別與 Size |
| 相同 Value Node 混淆 | 只印 Value | 同時記錄 Node Identity |
| Memory Leak | 未釋放 Node | 明確定義 Ownership 與 Postorder 銷毀 |
| Double Free | Subtree 被多個 Owner 共享 | 確認結構是否真的是 Tree |
| BFS Layer 錯誤 | Queue Size 或 Depth 更新時機錯 | 每層開始時固定該層 Node 數 |

原始章節也列出這些常見問題與第一輪檢查方向。citeturn43search1

### 23.18 練習題方向

#### 基礎題

計算 Tree 的 Node 數、Leaf 數、最大 Depth 與所有 Value 總和。

每題都要寫清楚：

- 遞迴函式語意。
- Base Case。
- Child Result。
- Combine。
- 時間 O(n)。
- 空間 O(h)。

#### 變化題

判斷兩棵 Tree 是否具有相同結構與 Value。先定義空 Tree 與 Node Identity 是否影響答案。

#### 綜合題

輸出每一層的 Node Value，並比較 BFS Queue 與 DFS 傳遞 Depth 的時間、空間與輸出順序。

原始章節也建議從 Node 數、Leaf 數、最大 Depth、Tree 相同與 Level Order 這些方向練習。citeturn43search1

### 23.19 本章檢查表

- 我能說明 Node、Edge 與 Root。
- 我能區分 Parent、Child、Sibling、Leaf、Ancestor 與 Descendant。
- 我知道相同 Value 不代表相同 Node。
- 我能明確定義 Depth、Level 與 Height。
- 我知道 Leaf Height 與空 Tree Height 取決於採用的定義。
- 我能說明 Subtree 為何適合遞迴。
- 我能為遞迴函式寫出精確回傳語意。
- 我能說明 Base Case、Child Subproblem 與 Combine。
- 我能區分 Binary Tree、Complete Tree、Perfect Tree 與 BST。
- 我知道 Binary Tree 本身沒有排序保證。
- 我能比較 Pointer 與 Array 表示。
- 我會檢查 Array Child Index 是否位於範圍內。
- 我能用結構歸納說明 Tree 遞迴正確性。
- 我知道每個 Node 只處理一次時，時間通常為 O(n)。
- 我會把遞迴 Call Stack O(h) 列入空間成本。
- 我能依需求選擇 DFS 或 BFS。
- 我會明確定義 Node Ownership 與釋放責任。
- 我能使用空 Tree、單一 Node、鏈狀與 Perfect Tree 測試。

### 23.20 本章重點

- Tree 是由 Node 與 Edge 構成的階層結構，Root 是沒有 Parent 的起點。
- Node Value 與 Node Identity 是不同概念。
- Parent、Child、Sibling、Leaf、Ancestor 與 Descendant 描述 Node 間的結構關係。
- Depth 從 Root 往下計算，Height 從目前 Node 往最深 Leaf 計算。
- Height 可用 Edge 數或 Node 數定義，但 Base Case、公式與測試必須一致。
- Subtree 讓 Tree 問題自然拆成 Child Subproblem 與目前 Node 的 Combine。
- Binary Tree 只限制 Child 數量，Binary Search Tree 才額外具有 Key 排序規則。
- Pointer 表示適合稀疏或動態形狀，Array 表示適合 Complete Tree。
- Tree 遞迴通常以空 Subtree 為 Base Case，並對嚴格較小的 Child Subtree 呼叫。
- 每個 Node 處理一次時，時間通常為 O(n)，空間還要計入 O(h) Call Stack。
- DFS 適合深入 Subtree 與向上組合答案，BFS 適合按 Depth Layer 處理。
- Raw Pointer 不表達 Ownership，正式程式必須定義 Node 的建立與釋放責任。
- Debug 時應記錄 Node Identity、Depth、Child Result 與 Combine，而不只觀察最終輸出。
