---
description: turbo-stream、presenter、likes、retweets、bookmaerks
---
# Turbo Stream 如何切換頁面
## 以 bookmarks 當例子
以 bookmarks 舉例來說，在我做的專案裡面，會有兩處有 bookmarks 的按鈕，但是兩個是不同頁面，卻是用同一個紀錄，在第一個頁面按下 bookmarks 按鈕，第二個頁面也要呈現 收藏 過的實心按鈕

例如這兩個頁面：
```js
// _show_status.html.erb
<div class="d-flex justify-content-evenly mt-3">
  <%= link_to '#', type:'button', class:'text-decoration-none views d-inline-flex align-items-center' do %>
    <span class="bg-icon-blue rounded-circle p-2 d-inline-flex fs-5">
      <i class="fa-solid fa-chart-simple"></i>
    </span>
  <% end %>
  <%= link_to tweet_path(tweet_presenter.tweet), type:'button', class:'text-decoration-none replies d-inline-flex align-items-center' do %>
    <span class="bg-icon-blue rounded-circle p-2 d-inline-flex fs-5">
      <i class="fa-regular fa-comment-dots"></i>
    </span>
  <% end %>
  <%= link_to tweet_presenter.tweet_retweet_url, data:{ 'turbo-method':tweet_presenter.turbo_data_retweet_method }, class:"text-decoration-none
    retweets d-inline-flex align-items-center #{tweet_presenter.tweet_retweeted_by_current_user ? 'text-green' : '' }" do %>
    <span class="bg-icon-green rounded-circle p-2  d-inline-flex fs-5">
      <i class="fa-solid fa-retweet"></i>
    </span>
  <% end %>
  <%= link_to tweet_presenter.tweet_like_url, data:{ 'turbo-method':tweet_presenter.turbo_data_like_method },class: "text-decoration-none likes d-inline-flex align-items-center #{tweet_presenter.tweet_liked_by_current_user ? ' text-pink' : '' }" do %>
    <span class="bg-icon-red rounded-circle p-2  d-inline-flex fs-5">
      <i class="<%= tweet_presenter.like_heart_icon %>"></i>
    </span>
  <% end %>
  <%= link_to tweet_presenter.tweet_bookmark_url, type:'button',data:{'turbo-method':tweet_presenter.turbo_data_bookmark_method }, class:"text-decoration-none bookmark d-inline-flex align-items-center #{tweet_presenter.tweet_bookmarked_by_current_user ? 'text-primary' : '' }" do %>
    <span class="bg-icon-blue rounded-circle p-2 d-inline-flex fs-5">
      <i class="<%= tweet_presenter.bookmark_icon %>"></i>
    </span>
  <% end %>
</div>
```

```js
//  _tweet.html.erb
<div class="d-flex justify-content-between mt-3">
  <%= link_to tweet_path(tweet_presenter.tweet), type:'button', data:{ 'ignore-click': 'true' }, class:'text-decoration-none views d-inline-flex align-items-center' do %>
    <span class="bg-icon-blue rounded-circle p-2 d-inline-flex">
      <i class="fa-solid fa-chart-simple"></i>
    </span>  
    <span><%= tweet_presenter.views_count %></span>
  <% end %>
  <%= link_to tweet_path(tweet_presenter.tweet),data: {turbo_frame: "_top" } , type:'button', class:'text-decoration-none
    replies d-inline-flex align-items-center' do %>
    <span class="bg-icon-blue rounded-circle p-2 d-inline-flex">
      <i class="fa-regular fa-comment-dots"></i>
    </span>
    <span>Replies</span>
    <% end %>
  <%= link_to tweet_presenter.tweet_retweet_url, data:{ 'turbo-method':tweet_presenter.turbo_data_retweet_method,'ignore-click': 'true' }, class:"text-decoration-none retweets d-inline-flex align-items-center #{tweet_presenter.tweet_retweeted_by_current_user ? 'text-green' : ''}" do %>
  <span class="bg-icon-green rounded-circle p-2  d-inline-flex">
    <i class="fa-solid fa-retweet"></i>
  </span>
    <%= tweet_presenter.retweets_count %>
  <% end %>

  <%= link_to tweet_presenter.tweet_like_url, data:{  'turbo-method':tweet_presenter.turbo_data_like_method, 'ignore-click':'true' }, class: "text-decoration-none likes d-inline-flex align-items-center #{tweet_presenter.tweet_liked_by_current_user ? ' text-pink' : ''}" do %>
    <span class="bg-icon-red rounded-circle p-2  d-inline-flex">
      <i class="<%= tweet_presenter.like_heart_icon %>"></i>
    </span>
    <%= tweet_presenter.likes_count %>
  <% end %>

  <%= link_to tweet_presenter.tweet_bookmark_url, type:'button', data:{ 'turbo-method':tweet_presenter.turbo_data_bookmark_method, 'ignore-click': 'true' }, class:"text-decoration-none bookmark d-inline-flex align-items-center #{tweet_presenter.tweet_bookmarked_by_current_user ? 'text-primary' : ''}" do %>
  <span class="bg-icon-blue rounded-circle p-2 d-inline-flex">
    <i class="<%= tweet_presenter.bookmark_icon %>"></i>
  </span>
    <span><%= tweet_presenter.bookmark_text %></span>
  <% end %>
</div>
```
## Render 
如果我要在任何地方顯示哪個畫面，就用 render 來顯示
```js
<%= render partial: "tweets/show_status", locals: { tweet_presenter: @tweet_presenter } %>
```

## 加入參數
如果想要區分兩個頁面的顯示方式，也需要在 html.erb 的其中一個頁面傳入不同的參數(source: "tweet_show")

```js
//_show_status.html.erb
<%= link_to tweet_presenter.tweet_bookmark_url(source: "tweet_show"), type:'button',data:{'turbo-method':tweet_presenter.turbo_data_bookmark_method }, class:"text-decoration-none bookmark d-inline-flex align-items-center #{tweet_presenter.tweet_bookmarked_by_current_user ? 'text-primary' : '' }" do %>
  <span class="bg-icon-blue rounded-circle p-2 d-inline-flex fs-5">
    <i class="<%= tweet_presenter.bookmark_icon %>"></i>
  </span>
<% end %>
```
* tweet_presenter.tweet_bookmark_url : 這個方法我是寫在 presenter 裡面，所以也要在裡面設置預設的參數 (source: "tweet_card")

## 連結 turbo-stream
但是因為有連結 turbo-stream，所以要在 turbo-stream 頁面寫些條件語句，套用在 create 及 destroy，同時也可以直接在 likes 及 retweets 頁面直接替換這個方法，就可以方便頁面切換啦
```js
//create.turbo_stream.erb
<% target_id = if params[:source] == "tweet_show" 
                "tweet_status"
              else 
                dom_id(@tweet)
              end
partial_name = if params[:source] == "tweet_show"
                "tweets/show_status"
              else 
                "tweets/tweet"
              end
%>
<%= turbo_stream.replace target_id do %>
  <%= render partial: partial_name, locals:{ tweet_presenter: TweetPresenter.new(tweet: @tweet, current_user: current_user) } %>
<% end %>
```