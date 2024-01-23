# Presenter Pattern是什麼
Presenter 是一種設計模式，指在將 views 邏輯從 model 和 controller 中分離出來，可以提高程式碼的可維護性及可讀性

## Presenter 模式實作
### 創建 presenter 類別
在 app/presenters 新建 tweet_presenter.rb，把原本在 view 裡的條件語句抓出來，用方法 `formatted_timestamp` 把條件語句包起來
```js
class TweetPresenter
  include ActionView::Helpers::DateHelper

  def initialize(tweet)
    @tweet = tweet
  end
  
  delegate :user, :body, to: :@tweet
  def formatted_timestamp
    if (Time.zone.now - @tweet.created_at) < 1.day
      time_ago_in_words(@tweet.created_at) + ' ' + 'ago'
    else
      @tweet.created_at.strftime("%^b %e")
    end
  end
end
```
* initialize 方法是期望接收一個 tweet 對象做為參數
* `delegate :user, :body, to: :@tweet` : user 和 body 方法都會委託給 @tweet 對象，這樣代表在 TweetPresenter 調用 user 方法會等於 `@tweet.user`


### 建立 TweetPresenter 的實例
在 DashboardController（在你想要增加提取出來的程式碼頁面） 新增 @tweet_presenters，目的是為了讓等等的程式碼不會過長
```js
class DashboardController < ApplicationController
  before_action :authenticate_user!
  def index 
    @tweets = Tweet.order(created_at: :desc).includes(:user)
    @tweet_presenters = @tweets.map { |tweet| TweetPresenter.new(tweet) }
  end
end
```

### 更新 turbo stream
因為我有使用 turbo stream，用於異步地更新 HTML 元素，所以目前的 app/views/dashboard/index.html.erb 的樣子長這樣：
```js
<%= turbo_frame_tag "tweets" do %>
  <%= render partial:'tweets/tweet', collection: @tweet_presenters, as: :tweet_presenter %>
<% end %>
```
* `collection: @tweet_presenters, as: :tweet_presenter`: `collection: @tweet_presenters` 則是指定了要呈現的數據集合，即 @tweet_presenters 中的每個元素都會被作為一條數據進行處理。`as: :tweet_presenter` 則指定了在局部視圖中可以使用的局部變數的名稱

:::danger note
@tweet_presenters 對應上剛剛在 dashboard.controller 建立的實體變數
:::

最後在渲染的地方修改成 tweet_presenter
```js
<div class="py-3 px-2 tweet-hover d-flex">
  <div class="flex-shrink-0">
    <%= avatar(current_user, size: { x: 48, y: 48 }) %>
  </div>
  <div class="flex-grow-1 ms-3">
    <% if defined?(tweet_presenter) %>
      <span class="fw-semibol text-dark"><%= tweet_presenter.user.name %></span>
      <span class="text-black-50">
        @<%= tweet_presenter.user.username %> ·
        <%= tweet_presenter.formatted_timestamp %>
      </span>
      <p class="mb-0 text-body-secondary"><%= tweet_presenter.body %></p>
    <% end %>
  </div>
</div>
```

:::danger note
使用 `<% if defined?(tweet_presenter) %>` 的原因是為了確保在引用 tweet_presenter 時它已經被賦值，**沒加的話可能會報錯**
:::

因為有搭配 turbo stream 的關係，所以要在create.turbo_stream.erb（turbo stream 創建一個推文進行異步更新的頁面） 引入 TweetPresenter
```js
<%= turbo_stream.prepend "tweets" do %>
  <%= render partial: "tweets/tweet", locals: { tweet_presenter: TweetPresenter.new(@tweet) } %>
<% end %>
```