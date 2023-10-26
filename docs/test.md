# 【 JavaScript 學習筆記 】迴圈
## 迴圈
迴圈經常會搭配陣列與物件之類的集合性資料結構使用，重覆地或循環地取出某些滿足條件的值，或是重覆性的更動其中某些值
## for... 迴圈
```
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

```
for (let count = 0, total = 10 ; count < 10 ; count++){
    console.log(count, total)
}
```

## 關於遞增運算符 (++) 與遞減運算符 (--)
```
let x = 1
let y = 1

console.log(x++) //1
console.log(x) //2

console.log(++y) //2
```
放在運算元前面的遞增 (++) 或遞減 (--) 符號，就會先把值作加 1 或減 1，也就會直接變動值。放後面的就是值的變動要在下一次再看得到！



:::note

但是現在陣列執行迴圈不像過去那麼麻煩，其中的 forEach 基本上可以達到 for... 迴圈的所有需求

:::

## for...  與 forEach 比較
### for...
```
var arr = ['1', '2', '3', '4']
for (var i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
```
### forEach
```
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
var arr = ['1', '2', '3', '4']
for (var i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
console.log(i);
```

:::note

解決辦法：使用 let, const 來解決作用域的問題

:::

```
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
console.log(i);
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
