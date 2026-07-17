## 附錄 G　C++ 演算法基礎

### 適用範圍

本附錄整理閱讀與撰寫 C++ 演算法程式時會反覆使用的語言觀念。重點是理解資料是否被複製、誰可以修改資料、物件何時仍有效，以及型別能否保存運算結果。

### G.1 Value、Reference 與 Pointer

<table>
<tr><th>形式</th><th>語意</th><th>常見用途</th></tr>
<tr><td>`T value`</td><td>函式擁有一份獨立值</td><td>小型型別、需要副本</td></tr>
<tr><td>`T& value`</td><td>呼叫端物件的別名，可修改</td><td>輸出參數、原地修改</td></tr>
<tr><td>`const T& value`</td><td>唯讀別名，不複製大型物件</td><td>唯讀 vector、string、struct</td></tr>
<tr><td>`T* value`</td><td>保存位址，可為 null</td><td>Tree Node、Linked List、可選物件</td></tr>
</table>

Reference 必須綁定有效物件；Pointer 可以是 `nullptr`，也可以重新指向其他物件。

### G.2 函式參數傳遞

#### Call by Value

```cpp
void increment(int value)
{
    ++value;
}
```

不會修改呼叫端整數。

#### Call by Reference

```cpp
void sortValues(std::vector<int>& values)
{
    std::sort(values.begin(), values.end());
}
```

會修改呼叫端容器。

#### Pointer Parameter

```cpp
void visit(TreeNode* node)
{
    if (node == nullptr)
    {
        return;
    }
}
```

呼叫前後都要考慮 null 與生命週期。

#### const Reference

```cpp
long long sum(const std::vector<int>& values);
```

適合不需修改的大型物件。

#### 選擇方式

- 小型、容易複製的型別：Value。
- 要修改呼叫端且必須存在：Reference。
- 不修改大型物件：const Reference。
- 允許沒有物件或處理 Node 連結：Pointer 或更明確的抽象。

### G.3 Stack Memory 與 Heap Memory

一般區域變數與 Call Frame 常與 Stack Lifetime 相關。動態配置物件通常位於 Dynamic Storage。

不要回傳區域變數的位址或 Reference：

```cpp
int* bad()
{
    int value = 3;
    return &value;
}
```

函式返回後，`value` 的生命週期已結束。

### G.4 Call Stack 與 Stack Frame

每次函式呼叫會保存參數、區域變數與返回位置。遞迴深度過大可能造成 Stack Overflow。

分析 Recursive Algorithm 空間時，要列入最大呼叫深度。

### G.5 Stack Container 與 Call Stack

`std::stack<T>` 是容器介面，由程式控制 `push` 與 `pop`。

Call Stack 是函式呼叫機制。兩者都符合 LIFO，但不是同一個東西。Iterative DFS 使用 `std::stack`，Recursive DFS 使用 Call Stack。

### G.6 RAII 與 Smart Pointer

RAII 讓資源生命週期綁定物件生命週期。

```cpp
std::unique_ptr<TreeNode> root =
    std::make_unique<TreeNode>();
```

- `unique_ptr` 表示單一 Ownership。
- `shared_ptr` 表示共享 Ownership，但有額外成本與 Cycle 風險。
- 非擁有關係可使用 Raw Pointer 或 Reference，但要確保被指物件仍存在。

演算法題常由平台管理 TreeNode 生命週期，此時應遵循題目介面，不任意 delete 外部提供的 Node。

### G.7 常見 STL 傳遞方式

#### vector 與 string

唯讀：

```cpp
void process(const std::vector<int>& values);
void process(const std::string& text);
```

原地修改：

```cpp
void normalize(std::vector<int>& values);
```

需要副本並預計修改：

```cpp
std::vector<int> sortedCopy(std::vector<int> values);
```

#### unordered_map 與 unordered_set

大型容器通常避免不必要複製。若函式會更新計數或集合，使用非 const Reference；只讀則使用 const Reference。

#### pair 與小型 struct

小型且容易複製的型別可使用 Value。若 Struct 很大，使用 const Reference。

#### TreeNode* 與 ListNode*

Pointer 指向 Node，不表示自動取得 Ownership。修改 `node->value` 會修改原 Node；只改區域 Pointer 的指向，不會改變呼叫端 Pointer，除非傳入 Pointer Reference 或 Pointer to Pointer。

### G.8 型別、Overflow 與效能

#### int

常見至少 32-bit，但精確寬度應依標準與環境確認。演算法題常用於 Index 與中等範圍整數。

#### long long

常用於總和、乘積、距離與計數。指定給 `long long` 不代表前面的 int 運算不會溢位：

```cpp
long long product = 1LL * a * b;
```

#### size_t

容器的 `size()` 回傳 unsigned 型別。以下可能 underflow：

```cpp
for (std::size_t i = values.size() - 1; i >= 0; --i)
```

若需反向使用負值終止，可先判斷空輸入，或使用合適的 signed Index。

#### signed 與 unsigned 比較

混合比較可能造成負數被轉成很大的 unsigned 值。可統一型別，並在轉型前確認資料大小可表示。

#### Integer Promotion

較小整數型別做算術時可能先提升為 int。位移與乘法前要確認實際運算型別。

### G.9 Iterator 與失效

`vector` 重新配置後，原本的 Pointer、Reference 與 Iterator 可能失效。中間插入或刪除也可能使位置之後的 Iterator 失效。

使用 Iterator 前應確認容器是否在中途改變。

### G.10 C++ 演算法程式閱讀檢查表

- 參數是 Value、Reference、const Reference 還是 Pointer？
- 函式是否修改呼叫端資料？
- 是否發生不必要的容器複製？
- Pointer 是否可能為 null？
- Object Lifetime 是否足夠？
- Iterator、Reference 是否因容器修改失效？
- signed 與 unsigned 是否混用？
- 中間算術是否可能 Overflow？
- 遞迴深度是否可能過高？
- 資源 Ownership 是否清楚？
