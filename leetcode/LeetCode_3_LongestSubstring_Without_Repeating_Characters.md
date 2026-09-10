# Problem description：

Given a string `s`, find the length of the **longest** **substring** without duplicate characters.

**Example 1:**

`Input: s = "abcabcbb"`
`Output: 3`
`Explanation: The answer is "abc", with the length of 3. Note that "bca" and "cab" are also correct answers.`

**Example 2:**

Input: s = "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.

**Example 3:**

`Input: s = "pwwkew"`
`Output: 3`
`Explanation: The answer is "wke", with the length of 3.`
`Notice that the answer must be a substring, "pwke" is a subsequence and not a substring.`

**Constraints:**

- `0 <= s.length <= 105`
- `s` consists of English letters, digits, symbols and spaces.

Python solution：

``` python
class Solution: 
	def lengthOfLongestSubstring(self, s: str) -> int:
		characters=set() 
		current_length,history_length,left=0,0,0 
		for right in range(len(s)): 
			current_char=s[right] 
			while current_char in characters:
				characters.remove(s[left]) left += 1
				characters.add(current_char)
				current_length = right - left +1 
			if current_length > history_length: 
				history_length = current_length
		return history_length
```

C++ solution：
```cpp
#include <array> 
#include <algorithm> 
#include <string> 
using namespace std; 

class Solution { 
public: 
	int lengthOfLongestSubstring(string s) { 
		array<int, 128> last_seen; 
		last_seen.fill(-1); 
		int left = 0; int history_length = 0; 
		
		for (int right = 0; right < s.size(); right++) { 
			char current_char = s[right]; 
			left = max(left, last_seen[current_char] + 1);
			last_seen[current_char] = right; 
			int current_length = right - left + 1; 
			history_length = max(history_length,current_length); 
		} 
		return history_length; 
	} 
	
};
```

## 1. 題目資訊

- 題號：3
    
- 題目：Longest Substring Without Repeating Characters
    
- 中文：無重複字元的最長子字串
    
- 使用語言：
    
    - Python 3：已完成並取得正確結果
        
    - C++：嘗試改寫與優化，尚未確認最終提交結果
        
- Python 提交結果：
    
    - Runtime：174 ms，Beats 67.02%
        
    - Memory：19.92 MB，Beats 25.78%
        

---

## 2. 題目核心

給定一個字串，找出其中最長、沒有重複字元的連續子字串，並回傳其長度。

### 子字串必須連續

例如：

```text
原字串："abcaef"
```

- `"bcaef"` 是合法子字串，長度為 `5`。
    
- `"abcef"` 雖然沒有重複字元，但跳跳過中間的 `"a"`，所以不是子字串。
    

這裡要區分：

- `string`：字串這種資料。
    
- `substring`：原字串中連續的一段。
    
- `subsequence`：可以跳過部分字元，不要求連續。
    

---

## 3. 核心解題方法：滑動視窗

使用兩個索引表示目前檢查的連續區間：

- `left`：目前視窗的左端。
    
- `right`：目前視窗的右端。
    
- `right` 持續向右走訪。
    
- 遇到重複字元時，移動 `left`，直到目前區間重新變成沒有重複字元。
    

視窗長度為：

```python
current_length = right - left + 1
```

之所以加 `1`，是因為 `left` 和 `right` 兩端都包含在區間內。

例如：

```text
left = 2
right = 3
長度 = 3 - 2 + 1 = 2
```

---

## 4. Python 解法使用的資料結構

### 題目指定的資料結構：`str`

Python 字串是有順序的字元序列，可以透過索引讀取：

```python
current_char = s[right]
```

但不能直接修改某個位置：

```python
s[0] = "a"  # 不允許
```

### 輔助資料結構：`set`

集合用來記錄目前視窗內有哪些字元：

```python
characters = set()
```

常用操作：

```python
current_char in characters
characters.add(current_char)
characters.remove(s[left])
len(characters)
```

