# 第 8 章　Stack

## 適用範圍

本章介紹 LIFO、括號匹配、Undo、Expression、DFS 與 Monotonic Stack，說明 Stack Element 應保存什麼狀態。

## 適用讀者

- 會使用 Push/Pop，但不理解 Stack State 語意的讀者。
- 需要學習括號與 Next Greater Element 的讀者。

## 快速導覽

- [核心模型](#81-核心模型)
- [括號匹配](#82-括號匹配)
- [Monotonic Stack](#83-monotonic-stack)
- [正確性與複雜度](#84-正確性與複雜度)

## 8.1 核心模型

Stack 是 Last In, First Out。它常保存「最近加入、尚未完成」的狀態：

- 尚未配對的開括號。
- DFS 尚未展開的 Node。
- 尚未確定答案的 Index。
- Undo 所需的歷史狀態。

## 8.2 括號匹配

```cpp
bool isValidParentheses(const std::string& text)
{
    std::stack<char> openings;
    for (char ch : text)
    {
        if (ch == '(' || ch == '[' || ch == '{')
        {
            openings.push(ch);
            continue;
        }
        if (ch != ')' && ch != ']' && ch != '}') continue;
        if (openings.empty()) return false;

        char open = openings.top();
        openings.pop();
        if (!((open == '(' && ch == ')') ||
              (open == '[' && ch == ']') ||
              (open == '{' && ch == '}')))
        {
            return false;
        }
    }
    return openings.empty();
}
```

Invariant：Stack 保存目前 Prefix 中尚未配對的開括號，順序和巢狀結構一致。

## 8.3 Monotonic Stack

找右側第一個更大值：

```cpp
std::vector<int> nextGreater(const std::vector<int>& nums)
{
    std::vector<int> answer(nums.size(), -1);
    std::stack<int> indices;

    for (int i = 0; i < static_cast<int>(nums.size()); ++i)
    {
        while (!indices.empty() && nums[indices.top()] < nums[i])
        {
            answer[indices.top()] = nums[i];
            indices.pop();
        }
        indices.push(i);
    }
    return answer;
}
```

Stack 保存尚未找到右側更大值的 Index。新值使條件成立時，它是該 Index 遇到的第一個更大值。

## 8.4 正確性與複雜度

每個 Index 最多 Push 一次、Pop 一次，因此總時間 `O(n)`，不是 `O(n²)`。空間最差 `O(n)`。

`<` 與 `<=` 取決於題目要找「嚴格更大」或「大於等於」，也影響重複值保留哪個 Index。

## 8.5 常見問題與判讀

| 現象 | 原因 | 檢查 |
|---|---|---|
| 空 Stack Crash | 未檢查 Empty | `top()` 前置條件 |
| 距離算不出 | 只存 Value | 是否需保存 Index |
| 重複值答案錯 | `<`、`<=` 語意錯 | 嚴格或非嚴格單調 |
| Stack 不再單調 | Push/Pop 條件錯 | 每輪 Stack 內容 |
| 複雜度誤判平方 | 未看總 Pop 次數 | 每元素最多處理次數 |

## 8.6 本章檢查表

- [ ] 能說明 Stack Element 代表的未完成狀態。
- [ ] `top()` 前會檢查 Empty。
- [ ] 能證明 Monotonic Stack 的 Pop 結果已確定。
- [ ] 能依題意選擇嚴格或非嚴格單調。
- [ ] 能說明總時間為 `O(n)`。

## 8.7 本章重點

1. Stack 適合管理最近尚未完成的狀態。
2. 括號匹配依賴巢狀順序。
3. Monotonic Stack 用支配關係移除不再需要的候選。
4. 保存 Value 或 Index 由輸出需求決定。
