# 第 4 章　正確性與測試方法

## 適用範圍

本章介紹 Precondition、Postcondition、Loop Invariant、遞迴正確性、邊界案例、測試 Oracle、對拍與最小失敗案例。

## 適用讀者

- 程式能通過 Sample，但常在隱藏測資失敗的讀者。
- 能寫出解法，但不確定如何說明正確性的讀者。
- 需要系統化 Debug 流程的讀者。

## 快速導覽

- [正確性的範圍](#41-正確性的範圍)
- [Invariant](#42-loop-invariant)
- [遞迴正確性](#43-遞迴正確性)
- [邊界案例](#44-邊界案例)
- [測試 Oracle 與對拍](#45-測試-oracle-與對拍)
- [最小失敗案例](#46-最小失敗案例)

## 4.1 正確性的範圍

正確性表示對所有符合前置條件的輸入，演算法都會終止並滿足後置條件。通過少數 Sample 只是測試結果，不是完整證明。

## 4.2 Loop Invariant

Invariant 是每次迴圈固定位置都成立的敘述，常用三段式說明：

1. Initialization：第一次迴圈前成立。
2. Maintenance：每輪執行後仍成立。
3. Termination：迴圈結束時能推出答案。

```cpp
int maximum(const std::vector<int>& nums)
{
    int answer = nums[0];
    for (std::size_t i = 1; i < nums.size(); ++i)
    {
        answer = std::max(answer, nums[i]);
    }
    return answer;
}
```

Invariant：每輪結束後，`answer` 是 `[0, i]` 的最大值。

## 4.3 遞迴正確性

遞迴需要：

- Base Case 直接正確。
- 每次呼叫讓問題縮小。
- 假設較小問題正確時，目前層能正確組合答案。

若問題未縮小，可能無窮遞迴；若遺漏 Base Case，可能 Stack Overflow。

## 4.4 邊界案例

通用案例：

- 空集合。
- 一個元素。
- 兩個元素。
- 全部相同。
- 已排序或反向排序。
- 沒有答案。
- 答案位於邊界。
- `INT_MIN`、`INT_MAX` 與加法乘法溢位。

## 4.5 測試 Oracle 與對拍

Oracle 是判斷輸出正確的基準：

- 容易人工驗證的小案例。
- 較慢但簡單的直接解法。
- 已知數學性質。
- 另一份獨立實作。

```cpp
for (int test = 0; test < 10000; ++test)
{
    auto input = generateSmallRandomInput();
    auto expected = bruteForce(input);
    auto actual = optimized(input);
    if (!equivalent(expected, actual, input))
    {
        print(input);
        break;
    }
}
```

多答案問題應檢查 Result 是否滿足 Postcondition，不一定要求和直接解法回傳相同答案。

## 4.6 最小失敗案例

發現錯誤後，逐步刪除輸入元素或縮小數值，直到留下仍能重現的最小案例。記錄：

```text
最小輸入
預期輸出
實際輸出
第一個錯誤 iteration
該輪前的 invariant
該輪更新
被破壞的條件
```

第一個錯誤狀態通常比錯誤的最終結果更容易定位。

## 4.7 Property-based Testing

可驗證一般性質：

- 排序後長度與元素 Multiset 不變。
- 反轉兩次回到原序列。
- Lower Bound 前方都小於 Target，後方第一個不小於 Target。
- BFS 距離沿 Tree Edge 相差不超過一層。

## 4.8 常見問題與判讀

| 現象 | 可能原因 | 檢查方向 |
|---|---|---|
| Sample 通過但隱藏測資失敗 | 邊界未涵蓋 | 空、單元素、重複、無答案 |
| 無窮迴圈 | State 未嚴格前進 | Pointer 或搜尋區間 |
| 多答案對拍失敗 | 比較方式太嚴格 | 檢查 Postcondition |
| 只在大數失敗 | Overflow | 中間運算型別 |
| 修一題壞另一題 | 缺少 Regression | 保留舊失敗案例 |

## 4.9 本章檢查表

- [ ] 能寫出 Precondition 與 Postcondition。
- [ ] 能用三段式說明 Invariant。
- [ ] 已建立邊界案例集合。
- [ ] 有明確的測試 Oracle。
- [ ] 能將失敗縮小成最小案例。
- [ ] 修正後會重新執行舊案例。

## 4.10 本章重點

1. Sample 通過不能代替正確性。
2. Invariant 說明迴圈如何維持正確狀態。
3. Base Case 與問題縮小是遞迴終止的基礎。
4. 對拍需要可靠 Oracle。
5. Debug 應找第一個錯誤狀態，而不是只看最終輸出。
