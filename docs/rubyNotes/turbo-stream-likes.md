---
description: turbo-stream、presenter、likes
---

# Turbo Stream 處理按讚行為
## 建立 likes model

建立 likes 的資料表，與 user 和 tweet 關聯
```js
rails g model Like user:reference tweet:reference
```
並且在 likes 的資料表中，建立具有唯一值的 tweet_id、 user_id

```js
rails generate migration add_unique_constraint_to_likes
```

```js
// migrate
class AddUniqueConstraintToLikes < ActiveRecord::Migration[6.1]
  def change
    add_index :likes, [:user_id, :tweet_id], unique: true
  end
end
```

```js
rails db:migrate
```

## 建立關聯
`validates :user_id, uniqueness:{ scope: :tweet_id }`：確保在like model 中 user_id 對應的 tweet_id 只能有一個，**也就是說一個用戶在一個推文裡只能按一次讚**
```js
// model/like.rb
class Like < ApplicationRecord
  belongs_to :tweet
  belongs_to :user

  validates :user_id, uniqueness:{ scope: :tweet_id }
end
```

表示一則推文可以擁有很多讚
```js
// model/tweet.rb
class Tweet < ApplicationRecord
  has_many :likes
end
```

表示一個用戶可以擁有很多讚，也可以透過點讚獲取到他（用戶）點過誰的讚
```js
// model/user.rb
class User < ApplicationRecord
  has_many :likes, dependent: :destroy
  has_many :liked_tweets, through: :likes, source: :tweet
end
```

## 畫面呈現

關聯都建立完後，可以來處理 views 呈現的畫面了，我是使用 turbo-stream 來處理我的局部更新，且使用 presenter 把 views 畫面抽取出來管理

* 這是用來渲染推文的部分視圖，渲染的頁面就是 `tweets/_tweet.html.erb` 檔案

```js
<%= turbo_frame_tag "tweets" do %>
  <%= render partial:'tweets/tweet', collection: @tweet_presenters, as: :tweet_presenter %>
<% end %>
```

* collection: 這是一個集合，使用 @tweet_presenters，對每個元素執行一次 tweets/tweet 這個部份視圖的更新，那為什麼是 @tweet_presenters 呢？

  是因為我在 controller 設置了方法：
  ```js
  class DashboardController < ApplicationController
    before_action :authenticate_user!
    def index 
      @tweets = Tweet.order(created_at: :desc).includes(:user)// 解決 n+1 問題
      @tweet_presenters = @tweets.map { |tweet| TweetPresenter.new(tweet) }
      //將每個推文轉換為對應的 presenter
    end
  end
  ```

:::danger note
`@tweets.map { |tweet| TweetPresenter.new(tweet) }` : 使用 TweetPresenter 類別的 new 方法創建一個新的 TweetPresenter 物件，將當前的推文 tweet 作為參數傳遞給 TweetPresenter 的初始化方法（initialize）
:::

再來，檢查使用者對特定推文（當前推文）是否已經點讚，如果使用者已經點讚，則顯示帶有紅色愛心圖標的按鈕，如果使用者尚未點讚，則顯示普通的空心按鈕
```js
// tweets/_tweet.html.erb
<% if tweet_presenter.likes.include?(current_user.likes.find_by(tweet: tweet_presenter.tweet)) %>
  <%= link_to tweet_like_path(tweet_presenter.tweet, tweet_presenter.tweet.likes.find_by(user: current_user)), type:'button', data:{ 'bs-toggle' => 'tooltip' , 'bs-title'=>'Likes','bs-placement'=>'bottom', 'turbo-method' => 'delete' }, class:'text-decoration-none likes d-inline-flex
    align-items-center text-pink' do %>
    <span class="bg-icon-red rounded-circle p-2  d-inline-flex">
      <i class="fa-solid fa-heart"></i>
    </span>
    <%= tweet_presenter.likes.size %>
  <% end %>
<% else %>
  <%= link_to tweet_likes_path(tweet_presenter.tweet), type: 'button', data:{ 'bs-toggle': 'tooltip', 'bs-title': 'Likes','bs-placement': 'bottom', 'turbo-method': 'post' }, class:'text-decoration-none likes d-inline-flex align-items-center' do %>
    <span class="bg-icon-red rounded-circle p-2  d-inline-flex">
        <i class="fa-regular fa-heart"></i>
    </span>
    <%= tweet_presenter.likes.size %>
  <% end %>
<% end %>
```

* 在 TweetPresenter 中使用 `delegate :user, :body, :likes, to: :@tweet` 這樣的程式碼，讓我們可以直接在 TweetPresenter 中訪問 likes（`tweet_presenter.likes`），而不必每次都寫 tweet_presenter.tweet.likes

最後要新增 likes/create.turbo_stream.erb 檔案，使用 Turbo Streams 的方式來實現在畫面上替換掉一個指定 ID 的區塊：
```js
<%= turbo_stream.replace dom_id(@tweet) do %>
  <%= render partial: "tweets/tweet", locals: { tweet_presenter: TweetPresenter.new(@tweet) } %>
<% end %>
```
`dom_id(@tweet)`：這裡的 ＠tweet 會因為找不到 id 而報錯，所以這時候我們要在表單 _tweet.html.erb 加入 id，讓他對應上 `dom_id(@tweet)`

