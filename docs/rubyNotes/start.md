---
description: irb、印出的方法、陣列、範圍、Hash、模組化
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

puts "Hello world" //無回傳值：回傳 nil

p "Hello world" //有回傳值
```

## 四大變數種類
* 區域變數 `username `
* 全域變數 `$username`
* 實體變數 `@username`
* 類別變數 `@@username`


## Ruby 小筆記


:::note
* 變數：first_name( 蛇式命名 )
* 常數：首字大寫( 類別也是開頭大寫 )
* %Q 雙引號：加入變數會翻譯
* %q 單引號：加入變數不會翻譯
* 口訣：整數除以除整數會得到整數
* 在 Ruby 世界裡只有 nil 跟 false 是假的，其他都是真的
* .to_a：轉型陣列
* .odd?：詢問是否為單數
* .shuffle：隨機排列
* .sample(5)：隨機取樣 5 個值
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

## 雜湊（ Hash ）
```js
//舊式寫法
profile = { :name => helena, :age => 18 }
//新式寫法
profile = { name:helena, age:18 }
//name : key, age : value
```
* 物件取值
```js
p.profile[:name] //helena
```

## 符號
* 有名字的物件，也是一個值
* 設定好的符號名稱，無法更改，但變數可以修改
```js
p :hello //hello
p 2 //2
```
:::danger
不能當作變數使用 ! ! !
`
:name = 'helena'
`
:::

為什麼不行當作變數使用，可以把它想成以下：
```js
2 = 'helena'
```

* 因為對於符號物件，等同於一個 2 的值

### object_id
* 印出記憶體序號（ 字串類別：每次按都不同，符號：相同 ）
* 而數字中的，object_id，是以 2n+1 來計算
```js
p 2.object_id //5
p 3.object_id //7
```


但還是有方法可以讓字串類別的序號，出現的是相同的：
```js
p "hello".freeze.object_id
```

字串轉符號 ( symbol )
```js
p "name".to_sym
```

符號轉字串( string )
```js
p :name.to_s
```


### 符號及字串的該如何選擇？
* 不可變：符號
* 字串：可變化性

## 方法
```js
def say_hello_to(someone) //參數：parameter
  puts "hello ${someone}"
end

// 小括號可省略
say_hello_to('helena') //引數：argument
//hello helena
```
如果有帶參數的地方沒有加入引數，會報錯，但是可以在參數的位置加入參數預設值，這樣一來，只要沒有加入引數的方法，都一律跑出預設值 Amy
```js
def say_hello_to(someone = 'Amy') //參數：parameter
  puts "hello ${someone}"
end

say_hello_to // Amy
```
* 呼叫方法
```js
def age
return 20 //return 可適時省略，會自動回傳最後一行的執行結果
end

puts age() //呼叫方法
```

* 引數的最後一個值是 Hash 的話，大括號可以省略：
```js
//原型
link_to ("首頁", root_path, {class:"btn", method:"post", comfirm:true})
//省略大括號
link_to ("首頁", root_path, class:"btn", method:"post", comfirm:true)
//常見
link_to "首頁", root_path, class:"btn", method:"post", comfirm:true
```

## 問號？
* 問號只能放在命名的最後面，詢問真假值
```js
def is_adult?(age)
  if age >= 18
    return ture
  else
    return false
  end
end 
```

## 驚嘆號！
原本用 sort 排列，會形成新的陣列，而不影響原來的陣列
```js
list = [5,7,1,3]
p list.sort //[1,3,5,7]
p list //[5,7,1,3]
```
但是加了驚嘆號，就會影響原本的陣列

```js
list = [5,7,1,3]
p list.sort! //[1,3,5,7]
p list //[1,3,5,7]
```

## 模組化
```jsx title="hello.rb"
def say_hello
  puts "hi"
end
```

```jsx title="main.rb"
//引入
require './hello' //副檔名可加可不加
load './hello.rb' 

say_hello //hi
```

### load & require 差異
* load 一定要加副檔名，require 則不用
* require 只會載入一次，而 load 每次執行就會載入一次

