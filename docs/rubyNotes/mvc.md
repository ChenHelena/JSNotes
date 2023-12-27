---
description: 什麼是 mvc
---
# 建立 ruby 專案
可以在 new 前面加上版本號，那這個專案就會是使用這個版本，而在專案名稱後面加上參數，就可以幫你在建立專案的時候，順便把 Webpacker 相關的套件以及設定一次搞定

```js
rails _6.1.4.6_ new ruby-demo --webpack
```
開啟 rails 的環境
```js
rails sever
```

## 什麼是 mvc ？
Rails 專案是採用 Model、View、Controller（簡稱 MVC）的方式設計的。MVC 的特性讓專案在開發的過程中，各個模組權責分離，也較好維護或調整

* Model：掌管資料，但 **本身並不是資料庫** 使用 ORM （Object Relationship Mapping）轉譯 SQL 跟資料庫溝通拿資料

* Controller：就像是櫃檯，來處理接收到的訊息請 View 或是 Model 工作，算是滿重要的中控流程中心

* View：負責資料顯示，往往是 user 看到的畫面，可以是網頁、手機應用程式的介面，或者其他 user 互動的形式。它只是將 Model 中的資料呈現給 user，並接收來自 user 的輸入

## 快速生成 rails 網頁
在 rails 中，scaffold 是生成器命令，用於一次性生成一個完整的 CRUD（Create, Read, Update, Delete）模型

以下是 scaffold 命令的基本用法：
```js
rails generate scaffold User name:string email:string tel:string
```
可以簡略成：
```js
rails g scaffold User name email tel
```
除了 generate 可以省略之外，因為 string 是常見的型別，所以也可省略

再新創第二個 model：
```js
rails g scaffold Post title content:text is_availible:boolean user:references
```
這裡的 `user:references` 等於 `user_id:integer`，這樣的語法是為了建立『 關聯 』，`user:references` 會自動在 post 模型中建立 `bolongs_to:user` 的關聯，這樣就可以在 post 模型中直接連結 user 模型（例如：post.user）


## 什麼是 migrate ?
用來描述資料表，當你需要新增、刪除或修改數據庫表格的結構時，會使用到 migrate，而 `rails db:migrate` 是一個 Rails 的命令，可以讓資料表具現化出來
* 描述資料表
* 歷史紀錄（版本控制）

## 下拉選單（collection_select）

```js
 <%= form.collection_select(:user_id, User.all, :id, :name) %>
```

## N+1 問題
拿 post 當例子，在這個頁面裡，每新增一筆資料，就會撈一次資料，這樣子會產生額外的效能問題

新增三筆資料，但卻跑了四次
```
Post Load (0.2ms)  SELECT "posts".* FROM "posts"
  ↳ app/views/posts/index.html.erb:17
  User Load (0.4ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  ↳ app/views/posts/index.html.erb:22
  User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  ↳ app/views/posts/index.html.erb:22
  CACHE User Load (0.0ms)  SELECT "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
```
includes 是 Rails **ActiveRecord** 中的一個方法，用於在檢索數據時預先載入關聯記錄，以避免 N+1 查詢問題

解決辦法：在 post_controller.rb 裡，
```js
def index
  @posts = Post.all.includes(:user)
end
```

回到終端機查看，這樣一來，就可以解決效能上的問題，一口氣把所有資料都撈出來
```
Post Load (0.3ms)  SELECT "posts".* FROM "posts"
  ↳ app/views/posts/index.html.erb:17
  User Load (0.8ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (?, ?)  [["id", 1], ["id", 2]]
```