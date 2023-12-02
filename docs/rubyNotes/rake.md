---
description: rake、block、lambda
---

# 什麼是 Rake ？
Rake 用於定義和執行任務，可以使用在任何環境，並非只有 Ruby 專案可以使用，只要有 Rakefile 就可以使用 Rake 來執行任務。

### Rake 使用方法
* 首先要確認 rake 是否存在，請在 CLI 輸入指令  ` $ gem list --local ` 來進行確認，若 gem list 中沒有 rake，請先執行 ` $ gem install rake `

再來建立一個資料夾，資料夾裡面加入 rakefile 的檔案
```js
mkdir rake
cd rake/
touch rakefile
```

接著，再到 rakefile 裡面去新增內容
```jsx title="rakefile"
desc "mail sender" //描述
task :sendmail do
  puts "get email list from database"
  sleep 2 //休息
  puts "sending email..."
  sleep 2
  puts "done"
end
```

回到終端機上輸入：`rake sendmail`，終端機會執行：

```
get email list from database
sending email...
done
```

這段語法可以顯示描述檔，會知道這段程式碼要做什麼事
```js
task -T // #mail sender
```

在終端機輸入：`rake`，後面沒有包含參數的話，會產生預設值，所以要先指定預設值的數值，可以讓預設值指向原有的方法
```js
task :default => :sendmail //指向
```

### Rake 任務相依性
```js
task goto_toliet => :open_the_door do //上廁所前先開門
  puts "上廁所"
end

task open_the_door do
  puts "開門"
end
```

### 使用 Namespace 為 task 命名
當有多個 task 是具相關性或是專案成長到一定程度時，可能會發生任務名稱命名上的衝突。為了避免衝突，可以使用 namespace 包覆它，使它區分開來
```js
namespace 'main' do
  task :hello

  end
end

namespace 'demo' do
  task :hello
  
  end
end
```

## block 應用

* Block 不是參數
* Block 不是物件
```js
def my_select(list)
  result = []  
  list.each do |n|
    result << n if yield(n) //判斷是否為單數
  end
  result
end

p my_select([1,2,3,4,5]) {|i| i.odd?}
```

* 搭配 yield
```js
def say_hello
  yield //主控權讓出去
end

say_hello //因為後面沒有加入 block, 所以會報錯
```

解決這個問題，可以使用內建 `block_given?` 的方法
```js
def say_hello
  if block_given? //檢查 block 是否存在，不存在就跳過
    yield
  end
end

say_hello //不存在就跳過
say_hello{
  puts "hello" //hello
}
```

### block 的寫法差異
```js
list = [1,2,3,4,5]
p list.map { |i|
  i * 2
} // [2,4,6,8,10]

p list.map do |i|
  i * 2
end // Enumerator:[1,2,3,4,5]:map
```
:::note
會造成不同結果的原因，是因為大括號的優先順序較高，而 do ... end 的優先順序較低，所以變成先跟 p 結合了，造成後面的 Block 就不會被處理了
:::

### Block 物件化
#### `Proc` 類別：

```js
add_two = Proc.new {
  |n| n + 2
}
```
要使用它的時候，只要執行這個物件上的 `call`：
```js
add_two.call(3) //5
```
Proc 呼叫方式：
```js
add_two.call(3)
add_two.[3]
add_two.(3)
add_two.===
```

### lambda 的兩種方式
  * lambda是一個可以創建**匿名**函數的對象
  * 變數作用域 : Lambda 具有閉包（closure）性質，可以捕獲和保留其創建時的變數環境
  * 參數處理 : Lambda 可以接受參數，並像一個函數一樣執行
```js
add_two = lambda {
  |n| n + 2
}
```
```js
add_two = -> (n) {
  n + 2
}
```
## Block & Lambda 的差異
```js
add_two_proc = Proc.new {|n| n + 2} 
add_two_lambda = lambda {|n| n + 2}

p add_two_proc.call(1,2,3) //1帶進去加上2，會等於三，後面就算有引數也只會忽略
p add_two_lambda.call(1,2,3) //error (因為引數只需要一個，但是帶了三個，所以報錯)
```

## 實體方法 & 類別方法
* 實體方法
```js
class Cat
  def say_hello
    puts "hi"
  end
end 

kitty = Cat.new
kitty.say_hello
```

* 類別方法
```js
class Cat
  def self.say_hello
    puts "hi"
  end
end 

Cat.say_hello
```

## 實體變數 & 類別變數
* @ 實體變數
  * 從外面取用實體變數的方法
```js
class Cat
  def initialize(name)
    @name = name
  end

  def say_my_name
    return @name
  end
end

kitty = Cat.new('kitty')
puts kitty.name // undefined method 'name'

```
因為在 Ruby 裡，很常會省略大括號、小括號，所以這裡的 `kitty.name`是 `kitty.name()` 呼叫 `.name`這個方法，但是因為程式碼裡面沒有，所以會報錯。

在裡面新增一個 name 方法，就可以使程式碼正常印出 kitty 啦

```js
class Cat
  def initialize(name)
    @name = name
  end

  def say_my_name
    return @name
  end

  def name  //getter
    return @name
  end
end

kitty = Cat.new('kitty')
puts kitty.name // kitty
```

