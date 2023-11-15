---
description: ruby
---

# Ruby 程式語言
## 什麼是 IRB (Interactive Ruby) ?
IRB 是一個互動的 Ruby 環境，可以讓我們練習和語法，做些簡單的實驗。請輸入 irb 就會進入互動模式：
```js
$ irb
2.7.2 :001 > 1+1
 => 2
2.7.2 :002 >
```

在 irb 之中，每行執行完 Ruby 都會自動幫你 puts 輸出結果
```
puts "Hello, World"
```

## 印出 Hello world 的三種方法
* 使用終端機
```js
$ ruby -e "puts 'Hello world'"
```
* 使用 irb
* 寫在 .rb檔案裡，例如：helena.rb，用 Ruby 來執行它
```js
$ ruby helena.rb
```

### 印出方式
```js
print "Hello world" 

puts "Hello world"

p "Hello world"
```

## 四大變數種類
* 區域變數 `username `
* 全域變數 `$username`
* 實體變數 `@username`
* 類別變數 `@@username`


## Ruby 小筆記


:::note
* 變數：first_name( 蛇式命名 )
* 常數：首字大寫
* %Q 雙引號：加入變數會翻譯
* %q 單引號：加入變數不會翻譯
* 口訣：整數除以除整數會得到整數
* 在 Ruby 世界裡只有 nil 跟 false 是假的，其他都是真的
* to_a：轉型陣列
* .odd?：詢問是否為單數
:::

## 迴圈種類
* for
* while 
* loop
* method
* 迭代式

## 陣列方法
* map
* select 
* reduce 
 * sum：可直接計算總和
* compact：刪除 nil
* sort
* uniq：刪除重複

## 範圍
* 1..10：1 ~ 10
* 1...10：1 ~ 9，不包含 10

