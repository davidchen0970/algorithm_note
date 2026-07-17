# 第 9 章　Queue 與 Deque

## 適用範圍

本章介紹 FIFO、BFS Layer、Multi-source BFS、Circular Queue、Deque 與 Monotonic Queue。

## 適用讀者

- 需要理解 BFS 最短步數理由的讀者。
- 需要處理 Sliding Window Maximum 的讀者。

## 快速導覽

- [Queue 核心模型](#91-queue-核心模型)
- [BFS Layer](#92-bfs-layer)
- [Deque 與 Monotonic Queue](#93-deque-與-monotonic-queue)
- [Queue 選擇](#94-queue-選擇)

## 9.1 Queue 核心模型

Queue 是 First In, First Out。較早進入的狀態先被處理，適合事件順序、工作排程與 BFS。

## 9.2 BFS Layer

普通 BFS 在每條 Edge 成本相同時，按距離層級擴張。Node 第一次被發現時，其路徑具有最少 Edge 數。

```cpp
std::vector<int> bfsDistances(
    const std::vector<std::vector<int>>& graph,
    int source)
{
    std::vector<int> distance(graph.size(), -1);
    std::queue<int> q;
    distance[source] = 0;
    q.push(source);

    while (!q.empty())
    {
        int node = q.front();
        q.pop();
        for (int next : graph[node])
        {
            if (distance[next] != -1) continue;
            distance[next] = distance[node] + 1;
            q.push(next);
        }
    }
    return distance;
}
```

在入列時標記可避免同一 Node 被多個父節點重複加入。

## 9.3 Deque 與 Monotonic Queue

Sliding Window Maximum：

```cpp
std::vector<int> maxSlidingWindow(const std::vector<int>& nums, int k)
{
    std::deque<int> candidates;
    std::vector<int> answer;

    for (int right = 0; right < static_cast<int>(nums.size()); ++right)
    {
        while (!candidates.empty() && candidates.front() <= right - k)
            candidates.pop_front();
        while (!candidates.empty() && nums[candidates.back()] <= nums[right])
            candidates.pop_back();

        candidates.push_back(right);
        if (right + 1 >= k)
            answer.push_back(nums[candidates.front()]);
    }
    return answer;
}
```

Front 移除位置過期的 Index；Back 移除被新值支配的候選。兩者原因不同。

## 9.4 Queue 選擇

- Queue：依到達順序。
- Deque：兩端都要更新。
- Priority Queue：依 Priority。

有不同權重的 Graph 通常不能直接用普通 BFS，可能需要 Dijkstra。

## 9.5 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| BFS 重複大量 Node | Visited 標記太晚 | 是否入列時標記 |
| Window 最大值過期 | 未移除 Front | Index 是否離開 Window |
| 最大值候選錯誤 | Back 比較方向錯 | Deque 單調性 |
| 有權圖距離錯誤 | 使用普通 BFS | Edge 成本是否一致 |

## 9.6 本章檢查表

- [ ] 能解釋 BFS 第一次發現為何最短。
- [ ] 知道 Visited 標記時機。
- [ ] 能區分 Deque Front 過期和 Back 支配移除。
- [ ] 能依順序需求選 Queue、Deque 或 Priority Queue。

## 9.7 本章重點

1. Queue 依加入順序處理狀態。
2. 無權 BFS 按距離 Layer 擴張。
3. Deque 支援兩端更新。
4. Monotonic Queue 保存仍可能成為答案的候選 Index。
