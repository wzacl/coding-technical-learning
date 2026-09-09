# 問題描述

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.
You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example 1:**
![[AddTwoNumbersEx.png]]

`Input: l1 = [2,4,3], l2 = [5,6,4]`
`Output: [7,0,8]`
`Explanation: 342 + 465 = 807.`

**Example 2:**

`Input: l1 = [0], l2 = [0]`
`Output: [0]`

**Example 3:**

`Input: l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]`
`Output: [8,9,9,9,0,0,0,1]`

**Constraints:**

- The number of nodes in each linked list is in the range `[1, 100]`.
- `0 <= Node.val <= 9`
- It is guaranteed that the list represents a number that does not have leading zeros.

# 資料結構描述

本題主要使用的資料結構是「單向鏈結串列（singly linked list）」。`dummy`、`tail`、`p`、`q` 都是協助操作它的變數，並不是不同的資料結構。

### 1. 節點：`ListNode`

每個節點包含兩個屬性：

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

|屬性|用途|本題中的內容|
|---|---|---|
|`.val`|儲存節點的值|一位數字，介於 `0`～`9`|
|`.next`|參考下一個節點|下一個 `ListNode`，或表示結尾的 `None`|

- `ListNode(7)`：建立值為 `7` 的節點，預設 `.next` 是 `None`。
    
- `node.val`：讀取節點的值。
    
- `node.next`：取得下一個節點，不需要加 `()`。
    

### 2. 單向鏈結串列

- 多個節點透過 `.next` 依序連接，從第一個節點就能向後走訪整條串列。
    
- 「單向」表示每個節點只記錄下一個節點，沒有記錄前一個節點。
    
- `l1`、`l2` 分別參考兩個輸入串列的第一個節點。
    
- 本題將數字反向儲存，例如 `[3, 5, 2]` 代表 `253`。這是題目的儲存規則，不是所有鏈結串列都必須如此。
    
- 回傳結果時，只需要回傳第一個結果節點，就能透過 `.next` 讀取後續內容。
    

### 3. 操作串列的變數

Python 變數持有物件的「參考」；本題常把這類走訪變數稱為指標。

|變數|角色|移動方式|
|---|---|---|
|`p`、`q`|記錄兩個輸入串列目前處理的位置|每輪移到 `.next`，直到 `None`|
|`dummy`|參考結果串列的虛擬頭節點，保留起點|保持不動|
|`tail`|參考結果串列目前的尾端，方便接上新節點|每次新增後移到新尾端|
|`node`|參考本輪建立的新節點|透過 `tail.next = node` 接入結果|

`dummy` 是普通的 `ListNode`，只是用途不同；它的值不屬於答案，因此最後回傳 `dummy.next`。

### 4. 與 Python `list` 的差別

|比較項目|Python `list`|本題的鏈結串列|
|---|---|---|
|儲存方式|以動態陣列儲存元素參考|每個節點記錄值與下一個節點|
|取得第一個元素／節點|`items[0]`|`l1` 本身就是第一個節點|
|取得指定位置|可使用 `items[i]`|需要從開頭沿 `.next` 走訪|
|取得長度|`len(items)`|題目的 `ListNode` 不支援直接取長度，需要走訪計數|
|新增尾端內容|`items.append(value)`|建立節點，再設定 `tail.next`|
|走訪方式|可直接使用 `for`|本題使用 `while`，手動移動節點參考|

### 5. 其他輔助資料

`x`、`y`、`total`、`digit`、`carry` 都是整數，分別記錄輸入位數、總和、本位結果與進位。要特別區分：**`digit` 是數字，`ListNode(digit)` 才是裝著該數字的節點。**

# 一、題目的核心做法

- 模仿直式加法，從個位數逐位相加；題目的節點剛好以個位數在前的順序儲存。
    
- 每輪讀取兩個節點的值；若某一邊已走完，該位視為 `0`。
    
- 計算 `total = x + y + carry`，包含上一輪的進位。
    
- 用 `digit = total % 10` 取得本位數字，用 `carry = total // 10` 更新進位。
    
- 將 `digit` 存入新節點、接到結果尾端，再移動輸入指標。
    
- 兩個串列都走完且沒有進位時結束。
    

