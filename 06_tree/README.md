## Part 6　Tree

### 這一 Part 在做什麼

這一 Part 介紹具有階層關係的資料結構。Tree 題的核心通常不是語法，而是定義節點、子樹與遞迴函式的責任。Traversal 決定資訊何時被處理；BST、Trie 與 Heap 則利用不同結構限制，提供搜尋、前綴查詢或極值維護能力。

### 學完後應該能做到

- 理解 Root、Child、Leaf、Height 與 Subtree
- 依需求選擇 Preorder、Inorder、Postorder 或 Level-order
- 利用 BST 的順序性進行搜尋與更新
- 理解 Trie 的前綴模型與 Heap 的極值模型

### 建議閱讀方式

- 先建立一般 Tree 與 Traversal 模型。
- 再閱讀 BST、Trie、Heap，觀察額外結構條件帶來的能力。

### 閱讀時可以反覆問自己

- 這個主題要解決哪一類問題？
- 它成立需要哪些前提？
- 目前維護的 State 或 Invariant 是什麼？
- 如果條件改變，原方法是否仍然成立？
- 時間與空間成本主要來自哪裡？