`set` 的特性：

- 不保存重複元素。
    
- 平均查詢時間為 O(1)O(1)。
    
- 適合判斷字某個字元是否已經位於目前視窗中。
    

---

## 5. Python 版本解題流程

每次處理 `s[right]` 時：

1. 取得目前字元。
    
2. 判斷該字元是否已經在 `characters` 中。
    
3. 如果重複，從視窗左側逐一移除字元。
    
4. 每移除一個字元，就將 `left` 加 `1`。
    
5. 重複消失後，將目前字元加入集合。
    
6. 計算目前視窗長度。
    
7. 更新歷史最長長度。
    
8. 最後回傳歷史最長長度。
    

關鍵片段：

```python
while current_char in characters:
    characters.remove(s[left])
    left += 1
```

---

## 6. 為什麼重複處理要使用 `while`

不能只使用 `if`，因為一次移除不一定能排除重複字元。

例如：

```text
s = "abcb"
目前視窗 = "abc"
新字元 = "b"
```

處理過程：

1. 移除 `"a"` 後，集合是 `{"b", "c"}`。
    
2. 新的 `"b"` 仍然重複。
    
3. 還要繼續移除舊的 `"b"`。
    
4. 重複消失後才能加入新的 `"b"`。
    

因此停止條件是：

```text
目前字元已經不在集合中
```

需要重複執行未知次數，所以使用 `while`。

---

## 7. 「目前長度」與「歷史最長長度」

目前視窗遇到重複字元後可能縮短，因此不能只回傳最後一個視窗的長度。

例如：

```text
s = "abcb"
```

- 曾經找到 `"abc"`，長度為 `3`。
    
- 最後的視窗可能是 `"cb"`，長度為 `2`。
    
- 正確答案仍然是 `3`。
    

因此需要兩個概念：

```python
current_length   # 目前視窗長度
history_length   # 到目前為止的最大長度
```

更新方式：

```python
history_length = max(history_length, current_length)
```

---

## 8. 本次 Python 實際發生的問題

### 8.1 字元沒有加上引號

原本寫法：

```python
characters = [a, b, c]
```

Python 會將 `a`、`b`、`c` 當成變數名稱。如果沒有宣告，會產生 `NameError`。

正確表示字元：

```python
characters = ["a", "b", "c"]
```

或者集合：

```python
characters = {"a", "b", "c"}
```

### 8.2 先加入再判斷，導致條件永遠成立

原本的執行順序：

```python
characters.add(current_char)

if current_char in characters:
```

加入後再檢查，結果必定是 `True`。

正確思考順序是：

```text
先檢查是否已經存在
→ 處理重複
→ 再加入目前字元
```

### 8.3 長度公式方向寫反

原本寫法：

```python
length = left - right + 1
```

正確方向：

```python
length = right - left + 1
```

### 8.4 混用 C/C++ 與 Python 的迴圈語法

原本使用類似：

```cpp
for(i = 0; i < len(s); i++)
```

Python 應使用：

```python
for right in range(len(s)):
```

Python 不使用 `{}` 劃分程式區塊，而是使用縮排。

### 8.5 多變數初始化錯誤

曾出現：

```python
current_length, histiry_length, left.right = 0
```

問題：

- `left.right` 的句點不是分隔變數。
    
- 多個變數不能直接接收一個不可拆分的整數。
    
- `histiry_length` 拼字錯誤。
    

應明確初始化每個變數，並保持名稱一致。

### 8.6 變數名稱不一致

曾混用：

```python
current_chart
current_char
```

`chart` 是圖表，`char` 才是 `character` 的縮寫。

Python 會將兩者視為完全不同的變數，因此本題統一使用：

```python
current_char
```

### 8.7 重複更新 `right`

當使用：

```python
for right in range(len(s)):
```

`right` 已經由 `for` 自動更新，不需要再寫：

```python
right += 1
```

