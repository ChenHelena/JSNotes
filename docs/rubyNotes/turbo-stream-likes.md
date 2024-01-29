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