# 二、最大的邏輯誤區：為什麼沿節點走訪、使用 `while`？

- **`l1`、`l2` 是第一個節點，不是 Python 的 list。** 題目提供的 `ListNode` 不能直接使用 `len(l1)` 或 `l1[i]`，需要透過 `.next` 找到下一個節點。
    
- `p = l1`、`q = l2` 是設定走訪起點，之後用 `p = p.next`、`q = q.next` 向後移動。
    
- 本題的停止依據是「是否還有節點或進位」，因此適合使用：
    
    ```python
    while p is not None or q is not None or carry != 0:
    ```
    
- 不是鏈結串列不能用 `for`，而是 Python 的 `for` 需要可迭代物件；這題的 `ListNode` 沒有直接提供迭代功能。即使用 `for range(...)`，仍需先計算長度、沿 `.next` 移動，並處理最後進位，反而增加步驟。
    
- `or` 表示任一條件成立就繼續；誤用 `and`，會因初始 `carry = 0` 而完全不執行，也會在其中一邊先走完時提早停止。
    

# 三、節點與指標的邏輯誤區

- **初始化與移動要分開：** `p = l1`、`q = l2` 只在迴圈前執行一次；放在迴圈內會反覆回到起點。
    
- **每輪都要前進：** 忘記更新 `p`、`q`，就會一直處理同一組節點。
    
- **先確認節點存在：** `None` 沒有 `.val` 或 `.next` 屬性；最後一個節點則可以讀取 `.next`，結果是 `None`。
    
- **讀取不等於移動：** `p.next.val` 只是查看下一個節點的值，只有 `p = p.next` 才會改變 `p` 指向的位置。
    
- **指派不會複製節點：** `p = l1` 讓兩個變數指向同一個節點；移動 `p` 不會移動 `l1`，但修改 `p.val` 會修改它們共同指向的節點。
    
- **數字與節點不同：** `digit` 是整數，`node`、`tail` 指向節點。`tail = digit` 會讓 `tail` 變成整數，之後無法使用 `.next`。
    
- 已將值存入 `x`、`y` 後，可以先移動 `p`、`q` 再計算總和；這不是錯誤，只要沒有漏掉移動即可。
    

# 四、`dummy` 與結果串列

- `dummy` 是虛擬頭節點，讓第一個結果節點也能使用相同方式接上：
    
    ```python
    dummy = ListNode(0)
    tail = dummy
    ```
    
- `dummy` 保留起點，`tail` 隨新增節點移到尾端：
    
    ```python
    node = ListNode(digit)
    tail.next = node
    tail = tail.next
    ```
    
- `digit = tail` 後修改 `digit.val`，只會修改既有節點，沒有建立新節點。
    
- `tail` 尚未初始化前，不能使用 `tail.next`。
    
- 最後回傳 `dummy.next`，跳過虛擬頭；回傳 `dummy` 會多出占位節點，回傳 `tail` 只會得到最後一個節點。
    

# 五、本次語法錯誤與修正

| 原本寫法                          | 修正方式／原因                                                      |
| ----------------------------- | ------------------------------------------------------------ |
| `double carry=0,p=0,q=0`      | Python 不需宣告 `double`；分行寫 `carry = 0`、`p = l1`、`q = l2`       |
| `p=l1,q=l2`                   | 改成 `p, q = l1, l2`，或分成兩行                                     |
| `for (int i = ...; ...; i++)` | 這是 C 式語法；Python 的計次迴圈可寫成 `for i in range(n):`，但本題適合用 `while` |
| 用 `{}` 包住迴圈                   | Python 使用冒號與縮排界定區塊                                           |
| `while` 比初始化多縮排一層             | `while` 應與初始化對齊，迴圈內容再向內縮排                                    |
| `else`                        | 改成 `else:`                                                   |
| 全形冒號 `：`                      | 改成半形冒號 `:`                                                   |
| `q is nit None`               | 改成 `q is not None`                                           |
| `p.next()`                    | 改成 `p.next`，因為 `.next` 是屬性，不是方法                              |
| `listnode(...)`               | 改成 `ListNode(...)`，Python 區分大小寫                              |
| `l1(2)`                       | 這是函式呼叫語法，不能用來取得節點；目前值用 `.val`，下一個節點用 `.next`                 |
