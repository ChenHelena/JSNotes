---
description: filter(), find(), map(), every(), some(), reduce()
---

# 陣列處理方法

## Map()
用來遍歷一個陣列中的每個元素，將元素分別傳入你指定的函數，最後將所有函數的返回值組成一個新的陣列

* map() 會將原陣列透過 function 進行處理，處理後的值會回傳一個新的陣列
* 並不會改變原來的陣列
* 兩個陣列的長度會相等

```js
let arr = [1, 4, 9, 16];
let arrNew = arr.map((x) => x * 2);
console.log(arrNew)//  [2, 8, 18, 32]
```

## 過濾陣列：
### filter()
過濾一個陣列中符合條件的元素，若不符合則刪除。不更改原陣列，而回傳新陣列
* 過濾( 留下所需元素 )
```
let numbers = [20, 10, 9, 25, 1, 3, 8, 11]; 
let result = numbers.filter(function(e){ 
  return e >= 10; 
}); 
console.log(result); // [20, 10, 25, 11]
```
## 陣列搜尋：
### find()
找出陣列中第一個符合條件的元素，回傳值是一個元素或 undefined
* find() 與 filter() 很像，和 filter 的不同之處在於，filter 會回傳包含所有符合條件的項目的「陣列」，而 find 會回傳第一個符合條件的「項目」

```js
const arr = [5, 12, 8, 130, 44];
const found = arr.find((e) => e > 10);
console.log(found); //12
```

### some()
用來檢查陣列裡面是否有一些符合條件。只要有一個以上符合條件就會回傳 true，全部都不是的話會回傳 false
```js
const arr = [1, 2, 3, 4, 5];
const even = arr.some((e) =>e % 2 === 0);

console.log(even);// true
```

### every()
用來測試陣列中的所有項目是否符合某個條件，回傳值是布林值
```js
const arr = [2, 4, 6, 8, 10];
const even = arr.every((e) =>e % 2 === 0);

console.log(even);
```
:::note

some() 和 every() 的差別是，every() 必須「所有」的項目都符合條件才會回傳 true，some() 只要其中一個項目符合條件就會回傳 true

:::

## reduce()
:::note

跟其他陣列方法的差別是它會 return 一個值，而不是一個新陣列

:::
```
Array.reduce(callback[accumulator, currentValue, currentIndex, array], initialValue)

```
可以與前一個回傳的值再次作運算。參數包含以下：
* accumulator: 前一個參數，如果是第一個陣列的話，值是以另外傳入或初始化的值(累加值)
* currentValue: 當前變數
* currentIndex: 當前索引
* array: 全部陣列
* initialValue：預設值，放在 function 的最後方，非必填


### 未提供 initialValue（預設值）
若未提供預設值，accumulator就會取陣列的第一個元素也就是 1，而 currentValue 就會從陣列的第二個值開始 loop。
```
let arr = [1, 2, 3, 4, 5];
let reduceArr = arr.reduce((accumulator, currentValue) => {
  return accumulator + currentValue
});
console.log(reduceArr); // 15
```

| loop | accumulator | currentValue | return value |
| :---- | ----: | ----: | ----: |
| 1 | 1 | 2 | 3 |
| 2 | 3 | 3 | 6 |
| 3 | 6 | 4 | 10 |
| 4 | 10 | 5 | 15 |

### 有提供 initialValue（預設值）
加上預設值 0，雖然 return value 跟前面範例一樣是 15，但內部結構已不相同
```js
const arr = [1, 2, 3, 4, 5];
const reduceArr = arr.reduce((accumulator, currentValue) => {
  return accumulator + currentValue
}, 0);
console.log(reduceArr); // 15
```

| loop | accumulator | currentValue | return value |
| :---- | ----: | ----: | ----: |
| 1 | 0 | 1 | 1 |
| 2 | 1 | 2 | 3 |
| 3 | 3 | 3 | 6 |
| 4 | 6 | 4 | 10 |
| 5 | 10 | 5 | 15 |

## Object.keys()
取得物件可列舉的屬性，組成陣列後回傳
```js
var obj = { 0: "a", 1: "b", 2: "c" };
console.log(Object.keys(obj)); //  ['0', '1', '2']
```

## Object.values()
取得物件的值，組成陣列後回傳
```js
const obj = {
  a: 'somestring',
  b: 42,
  c: false,
};

console.log(Object.values(obj));// ["somestring", 42, false]
```

## Object.entries()
取得物件的屬性值，組成陣列後回傳
* 每個都是一個包含兩個元素的陣列：第一個元素是屬性，第二個元素是值。

```js
const obj = { name: "helena", age: 42 };
console.log(Object.entries(obj)); // [ ['name', 'helena'], ['age', 42] ]

```