接著，用屬性設定的方式，去改掉裡面的內容
```js
class Cat
  def initialize(name)
    @name = name
  end

  def say_my_name
    return @name
  end

  def name  //getter
    return @name
  end
end

kitty = Cat.new('kitty')
puts kitty.name
puts kitty.name = "nancy" // undefined method 'name'
```

一樣會報錯的原因是因為 `kitty.name = "nancy"`，這句話也省略的小括號：`kitty.name = ("nancy")`

`name=()` 是一個方法，而 nancy 是它的參數，程式碼裡面沒有 `name=()` 這個方法，所以自然會報錯

```js
class Cat
  def initialize(name)
    @name = name
  end

  def say_my_name
    return @name
  end

  def name  //getter
    return @name
  end

  def name=(new_name)  //setter
     @name = new_name
  end
end

kitty = Cat.new('kitty')
puts kitty.name
puts kitty.name = "nancy" // nancy
```

解決以上的複雜化
```js
class Cat
  attr_reader :name
  attr_writer :name
  def initialize(name)
    @name = name
  end

  def say_my_name
    return @name
  end
end

kitty = Cat.new('kitty')
puts kitty.name
puts kitty.name = "nancy" // nancy
```

再更精簡化：
```js
class Cat
  attr_accessor :name
  def initialize(name)
    @name = name
  end

  def say_my_name
    return @name
  end
end

kitty = Cat.new('kitty')
puts kitty.name
puts kitty.name = "nancy" // nancy
```

* @@ 類別變數 
```js
class Cat
  @@count = 0
  
  def initialize
    @@count += 1
  end

  def self.counter
    return @@count
  end
end 

5.times { Cat.new }
p Cat.counter
```

## 開放類別
* 兩個同名的類別存在著，並不會報錯或是覆蓋，而是裡面的方法可以通用，這是 Ruby 特有的
```js
class Cat 
  def hello
  end
end 

class Cat 
  def hi
  end
end 

kitty = Cat.new
kitty.hello
kitty.hi
```

開放類別可以使用在 Ruby 裡內建的功能，例如：string
```js
class String
  def say_hello
    "hi#{self}"
  end
end

puts "helena".say_hello // hi helena 
```

## 封裝
有助於保護程式碼，提高程式碼的可維護性
### Public
  * 公開方法：一般預設時的方法
### Private
  * 前面不能有接受者(.小數點)
`Private`這個方法，沒辦法從外面去取得他的內容，導致會報錯
```js
class Cat
  def say_hello
  end

  private
  def say_hi
  end
end

kitty = Cat.new
kitty.say_hello
kitty.say_hi //Private Method
```

Private Method 的前面不能有接收者，例如：kitty.say_hi，kitty 就是接收者，解決這個問題的方法就是：避免前面有接收者，放在另一個方法的裡面，就可以避免這個問題，同時也讓程式碼正常運作
```js
class Cat
  def say_hello
    say_hi
  end

  private
  def say_hi
  end
end

kitty = Cat.new
kitty.say_hello
```

還有一個方法可以使程式碼正常運作，在前面不能有小數點的情況下：運用 send 方法，將 say_hi 的方法當作參數傳遞進去，這樣也可以使程式碼正常運作
```js
class Cat
  private
  def say_hi
    puts "hi"
  end
end

kitty = Cat.new
kitty.send(:say_hi) //hi
```

### Protected（ 不常用 ）
  * 前面 **不限定** 有接受者(.小數點)

say_hi 的前面有加小數點，是可以正常運作的
```js
class Cat
  def say_hello
    self.say_hi
  end

  protected
  def say_hi
  end
end

kitty = Cat.new
kitty.say_hello
```

## 繼承
可以讓程式碼重複利用及擴展

一個個類別使用繼承後，Dog 跟 Cat 都可以使用 Animal 的 speak 方法，同時還擁有自己的方法
```js
class Animal
  def speak 
    "Animal speaks"
  end
end

class Dog < Animal
  def
    "wolf"
  end
end

class Cat < Animal
  def
    "meow"
  end
end

dog = dog.new
puts dog.speak //Animal speaks
cat = Cat.new
puts cat.speak //Animal speaks
```

## 模組 Module
模組是一種類似於類別，但不能實例化的方式
* 定義模組：必須是常數（字首大寫），與定義類別一樣

### include
* 新增的是：實體化方法
```js
module Flyable
def fly
  "I can fly"
end

class Cat
  include Flyable
end

cat = Cat.new
cat.fly //實體化方法
```

### extend
* 新增的是：類別方法
```js
class Cat
  extend Flyable
end

Cat.fly //類別方法
```

## 類別與模組差異
* 模組無繼承功能
```js
class Cat < Anmial

end
```
* 模組不能實體化
```js
cat = Cat.New
```

只能用引入的方式載入：
```js
class Cat
  include Flyable
end
```

## 解決同名的方法：namspace
如果有兩個類別，出現撞名的話，可以建立兩個不同的 module，維持程式碼的可讀性
```js
module A
  class Cat
  end
end

module B
  class Cat
  end
end

kitty = A::Cat.new
helena = B::Cat.new //用 ::運算符方式呼叫 module 裡的方法
```