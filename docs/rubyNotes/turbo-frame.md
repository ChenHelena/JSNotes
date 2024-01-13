---
description: turbo-frame、hotwire
---

# Turbo Frames 是什麼？
Turbo Frames 是 Hotwire 框架中的一個組件，它用於實現 Turbo 模式下的**局部更新**，而不是整個頁面。這樣可以實現更流暢的用戶體驗，而無需重新加載整個頁面

## Turbo Frames 的流程

### **第一步：**
在 Gemfile 中確認 turbo-rails 的存在：
```js
gem 'turbo-rails', '~> 1.5'
```

```js
bundle install
```

### **第二步：**
在提交表單的地方使用 turbo_frame_tag：
```js
<%= turbo_frame_tag @tweets do %>
  <%= form_with model: Tweet.new, url: tweets_path,class:'my-3' do |f| %>
    <%= f.text_area :body, class:'form-control tweet-input fs-5', placeholder:"what's happening?" %>
    <div class="d-flex justify-content-end mt-2">
      <%= f.submit 'tweet' , class:'btn btn-primary text-white rounded-pill' %>
    </div>
  <% end %>
<% end %>
```
:::danger note
 這裡的 @tweets 是 dashboard_controller 中寫的方法
:::
```js
// dashboard_controller.rb
class DashboardController < ApplicationController
  before_action :authenticate_user!
  def index 
    @tweets = Tweet.order(created_at: :desc)
  end
end
```

### **第三步：**
建立 create.turbo_stream.erb 用於提交表單後的渲染：
```js
<%= turbo_stream.append "tweets" do %>
  <%= render partial: "tweets/tweet", local: { tweet: @tweet } %>
  //<%= render @tweets %> 也可這樣撰寫，rails 會自動渲染 _tweet.html.erb 檔案
<% end %>
```
:::danger note
這裡的 "tweets" 是 Turbo Frame 的 ID，確保與表單中一致
:::

渲染頁面加入 Turbo Frame 的 ID
```js
<div id="tweets">
  <%= render @tweets %>
</div>
```

這樣就可以正確連結上嚕，總而言之，Turbo Frames 使得實現動態和局部更新的 Web 頁面變得更加簡單，減少了對傳統 Ajax 和 JavaScript 的依賴！

