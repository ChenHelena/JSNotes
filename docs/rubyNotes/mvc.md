---
description: 什麼是 mvc
---

# 什麼是 mvc ？
Rails 專案是採用 Model、View、Controller（簡稱 MVC）的方式設計的。MVC 的特性讓專案在開發的過程中，各個模組權責分離，也較好維護或調整

* Model：掌管資料，但 **本身並不是資料庫** 使用 ORM （Object Relationship Mapping）轉譯 SQL 跟資料庫溝通拿資料

* Controller：就像是櫃檯，來處理接收到的訊息請 View 或是 Model 工作，算是滿重要的中控流程中心

* View：負責資料顯示，往往是 user 看到的畫面，可以是網頁、手機應用程式的介面，或者其他 user 互動的形式。它只是將 Model 中的資料呈現給 user，並接收來自 user 的輸入

## 快速生成 rails 網頁
在 rails 中，scaffold 是生成器命令，用於一次性生成一個完整的 CRUD（Create, Read, Update, Delete）模型

以下是 scaffold 命令的基本用法：
```js
rails generate scaffold user name:string email:string tel:string
```
可以簡略成：
```js
rails g scaffold user name email tel
```
除了 generate 可以省略之外，因為 string 是常見的型別，所以也可省略

## 什麼是 migrate ?
用來描述資料表，當你需要新增、刪除或修改數據庫表格的結構時，會使用到 migrate，而 `rails db:migrate` 是一個 Rails 的命令，可以讓資料表具現化出來
* 描述資料表
* 歷史紀錄（版本控制）