否則程式中的索引與實際走訪位置會不一致。

### 8.8 回傳目前長度而不是歷史最大值

回傳：

```python
return right - left + 1
```

只能取得最後一個視窗的長度，不能保證是整個過程的最大值。

應保存並回傳歷史最大長度。

### 8.9 比較條件使用相同變數

曾出現類似：

```python
if history_length > history_length:
```

同一個值不可能大於自己。應比較「目前長度」與「歷史最大長度」。

---

## 9. C++ 版本使用的基礎語法

### C++ 的 `for` 迴圈

```cpp
for (int right = 0; right < s.size(); right++) {
    // 處理 s[right]
}
```

與 Python 的：

```python
for right in range(len(s)):
```

功能相近。

### 取得目前字元

```cpp
char current_char = s[right];
```

- `char`：單一字元型別。
    
- `string`：字串型別。
    
- `s[right]`：取得索引 `right` 的字元。
    

### `len()` 與 `.size()`

Python：

```python
len(s)
```

C++：

```cpp
s.size()
```

C++ 不能直接寫：

```cpp
len(s)
```

---

## 10. C++ 與 Python 集合操作對照

如果完全沿用 Python 的 `set + while` 邏輯，可以使用：

```cpp
unordered_set<char> characters;
```

|Python|C++|
|---|---|
|`set()`|`unordered_set<char>`|
|`x in characters`|`characters.count(x) > 0`|
|`characters.add(x)`|`characters.insert(x)`|
|`characters.remove(x)`|`characters.erase(x)`|
|`len(s)`|`s.size()`|

這種寫法與 Python 邏輯相同，但 `unordered_set` 具有雜湊表與節點管理開銷。

---

## 11. C++ 優化版本的資料結構：`array<int, 128>`

因為本題字元範圍是英文字母、數字、符號與空白，可以利用 ASCII 編碼，把字元直接當成陣列索引。

宣告方式：

```cpp
array<int, 128> last_seen;
last_seen.fill(-1);
```

資料意義：

```text
last_seen[字元的ASCII值] = 該字元最近一次出現的索引
```

例如：

```cpp
char current_char = s[right];
int previous_index = last_seen[current_char];
```

### 為什麼初始化為 `-1`

索引 `0` 是有效位置，因此不能用 `0` 表示「從未出現」。

```text
-1：尚未出現
0 以上：上次出現的位置
```

### 為什麼可以移除 `while`

使用集合時，只知道「是否出現」，不知道「出現在哪裡」，所以需要逐一移動 `left`。

`last_seen` 則保存了具體索引，可以直接更新：

```cpp
left = max(left, last_seen[current_char] + 1);
```

`max` 的作用是避免 `left` 往回移動。

例如 `"abba"` 最後讀到 `"a"` 時：

- `"a"` 上次出現在索引 `0`。
    
- 但 `left` 已經移動到索引 `2`。
    
- 不能重新退回索引 `1`。
    

因此必須保留較大的位置。

---

## 12. 本次 C++ 實際發生的問題

### 12.1 不熟悉 `array` 宣告格式

不完整寫法：

```cpp
array
```

正確語法格式：

```cpp
array<元素型別, 固定長度> 變數名稱;
```

本題使用：

```cpp
array<int, 128> last_seen;
```

### 12.2 將 `last_seen` 宣告成單一整數

原本概念：

```cpp
int last_seen = 0;
```

但 `last_seen` 需要記錄多個字元的位置，因此不能是單一整數，應改成固定陣列。

### 12.3 使用 Python 的 `len(s)`

C++ 不使用：

```cpp
len(s)
```

應使用：

```cpp
s.size()
```

### 12.4 C++ 依靠大括號決定作用域

曾遇到：

```text
a type specifier is required for all declarations
```

而錯誤位置顯示：

```cpp
left = max(left, last_seen[current_char] + 1);
```

