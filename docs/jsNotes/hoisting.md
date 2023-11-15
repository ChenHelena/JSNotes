---
description: var, let, const 用法
---

# Hoisting 變數提升及 暫時性死區（TDZ）
## var 的特性
* 可以重複宣告
* 可以在使用之後才被宣告
* 作用域為函式
* 會建立一個新屬性在 windows 下
* 具有提升現象

## var 的函式作用域
用 var 宣告的變數的作用域是函式作用域 (function-level scope)。換句話說，**用 var 宣告的變數，只有在「函式」裡可以看得到**

```js
{
  var myName = "helena";
}
console.log(myName); // helena
```

:::note

雖然結果會印出 helena ，但是用 var 宣告的變數，只有在定義它的函式裡面可以看得到它，但這裡沒有函式，所以這裡的 myName 是一個 global 變數

:::

上段程式碼等同於以下，在最開始的時候宣告了global 的 myName 的變數

```js
var myName;
{
  myName = "helena";
}
console.log(myName); // helena
```
* 簡單來說，用大括弧將一個用 var 宣告的變數包起來，並沒有辦法讓它變成一個區域變數

## var 如何變成區域變數？
當我們在一個函式內用 var 宣告變數，這個變數就會變成一個區域變數，只存在此函式中

```js
function printName() {
  var myName = "helena";
  console.log(myName);
}
printName(); // helena
console.log(myName); // ReferenceError: myName is not defined
```

## 什麼是 Hoisting？

宣告 var 的時候，宣告會被提前至函式作用域的開頭，這個特性又稱為 hoisting

```js
function year(){
    console.log('hello')
}

//匿名函式
var year2 = function(){
    console.log('hello hello')
}

year();
year2();
```

程式碼逐行讀取，也就是說，不論你宣告 var 變數的位置在哪，宣告的動作一律都會被「抬升」到函式的最頂端，這個特性就叫做 hoisting (提升)

```js
var year2; //hoisting

function year(){
    console.log('hello')
}

//匿名函式
year2 = function(){
    console.log('hello hello')
}

year();
year2();
```

因為宣告都會被拉到最頂端的關係

```js
var year2; //undefined
year2();
function year(){
    console.log('hello')
}

//匿名函式
year2 = function(){
    console.log('hello hello')
}

year();
```
結果會出現錯誤

`
year2 is not a funtion
`

這是因為還沒執行到第二行的函式，結果會出現 error

```js
var year2; //undefined
year2(); //還沒執行到 year2 的函式
function year(){
    console.log('hello')
}

//匿名函式
year2 = function(){
    console.log('hello hello')
}

year();
```
:::note

只有「宣告」這個動作有 hoisting (提升) 的特性，賦值 (把值指定給變數) 的動作不會 hoisting。

:::

### 函式與變數同名，函式優先

```js
console.log(a) //[Function: a]
var a
function a(){}
```
:::note

除了變數宣告以外，function 的宣告也會提升而且優先權比較高，因此上面的程式碼會輸出 function 而不是 undefined

:::



## Hoisting 的好處
Hoisting 這個特性可以解決一個問題，兩個函數需要互相呼叫彼此的狀態，也就是 A( ) 裡面會呼叫到 B( )，而 B( ) 裡面會呼叫的 A( ) 的遞迴狀況。
```js
function isEven(n) {
  if (n === 0) return true;
  return isOdd(n - 1);
}

function isOdd(n) {
  if (n === 0) return false;
  return isEven(n - 1);
}

isEven(10); // true
```



## let 和 const 的區塊作用域 (block-level scope)
### let, const 的特性
* **不**可以重複宣告
* 作用域為區塊：let 和 const 宣告的變數，只有在「**區塊**」裡面才看得到
* **不**會建立一個新屬性在 windows 下
* 暫時性死區

```js
{
  let myName = "helena";
  console.log(myName); // helena
}
console.log(myName); // ReferenceError: myName is not defined
```

## 什麼是暫時性死區 (Temporal Dead Zone) ？
在 JavaScript 中，當我們使用 let 或 const 宣告某個變數或常數時，在該變數或常數還沒有被賦予值之前會產生一段它們不能被使用的狀況，這段時間就是所謂的暫時性死區 (Temporal Dead Zone) 的情況

```js
console.log(myName); // 這裡會報 ReferenceError 錯誤
let myName = helena;
```
:::note

在 ES6 之前只有 var 可以宣告變數，所以很容易就發生變數在還沒有被宣告前就能拿來使用的狀況。但在暫時性死區 (TDZ) 的出現之後，如果在變數還沒有宣告之前就使用的話便會出現錯誤訊息。因此 TDZ 的產生可以培養我們良好的使用習慣，也就是説在變數還沒有被宣告的時候就不要使用它。

:::

## 作用域的差異 （var, let）
```js
for(var i = 0; i < 10; i++){
   setTimeout( () => {

       console.log(i)

   },10 )

} // 10
```

當 JavaScript 在執行 for 迴圈時，碰到 setTimeout (是一個非同步的函式) 時，編譯器不會馬上執行它，會直接跳過它繼續累加i。

而 for 迴圈的 i 是用 var 宣告的，且我們並不是在函式內部宣告的，所以是全域宣告，導致i累加的結果會被保留下來。

等到 i 被累加到 10 時，因為它不小於 10，此時，編譯器就會去執行 setTimeout的結果，而此時i被累加到 10 了，所以，會出現 10 的錯誤結果。

:::note

解決辦法：使用 let 來解決作用域的問題

:::

```js
for(let i = 0; i < 10; i++){
   setTimeout( () => {

       console.log(i)

   },10 )

} // 0,1,2,3,4,5,6,7,8,9
```


