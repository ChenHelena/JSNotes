---
description: rack、middleware、sinatra
---

# 什麼是 Rack ？
提供一個介面，用於連接網站伺服器、應用程式、開發框架

## Rack 期望的規格
提供一個能夠回應 call 方法的物件，並且回傳一個包含以下三個元素的陣列：

➀ HTTP 狀態( number )

➁ HTTP Header ( Hash )

➂ HTTP Body ( array )

## Rack 的應用程式

先開一個 rack 資料夾，裡面放入 config.ru 的檔案

```js
mkdir rack
cd rack/
touch config.ru
```

config.ru 裡面所需的資料
```jsx title="config.ru" 
run Proc.new { |env|
    [
      200, // HTTP 狀態
      { "content-type" => "text/html" }, // HTTP Header
      ["Hello, Rack"] //HTTP Body
    ]
}
```
接著，回到終端機打上 `rackup`，就可以試著在瀏覽器打上，localhost:9292，會出現 Hello, Rack 那就是成功啦

這是第二種方法：類別方法，簡單來說，只要有 call 方法的物件，就可以使 rack 動起來

```js
class App
  def call(dev)
    [
      200, // HTTP 狀態
      { "content-type" => "text/html" }, // HTTP Header
      ["Hello, Rack"] //HTTP Body
    ]
  end
end

run App.new
```

## middleware
Rack 提供了一個middleware的概念，通常用於處理 HTTP 請求和回應的不同階段，這些階段包括請求到達應用程式之前以及回應離開應用程式之後

```js
class Backdoor
  def initialize(app, who = "no one")
    @app = app
    @who = who
  end
  def call(env)
  //處理請求之前的邏輯
    status, headers, body = @app.call(env)
    body << "hacked by #{@who}"
  //處理請求之後的邏輯
    [status, headers, body]
  end
end

use Backdoor, "helena"

run Proc.new { |env|
    [
      200, // HTTP 狀態
      { "content-type" => "text/html" }, // HTTP Header
      ["Hello, Rack"] //HTTP Body
    ]
}
```


## Sinatra
以 Rack 為基底的輕量級框架
輸入 `gem install sinatra`，開啟資料夾創建檔案 sinatra.rb

```jsx title="sinatra.rb"
require 'sinatra'

get '/' do
  "Hello World"
end
```

`ruby sinatra.rb` 這樣就可以使用 sinatra 開啟一個檔案，但因為每當要更新一個資料，他不會自動更新，必須先中斷程式，在重新跑一次，這樣的效率太低，所以要下載 `gem install sinatra-contrib` 並且在檔案裡輸入：

```jsx title="sinatra.rb"
require 'sinatra'
require 'sinatra/reloader' if development? //只有在開發環境中或測試階段可以重新整理

get '/' do
  "Hello World"
end
```

這樣不管要更新什麼，按下重新整理後，會自動更新

最後在下載 `gem install puma` ，這是效能更好的伺服器

### 帶參數

```js
require 'sinatra'
require 'sinatra/reloader' if development? 

get '/cats' do
  "Hello cats"
end

get '/cats/:id' do
  "Hello #{params[:id]} cats"
end
```

### 整理 html
如果東西都放在同一頁，勢必會很亂，所以需要去整理它，這裡使用 erb 來做整理

```jsx title="sinatra.rb"
require 'sinatra'
require 'sinatra/reloader' if development? 

get '/' do
  @name = 'helena'
  erb :index
  bmi = 
end
```

需要先建立一個 views 資料夾，裡面放入 index.erb 的檔案，這樣就可以把畫面顯示的資料放在同個檔案裡

```jsx title="index.erb"
<h1>hello <%= Time.now %></h1> //ruby 語法
<h1>hello <%= @name %></h1> //實體變數
```

### 區域變數
建立一個表單，可以填入資料的表單 form.erb 的檔案，一樣放在 views 資料夾裡（ erb 的檔案相當於 html )

```jsx title="form.erb"

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  
  <form action="/form" method="POST">
    height:<input type="text" name="height">cm<br/>
    weight:<input type="text" name="weight">kg<br/>
    <input type="submit" value="提交">
  <form/>

</body>
</html>
```
回到 sinatra.rb 檔案，新增 POST 方法

```jsx title="sinatra.rb"
require 'sinatra'
require 'sinatra/reloader' if development? 

get '/' do
  @name = 'helena'
  erb :index
  bmi = 
end

get '/form' do //尚未提交前看到的畫面
  erb :form
end

post '/form' do //提交後看到的畫面（目前尚未建立）
  erb :result
end
```

接著，再到 views 資料夾新增 result.erb ，也就是按下提交按鈕會看到的畫面

```jsx title="result.erb"
BMI result 
```

結果會出現『 BMI result 』這段文字！
但我的預期結果是：想要在按下提交按鈕後，把 input 裡面的文字取出來做計算值顯示在畫面上，這時，需要回到 sinatra.rb 的檔案，將數值做區域變數，回傳到 result.erb，而提交的數值會存在於 params 這個 hash 裡面，用物件取值的方式，把值取出來

```jsx title="sinatra.rb"
require 'sinatra'
require 'sinatra/reloader' if development? 

get '/' do
  @name = 'helena'
  erb :index
  bmi = 
end

get '/form' do //尚未提交前看到的畫面
  erb :form
end

post '/form' do //提交後看到的畫面
  h = params[:height].to_f / 100 
  w = params[:weight].to_f //字串轉換成數字（含小數點）
  bmi = w / (h * h)
  erb :result, locals:{result: bmi}
end
```
這裡的物件取值的 height 跟 weight 的名稱，就是一開始 form.erb 的檔案裡面 name 的屬性名稱

建立好 bmi 的值之後，再回到 result.erb ，也就是按下提交按鈕後會出現的結果頁面

```jsx title="result.erb"
BMI result <%= result %> //區域變數
```
這裡使用區域變數，相對於實體變數（ex. @bmi ）彈性較大