這一行放在函式內時本身合法。這類錯誤通常表示前方大括號位置錯誤，導致編譯器把它當成類別層級的宣告。

需要檢查：

- 是否過早寫了 `}`...`
    
- 函式宣告後是否多了分號。
    
- 程式碼是否仍位於函式的大括號內。
    

### 12.5 重複宣告相同變數

曾在同一個迴圈內寫兩次：

```cpp
char current_char = s[right];
char current_char = s[right];
```

同一作用域不能重複宣告同名變數，保留一行即可。

### 12.6 尚未完成長度與最大值更新

C++ 版本一度只完成：

- 取得字元。
    
- 更新 `left`。
    
- 更新字元最近位置。
    

還需要補上：

```cpp
int current_length = right - left + 1;
history_length = max(history_length, current_length);
```

---

## 13. 效率分析

### Python `set` 滑動視窗

- `right` 最多走訪 nn 次。
    
- `left` 最多向右移動 nn 次。
    
- 每個字元最多被加入與移除一次。
    
- 時間複雜度：O(n)O(n)。
    
- 空間複雜度：O(k)O(k)，其中 `k` 是視窗中的不同字元數量。
    

雖然程式中有巢狀的 `while`，但 `left` 不會退回，因此整體不是 O(n2)O(n^2)。

### C++ `array<int, 128>` 版本

- 每個字元只處理一次。
    
- 遇到重複時直接更新 `left`。
    
- 時間複雜度：O(n)O(n)。
    
- 輔助空間複雜度：O(1)O(1)，因為陣列固定為 128 格。
    
- 若 `int` 為 4 bytes，陣列本身約占 512 bytes。
    

---

## 14. 效能結果的判讀

Python 的 `174 ms` 已經擊敗約 67% 的提交，不能只根據分布圖判斷有很大的演算法優化空間。

原因：

- LeetCode 的執行時間會受到伺服器負載影響。
    
- 相同程式重新提交，百分位可能不同。
    
- Python 記憶體包含直譯器及執行環境成本。
    
- `19.92 MB` 與主要分布區間的差距可能只有零點幾 MB。
    

本次真正有意義的優化是：

```text
set 判斷重複並逐一移除
→ 記錄最近索引並讓 left 直接跳躍
```

兩者都是 O(n)O(n)，差別主要在常數操作與資料結構開銷。

---

## 15. 測試案例

|輸入|預期輸出|測試目的|
|---|--:|---|
|`""`|`0`|空字串|
|`"a"`|`1`|單一字元|
|`"abcabcbb"`|`3`|一般重複|
|`"bbbbb"`|`1`|全部字元相同|
|`"pwwkew"`|`3`|連續移動左界|
|`"abcb"`|`3`|最長視窗不在結尾|
|`"abba"`|`2`|檢查 `left` 不能往回移動|

目前已確認 Python 版本得到正確結果；C++ `array` 版本仍應完成提交測試後才能確認通過。

---

## 16. 後續複習重點

1. 分清楚子字串與子序列。
    
2. 熟悉滑動視窗的 `left`、`right` 各自負責什麼。
    
3. 理解為何集合版本使用 `while`，而最近索引版本可以直接跳躍。
    
4. 理解巢狀迴圈不一定代表 O(n2)O(n^2)，需要觀察指標總共移動幾次。
    
5. Python 熟悉：
    
    - `set`
        
    - `in`
        
    - `add`
        
    - `remove`
        
    - `range(len(s))`
        
    - `max`
        
6. C++ 熟悉：
    
    - `string`
        
    - `char`
        
    - `array<int, 128>`
        
    - `.fill(-1)`
        
    - `.size()`
        
    - `std::max`
        
    - 大括號與作用域
        
7. 不要混用 Python 與 C++ 的迴圈、容器和長度語法。
    
8. 保持變數名稱一致，特別是 `char`、`current_char`、`history_length`。
    
9. 用 `"abba"` 檢查 `left` 是否可能錯誤地往回移動。