```js
// _tweet.html.erb
<%= turbo_frame_tag dom_id(tweet_presenter.tweet) do %>
  ...
<% end %>
```

:::danger note
`tweet_presenter.tweet` 實際上是 @tweet 為 Tweet 對象，因為 Tweet 是 ActiveRecord 模型，ActiveRecord 模型預設擁有一個主鍵 id， `dom_id(tweet_presenter.tweet)` 將返回這個推文對象的唯一 id，所以才可以使 dom_id(@tweet) 對應上 dom_id(tweet_presenter.tweet)，進而更新按讚功能
:::

## 建立 likes_count 欄位
在 tweets 表中添加一個新的列 likes_count，用於儲存該推文（tweet）獲取的點讚（likes）數量。這樣的用意通常用於提高查詢效率，以避免每次都實時計算點贊的數量
```js
rails g migration add_likes_count_to_tweets likes_count:integer
```

新增 migation 的方法
```js
class AddLikesCountToTweets < ActiveRecord::Migration[6.1]
  def up
    add_column :tweets, :likes_count, :integer, null: false, default: 0

    Tweet.find_each do |tweet| 
      tweet.update! likes_count: tweet.likes.size
    end
  end

  def down
    remove_column :tweets, :likes_count
  end
end
```

* up 方法： 建立一個欄位 likes_count 為整數，不能為空值且預設值為 0，接著，通過 `Tweet.find_each` 迭代每篇推文，並使用 update! 方法將 likes_count 設置為每則推文的實際獲取的按讚數量

* down 方法：用於還原 migration 的 up 方法，如果你需要刪除 likes_count 欄位，則可以使用 `rails db:rollback` 就會調用 down 方法刪除欄位

```js
rails db:migrate
```
在要顯示按讚數量的 view 中更新：
```js
<%= tweet_presenter.likes_count %>
```

如果使用 Presenter 方法記得要在頁面加入 `:likes_count`，這樣才可以在 TweetPresenter 的實例上調用 likes_count
```js
delegate :user, :body, :likes, :likes_count, to: :@tweet
```


## n+1 問題

依據上面的程式碼後台會呈現像這樣，一筆一筆的查詢，這樣會造成速度變慢，而影響使用者體驗
```js
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (2.6ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 28], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (1.4ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 27], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (2.0ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 26], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (27.4ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 25], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (1.9ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 24], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (2.5ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 23], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (27.7ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 22], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (1.6ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 21], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (1.7ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 20], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (27.3ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 19], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (1.5ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 18], ["LIMIT", 1]]
14:15:45 web.1     |   ↳ app/views/tweets/_tweet.html.erb:34
14:15:45 web.1     |   Like Load (1.4ms)  SELECT "likes".* FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 17], 
```

頁面指出在 _tweet.html.erb:34 行會造成一直查詢的問題，接著我們來更改 _tweet.html.erb 頁面。在當前使用者有無按讚這篇推文的讚，有的話呈現實心的愛心，沒有的話呈現空心的愛心
```js
<% if current_user.liked_tweets.include?(tweet_presenter.tweet) %>
  <%= link_to tweet_like_path(tweet_presenter.tweet, current_user.likes.find_by(tweet: tweet_presenter.tweet)), type:'button', data:{ 'bs-toggle' => 'tooltip' , 'bs-title'=>'Likes','bs-placement'=>'bottom', 'turbo-method' => 'delete' }, class:'text-decoration-none likes d-inline-flex
    align-items-center text-pink' do %>
    <span class="bg-icon-red rounded-circle p-2  d-inline-flex">
      <i class="fa-solid fa-heart"></i>
    </span>
    <%= tweet_presenter.likes_count %>
  <% end %>
<% else %>
  <%= link_to tweet_likes_path(tweet_presenter.tweet), type: 'button', data:{ 'bs-toggle': 'tooltip', 'bs-title': 'Likes','bs-placement': 'bottom', 'turbo-method': 'post' }, class:'text-decoration-none likes d-inline-flex align-items-center' do %>
    <span class="bg-icon-red rounded-circle p-2  d-inline-flex">
        <i class="fa-regular fa-heart"></i>
    </span>
    <%= tweet_presenter.likes_count %>
  <% end %>
<% end %>
```

* `current_user.liked_tweets`: 可以在當前使用者抓取按讚的推文，原因是因為在 user model 有建立關聯 
  ```
  class User < ApplicationRecord
    has_many :liked_tweets, through: :likes, source: :tweet
  end
  ```

* 因為使用 Presenter 方法所以要在 Presenter 頁面新增 `@current_user = current_user`，使得在 Presenter 的實例中可以訪問 current_user
```js
//tweet_presenter
def initialize(tweet:, current_user:)
  @tweet = tweet
  @current_user = current_user
end
```

