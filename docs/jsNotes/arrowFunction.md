---
description: 陳述式與表達式
---

# 箭頭函式

## 箭頭函式的好處
箭頭函式(Arrow Functions)是ES6標準中，最受歡迎的一種新語法。
* 語法簡單
* 讓程式碼的可閱讀性提高
* 某些情況下可以綁定this值

判斷一段程式碼為陳述式或表達式，可依據「是否會回傳結果(值)」來做判斷

## 什麼是表達式與陳述式？

### 陳述式 Statement
:::note

執行命令的動作，下了指定執行完畢後，不會回傳值，常見的如下：

:::

* 流程控制（if...else）
* 宣告（var, let, const）、return
* 迴圈（for）
* 其他（import, export）

### 表達式 Expression

:::note

特點在於會回傳一個值，常見的如下：

:::

* 純值
* 運算值
* 執行函式
* 函式表達式
* 正規表達式

#### 函式陳述式

:::note

也稱為具名涵式

:::

* 定義一個函式並呼叫它

```js
function numA(x){
  return x * x
}
```

#### 函式表達式

:::note

又稱為匿名函式

:::

* 先宣告一個變數，並且用 ' = ' 運算子將函式賦值到宣告的變數上

```js
const numB = function {
  return x * x
}
```
```js
let numB = function {
  return x * x
}
```