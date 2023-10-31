---
description: for..., forEach 用法
---

# 迴圈
## 迴圈
迴圈經常會搭配陣列與物件之類的集合性資料結構使用，重覆地或循環地取出某些滿足條件的值，或是重覆性的更動其中某些值
## for... 迴圈
```js
for (let count = 0 ; count < 10 ; count++){
    console.log(count)
}
```

程式碼解說：
* 在迴圈一開始時，將 count 變數指定為數字 0
* count < 10 時才能執行 for 迴圈
* 每次當執行 for 中區塊的語句執行後，即會執行 count++ ，也就是 count+1

## 可自訂的表達式
第一個表達式是用作初始值的定義。它是可以設定多個定義值的，每個定義值之間使用逗號(,)作為分隔，定義值可使用的範圍只在迴圈內部的語句中：

```js
for (let count = 0, total = 10 ; count < 10 ; count++){
    console.log(count, total)
}
```

## 關於遞增運算符 (++) 與遞減運算符 (--)
```js
let x = 1
let y = 1

console.log(x++) //1
console.log(x) //2

console.log(++y) //2
```
放在運算元 **前面** 的遞增 (++) 或遞減 (--) 符號，就會先把值作加 1 或減 1，也就會直接變動值。放後面的就是值的變動要在下一次再看得到！



:::note

但是現在陣列執行迴圈不像過去那麼麻煩，其中的 forEach 基本上可以達到 for... 迴圈的所有需求

:::

## for...  與 forEach 比較
### for...
```js
var arr = ['1', '2', '3', '4']
for (var i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
```
### forEach
```js
var arr = ['1', '2', '3', '4']
arr.forEach(function(item) {
  console.log(item)
});
```

:::note

結果相同，forEach 更好閱讀

:::

## for... 可能會產生全域變數
* i 屬於全域變數
```js
for (var i = 0; i < 10; i++) {
  // ...
}
console.log(i); //10
```

:::note

解決辦法：使用 let, const 來解決作用域的問題

:::

```js
for (let i = 0; i < 10; i++) {
  // ...
}
console.log(i); //i is not defined
```
### 作用域的差異
```js
for(var i = 0; i < 10; i++){
   setTimeout( () => {

       console.log(index)

   },10 )

} // 10
```

當 JavaScript 在執行 for 迴圈時，碰到 setTimeout (是一個非同步的函式) 時，編譯器不會馬上執行它，會直接跳過它繼續累加i。

而 for 迴圈的 i 是用 var 宣告的，且我們並不是在函式內部宣告的，所以是全域宣告，導致i累加的結果會被保留下來。

等到i被累加到3時，因為它不小於3，此時，編譯器就會去執行 setTimeout的結果，而此時i被累加到3了，所以，會出現3次 "這是第3次" 的錯誤結果。

:::note

解決辦法：使用 let 來解決作用域的問題

:::

```js
for(let i = 0; i < 10; i++){
   setTimeout( () => {

       console.log(index)

   },10 )

} // 0,1,2,3,4,5,6,7,8,9
```

## for... 的特點
* 可以被中斷
```
for (let i = 0; i < arr.length; i++) {
  if (i === 2) { // 執行到索引 2 就會被中斷
    break;
  }
  console.log(i, arr[i]);
}
```
:::note

基本上用了 forEach 不太會再去使用 for loop

:::
