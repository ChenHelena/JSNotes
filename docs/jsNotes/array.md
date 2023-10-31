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

## filter()
過濾一個陣列中符合條件的元素，若不符合則刪除。不更改原陣列，而回傳新陣列
* 過濾( 留下所需元素 )
```
let numbers = [20, 10, 9, 25, 1, 3, 8, 11]; 
let result = numbers.filter(function(e){ 
  return e >= 10; 
}); 
console.log(result); // [20, 10, 25, 11]
```

## reduce()
:::note

跟其他陣列方法的差別是它會 return 一個值，而不是一個新陣列

:::
```
Array.reduce(callback[accumulator, currentValue, currentIndex, array], initialValue)

```
可以與前一個回傳的值再次作運算。參數包含以下：
* accumulator: 前一個參數，如果是第一個陣列的話，值是以另外傳入或初始化的值
* currentValue: 當前變數
* currentIndex: 當前索引
* array: 全部陣列
* initialValue：預設值，放在 function 的最後方，非必填


* 未提供 initialValue（預設值）
若未提供預設值，accumulator就會取陣列的第一個元素也就是 1，而 currentValue 就會從陣列的第二個值開始 loop。
```
let arr = [1, 2, 3, 4, 5];
let reduceArr = arr.reduce((accumulator, currentValue) => {
  return accumulator + currentValue
});
console.log(reduceArr); // 15
```


