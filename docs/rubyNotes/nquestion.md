---
description: n+1 問題所遇到的狀況
---

#  n+1 問題集合
## counter_cache
只有 2 筆資料，但卻對資料庫做了 3 次的查詢，這樣子非常浪費系統的效能，要解決這個問題，可以使用 Rails 提供的 Counter Cache 做法

```js
Started GET "/candidates" for ::1 at 2023-11-29 15:41:07 +0800
Processing by CandidatesController#index as HTML
  Rendering layout layouts/application.html.erb
  Rendering candidates/index.html.erb within layouts/application
  Candidate Load (0.2ms)  SELECT "candidates".* FROM "candidates"
  ↳ app/views/candidates/index.html.erb:14
   (0.4ms)  SELECT COUNT(*) FROM "vote_logs" WHERE "vote_logs"."candidate_id" = ?  [["candidate_id", 4]]
  ↳ app/views/candidates/index.html.erb:17
   (0.3ms)  SELECT COUNT(*) FROM "vote_logs" WHERE "vote_logs"."candidate_id" = ?  [["candidate_id", 8]]
  ↳ app/views/candidates/index.html.erb:17
  Rendered candidates/index.html.erb within layouts/application (Duration: 13.7ms | Allocations: 2685)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 23.3ms | Allocations: 6235)
Completed 200 OK in 25ms (Views: 23.3ms | ActiveRecord: 0.9ms | Allocations: 6629)
```
:::danger note
counter_cache 是 Rails **ActiveRecord** 提供的一項功能，用於在關聯關係中快速獲取關聯記錄的數量，而不需要執行額外的查詢。這在處理關聯 **記錄數量頻繁變動** 的情況下，可以有效提高性能
:::

首先在 Candidate Model 開一個 integer 型態的欄位，名稱叫做 vote_logs_count，我們使用 Migration 來做這件事：
在專案終端機打上 `rails g migration add_counter_to_candidate vote_logs_count:integer`

這時候 migration 的內容長這樣：
```js
// frozen_string_literal: true
class AddCounterToCandidate < ActiveRecord::Migration[6.1]
  def change
    add_column :candidates, :vote_logs_count, :integer
  end
end
```
:::note
在程式碼第一行加入 frozen_string_literal: true 凍結此檔，可以稍微加快往後的讀取速度
:::

執行： `rails db:migrate`

在 VoteLog Model 的 belongs_to 後面加上 counter_cache: true 參數：

```jsx title="vote_log.rb"
class VoteLog < ApplicationRecord
  belongs_to :candidate, counter_cache:true
end
```
:::note
如果想指定其他欄位儲存 counter cache 請改寫為

counter_cache: ‘COLUMN_NAME’
:::

最後在 index 檔案，顯示投票數的地方使用 size 方法：

  * size 方法可以應用在關聯的集合、查詢結果等地方
  * size 方法的使用方式類似於 count 方法
  * size 會儘可能在內存中計算集合的大小，而不去數據庫查詢，這對於已經加載到內存中的集合來說效率更高
  * 如果集合還沒有被加載，或者是一個條件較為複雜的查詢，Rails 可能會選擇使用 count 方法來發送一個查詢到數據庫
```js
<td><%= candidate.vote_logs.size %></td>
```

這樣一來，成功解決 n+1 的問題：只有抓取一次資料
```js
Started GET "/candidates" for ::1 at 2023-11-29 15:41:51 +0800
Processing by CandidatesController#index as HTML
  Rendering layout layouts/application.html.erb
  Rendering candidates/index.html.erb within layouts/application
  Candidate Load (0.1ms)  SELECT "candidates".* FROM "candidates"
  ↳ app/views/candidates/index.html.erb:14
  Rendered candidates/index.html.erb within layouts/application (Duration: 4.0ms | Allocations: 1230)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 11.8ms | Allocations: 4780)
Completed 200 OK in 13ms (Views: 12.4ms | ActiveRecord: 0.1ms | Allocations: 5174)
```

## includes
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

:::danger note
includes 是 Rails **ActiveRecord** 中的一個方法，用於在檢索數據時預先載入關聯記錄，以避免 N+1 查詢問題
:::

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

## with_attached
:::danger note
with_attached 是 **Active Storage** 提供的方法，主要用於在**檢索記錄的同時獲取附件**，避免多次查詢附件表，提高性能
:::

舉例來說，如果你的 Story model 中有一個附加的 cover_image，想要顯示在首頁，可以使用 `with_attached ` 方法來在查詢時加載附件
```js
class PagesController < ApplicationController
  def index
    @stories = Story.published.with_attached_cover_image.order(created_at: :desc).includes(:user)
  end
end
```

使用 with_attached_cover_image 可以在一次查詢中預先載入所有的附件，而不是在後續的循環中逐一查詢附件，避免了 N+1 查詢的性能問題

而 includes(:user) 的作用是預先載入與 Story 關聯的使用者資料，同樣避免了在循環中逐一查詢使用者資訊的問題。這樣一來，在使用 @stories 數據時，已經包含了附件和使用者資訊，而不需要額外的數據庫查詢，提高了性能效率

### Active Storage & ActiveRecord 差異
ActiveRecord 和 Active Storage 都是 Ruby on Rails 中的模組

* ActiveRecord 

  ORM 框架，用於資料庫操作，無需直接編寫 SQL 查詢語句

* Active Storage 
  用於處理文件上傳和存儲，提供了 `has_one_attached` 或 `has_many_attached` 方法，用於將附件與 model 關聯

具體來說，由於圖片檔案通常是通過 Active Storage 進行管理和存儲的，而文字數據則可能是直接存儲在數據庫中的 ActiveRecord，因此它們有不同的處理方式