並且在 turbo-stream 建立的 TweetPresenter 的實例中也把 current_user 傳遞進去，確保在 TweetPresenter 中可以使用它
```js
//create.turbo_stream.erb
<%= turbo_stream.replace dom_id(@tweet) do %>
  <%= render partial: "tweets/tweet", locals: { tweet_presenter: TweetPresenter.new(tweet: @tweet, current_user: current_user) } %>
<% end %> 
```

這麼一來，更新過後來查看後台數據，整個頁面跑快了不少唷
```js
User Load (0.9ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 ORDER BY "users"."id" ASC LIMIT $2  [["id", 1], ["LIMIT", 1]]
15:59:20 web.1     |   ↳ app/controllers/application_controller.rb:2:in `block in <class:ApplicationController>'
15:59:20 web.1     |   Tweet Load (0.9ms)  SELECT "tweets".* FROM "tweets" WHERE "tweets"."id" = $1 LIMIT $2  [["id", 295], ["LIMIT", 1]]
15:59:20 web.1     |   ↳ app/controllers/likes_controller.rb:19:in `tweet_id'
15:59:20 web.1     |   TRANSACTION (1.1ms)  BEGIN
15:59:20 web.1     |   ↳ app/controllers/likes_controller.rb:4:in `create'
15:59:20 web.1     |   Like Exists? (1.2ms)  SELECT 1 AS one FROM "likes" WHERE "likes"."user_id" = $1 AND "likes"."tweet_id" = $2 LIMIT $3  [["user_id", 1], ["tweet_id", 295], ["LIMIT", 1]]
15:59:20 web.1     |   ↳ app/controllers/likes_controller.rb:4:in `create'
15:59:20 web.1     |   Like Create (1.4ms)  INSERT INTO "likes" ("tweet_id", "user_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["tweet_id", 295], ["user_id", 1], ["created_at", "2024-01-30 07:59:20.125429"], ["updated_at", "2024-01-30 07:59:20.125429"]]
15:59:20 web.1     |   ↳ app/controllers/likes_controller.rb:4:in `create'
15:59:20 web.1     |   Tweet Update All (2.4ms)  UPDATE "tweets" SET "likes_count" = COALESCE("likes_count", 0) + $1 WHERE "tweets"."id" = $2  [["likes_count", 1], ["id", 295]]
```

這樣做雖然跑快了不少，但還是 loading 的比平常慢，所以我們換個方式做

在 Tweet 模型中，建立一個 has_many 關聯，通過 likes 表檢索喜歡的用戶
```js
class Tweet < ApplicationRecord
  has_many :liked_users, through: :likes, source: :user
end
```

使用 'includes' 預先加載相關數據（user 和 liked_users），避免 N+1 查詢
```js
class DashboardController < ApplicationController
  def index 
    @tweets = Tweet.includes(:user, :liked_users).order(created_at: :desc)
  end
end
```

`tweet_liked_by_current_user ||=` : 用於確保運算結果只計算一次。如果 tweet_liked_by_current_user 已經有值，就不再執行後面的運算
```js
class TweetPresenter
  def tweet_liked_by_current_user
    tweet_liked_by_current_user ||= tweet.liked_users.include?(@current_user)
  end
end
```
用這個方法可以看到後台數據呈現：
```js
 Like Load (6.7ms)  SELECT "likes".* FROM "likes" WHERE "likes"."tweet_id" IN ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14, $15, $16, $17, $18, $19, $20, $21, $22, $23, $24, $25, $26, $27, $28, $29, $30, $31, $32, $33, $34, $35, $36, $37, $38, $39, $40, $41, $42, $43, $44, $45, $46, $47, $48, $49, $50, $51, $52, $53, $54, $55, $56, $57, $58, $59, $60, $61, $62, $63, $64, $65, $66, $67, $68, $69, $70, $71, $72, $73, $74, $75, $76, $77, $78, $79, $80, $81, $82, $83, $84, $85, $86, $87, $88, $89, $90, $91, $92, $93, $94, $95, $96, $97, $98, $99, $100, $101, $102, $103, $104, $105, $106, $107, $108, $109, $110, $111, $112, $113, $114, $115, $116, $117, $118, $119, $120, $121, $122, $123, $124, $125, $126, $127, $128, $129, $130, $131, $132, $133, $134, $135, $136, $137, $138, $139, $140, $141, $142, $143, $144, $145, $146, $147, $148, $149, $150, $151, $152, $153, $154, $155, $156, $157, $158, $159, $160, $161, $162, $163, $164, $165, $166, $167, $168, $169, $170, $171, $172, $173, $174, $175, $176, $177, $178, $179, $180, $181, $182, $183, $184, $185, $186, $187, $188, $189, $190, $191, $192, $193, $194, $195, $196, $197, $198, $199, $200, $201, $202, $203, $204, $205, $206, $207, $208, $209, $210, $211, $212, $213, $214, $215, $216, $217, $218, $219, $220, $221, $222, $223, $224, $225, $226, $227, $228,
```

我們使用 includes 預先加載了 liked_users，這就避免了 N+1 查詢問題，提高了性能，loading 的速度也是正常的