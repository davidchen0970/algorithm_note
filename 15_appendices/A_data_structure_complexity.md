## 附錄 A　常見資料結構複雜度表

### 適用範圍

本附錄提供常見資料結構的快速查閱表。表中的成本是一般常見實作下的複雜度，不代表所有語言、容器或輸入條件都相同。

使用前應確認：

- Average、Amortized 或 Worst Case。
- 是否已知元素位置。
- 是否包含重新配置、Hash Collision 或 Tree 退化。
- Graph 使用 Adjacency List 還是 Matrix。

### A.1 Array

<table>
<tr><th>工作</th><th>成本</th><th>注意事項</th></tr>
<tr><td>依 Index 讀寫</td><td>O(1)</td><td>Index 必須合法</td></tr>
<tr><td>線性搜尋</td><td>O(n)</td><td>未排序資料</td></tr>
<tr><td>尾端加入</td><td>Amortized O(1)</td><td>重新配置那次可能 O(n)</td></tr>
<tr><td>中間插入或刪除</td><td>O(n)</td><td>需搬移後方元素</td></tr>
</table>

### A.2 Linked List

<table>
<tr><th>工作</th><th>成本</th><th>注意事項</th></tr>
<tr><td>已知 Node 後插入</td><td>O(1)</td><td>不含尋找 Node</td></tr>
<tr><td>已知 Node 刪除</td><td>O(1)</td><td>Singly List 可能需前一節點</td></tr>
<tr><td>依位置存取</td><td>O(n)</td><td>無 Random Access</td></tr>
<tr><td>搜尋</td><td>O(n)</td><td>需逐節點走訪</td></tr>
</table>

### A.3 Stack 與 Queue

<table>
<tr><th>工作</th><th>Stack</th><th>Queue</th></tr>
<tr><td>加入</td><td>push O(1)</td><td>push O(1)</td></tr>
<tr><td>移除</td><td>pop O(1)</td><td>pop O(1)</td></tr>
<tr><td>查看下一個</td><td>top O(1)</td><td>front O(1)</td></tr>
<tr><td>任意搜尋</td><td>O(n)</td><td>O(n)</td></tr>
</table>

### A.4 Hash Table

<table>
<tr><th>工作</th><th>平均</th><th>最差</th></tr>
<tr><td>查詢</td><td>O(1)</td><td>O(n)</td></tr>
<tr><td>插入</td><td>O(1)</td><td>O(n)</td></tr>
<tr><td>刪除</td><td>O(1)</td><td>O(n)</td></tr>
</table>

最差情況與 Collision、Hash Function、Load Factor 與 Rehash 有關。Hash Table 不提供一般排序順序。

### A.5 Binary Search Tree

<table>
<tr><th>工作</th><th>平均</th><th>最差退化</th></tr>
<tr><td>查詢</td><td>O(log n)</td><td>O(n)</td></tr>
<tr><td>插入</td><td>O(log n)</td><td>O(n)</td></tr>
<tr><td>刪除</td><td>O(log n)</td><td>O(n)</td></tr>
</table>

未平衡 BST 可能退化成 Linked List。

### A.6 Balanced Search Tree

<table>
<tr><th>工作</th><th>成本</th></tr>
<tr><td>查詢</td><td>O(log n)</td></tr>
<tr><td>插入</td><td>O(log n)</td></tr>
<tr><td>刪除</td><td>O(log n)</td></tr>
<tr><td>依序走訪全部元素</td><td>O(n)</td></tr>
</table>

C++ `std::set`、`std::map` 常提供此類複雜度保證，但具體內部結構由標準函式庫實作決定。

### A.7 Heap

<table>
<tr><th>工作</th><th>成本</th></tr>
<tr><td>查看 Top</td><td>O(1)</td></tr>
<tr><td>插入</td><td>O(log n)</td></tr>
<tr><td>移除 Top</td><td>O(log n)</td></tr>
<tr><td>Build Heap</td><td>O(n)</td></tr>
<tr><td>查找任意值</td><td>O(n)</td></tr>
</table>

Heap 不是完整排序結構。

### A.8 Trie

令 L 為字串長度：

<table>
<tr><th>工作</th><th>成本</th></tr>
<tr><td>插入字串</td><td>O(L)</td></tr>
<tr><td>完整查詢</td><td>O(L)</td></tr>
<tr><td>前綴查詢</td><td>O(L)</td></tr>
</table>

空間取決於總節點數、字元集合與 Children 表示方式。

### A.9 Graph Representation

<table>
<tr><th>表示方式</th><th>空間</th><th>列出 u 的 Neighbor</th><th>檢查 Edge u,v</th></tr>
<tr><td>Adjacency List</td><td>O(V + E)</td><td>O(deg(u))</td><td>一般 O(deg(u))</td></tr>
<tr><td>Adjacency Matrix</td><td>O(V²)</td><td>O(V)</td><td>O(1)</td></tr>
<tr><td>Edge List</td><td>O(E)</td><td>一般 O(E)</td><td>一般 O(E)</td></tr>
</table>

### A.10 Fenwick Tree 與 Segment Tree

<table>
<tr><th>工作</th><th>Fenwick Tree</th><th>Segment Tree</th></tr>
<tr><td>建立</td><td>O(n) 或 O(n log n)</td><td>O(n)</td></tr>
<tr><td>Point Update</td><td>O(log n)</td><td>O(log n)</td></tr>
<tr><td>Prefix Query</td><td>O(log n)</td><td>O(log n)</td></tr>
<tr><td>一般 Range Query</td><td>依可逆運算而定</td><td>O(log n)</td></tr>
<tr><td>空間</td><td>O(n)</td><td>O(n)</td></tr>
</table>

### 使用檢查表

- 表中的成本是平均、攤銷還是最差？
- 是否把尋找位置的成本漏掉？
- 是否需要排序順序？
- 是否會大量修改中間位置？
- Graph 的 V、E 與表示方式是否一致？
- 額外空間是否包含節點、Pointer 與 Bucket？
