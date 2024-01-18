---
description: turbo-frame、hotwire
---

# Turbo Frames 是什麼？
Turbo Frames 是 Hotwire 框架中的一個組件，它用於實現 Turbo 模式下的**局部更新**，而不是整個頁面。這樣可以實現更流暢的用戶體驗，而無需重新加載整個頁面

## Turbo Frames 的流程

使用版本為：（且使用 webpack 做打包）
```js
❯ rails -v
Rails 6.1.7.6
❯ ruby -v
ruby 2.7.2p137
```

:::danger note
Rails 7 將 Hotwire 整合為其標配的一部分，可以在 Rails 7 中輕鬆使用 Hotwire，而不需要額外的安裝步驟，但是目前專案為 rails 6.1.7.6，但還是可以透過下載的方式引入 Hotwire
:::

### **第一步：** 安裝環境

在終端機打上：
```js
yarn add @hotwired/turbo-rails@7.0.0
```

```js
rails turbo:install
```

再到 webpack 的入口 (app/javascript/packs/application.js) 引入：
```js
import "@hotwired/turbo-rails"
```

### **第二步：** 框起需要局部更新的部分
在提交表單（ shared/tweet_form ）的地方運用 turbo_frame_tag 把它包起來：
```js
<%= turbo_stream_from "tweets" %>
<%= turbo_frame_tag "tweet_form" do %>
  <%= render 'shared/tweet_form' %>
<% end %>
<%= turbo_frame_tag "tweets" do %>
  <%= render @tweets %>
<% end %>
```

`<%= turbo_stream_from "tweets" %>` : 這一行在告訴 Turbo Stream 這個頁面關注名為 "tweets" 的 Turbo Stream 訊息，這樣，當有與 "tweets" 有關的操作發生時，頁面就會收到更新

`<%= turbo_frame_tag "tweet_form" do %>` : 這是提交推文表單的 Turbo Frame。在推文提交後，可以使用 Turbo Stream 將新的推文表單內容插入這個特定的 Turbo Frame 中，以實現即時更新

`<%= turbo_frame_tag "tweets" do %>` : 這是顯示推文的區域的 Turbo Frame。當新推文被創建，可以使用 Turbo Stream 將新的推文內容插入這個 Turbo Frame 中，以實現即時更新

:::danger note
 Turbo Frame 是一個標示可以更新的區域，而 Turbo Stream 則是更新這個區域的內容。這兩者的配合，實現了在不重新載入整個頁面的情況下，更新前端內容
:::

:::note
 這裡的 `tweets`、 `tweet_form` 都是可以自定義的，在網頁會呈現：`<turbo-frame id="tweet_form">` 這樣的 id
:::


### **第三步：** Turbo Frame 插入最新內容
建立 create.turbo_stream.erb ：**使用 prepend 在指定的 Turbo Frame 中的最前面插入新的內容**
* 也可以使用 append 插入到頁面內容最後面
```js
<%= turbo_stream.prepend "tweets" do %>
  <%= render partial: "tweets/tweet", locals: { tweet: @tweet } %> //這個頁面為推文顯示的頁面
<% end %>
```
:::danger note
這裡的 "tweets" 是 Turbo Frame 的 ID，確保與表單中局部更新（使用 Turbo Frame）的地方的 id 相匹配，且這個 id 需要和 Turbo Stream 操作中指定的名稱相匹配，確保正確的 Frame 被更新，為了好理解所以我都使用 tweets 命名
:::

### **第四步：** Turbo Stream 回應格式
最後，在 tweet_controller.rb 新增處理 Turbo Stream 格式的回應，先找到當前使用者新增的推文後，如果處理成功推文會回應格式有兩種：HTML 和 Turbo Stream
```js
def create
  @tweet = current_user.tweets.new(tweet_params)
  respond_to do |format|
    if @tweet.save
      format.html { redirect_to dashboard_path }
      format.turbo_stream 
    end
  end
end

def tweet_params
  params.require(:tweet).permit(:body)
end
```



那麼這時候在表單打入文字，按下提交後，可以在日誌裡看到 **TURBO_STREAM** 的字樣就算是成功完成啦，如果是看到 `as HTML` 代表沒有成功局部更新唷
```js
Processing by TweetsController#create as TURBO_STREAM
```

## Bootstrap 結合 Stimulus
那因為我的專案是使用 bootstrap 去做樣式的設計，因為使用 TURBO_STREAM 會造成 bootstrap 一些問題的產生

先載入 Stimulus：
```js
rails webpacker:install:stimulus
```

```js
rails stimulus:install
```
接著一樣到 webpack 的入口去引入 stimulus
```js
import "stimulus"
import "controllers"
```
同時會看到裡面有引入 `import "controllers"`，因為在下載的同時 app/javascript/controllers 會產生這個檔案，可以在這裡操作 javascript

### 問題 1
  * 表單送出後如何清空 input 資料
```js
先在想要控制的表單中加入 `data: { controller: 'tweet' }`
// form
<%= form_with model: Tweet.new, url: tweets_path,class:'my-3',data: { controller: 'tweet' } do |f| %>
  <%= f.text_area :body, class:'form-control tweet-input fs-5', placeholder:"what's happening?" %>
  <div class="d-flex justify-content-end mt-2">
    <%= f.submit 'tweet' , class:'btn btn-primary text-white rounded-pill' %>
  </div>
<% end %>
```

再來 app/javascript/controllers 建立新的 controller : `tweet_controller.js`，命名會跟表單中建立的 controller 名字相同。

這時候當元素(this.element)連接到 DOM 時，會綁定一個事件監聽器，監聽 Turbo Streams 中的 [**turbo:submit-end** ](<https://turbo.hotwired.dev/reference/events>) 事件（可以參考官網）。這個事件在 Turbo Frame 中的表單提交結束後觸發


```js
// tweet_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  
  connect() {
    this.element.addEventListener('turbo:submit-end', () => {
      this.element.reset();
    })
  }
}
```


:::note 
  this.element 會呈現 form 的 html 的結構：
  `<form class="my-3 tweet-form" data-controller="tweet" action="/tweets" accept-charset="UTF-8" method="post" >…</form>`
:::

### 問題 2
* 送出資料後 modal 無法正常關閉
因為這時候已經與 Stimulus 連結好了，這時候只要找到 modal 的關閉 btn 的 class (.btn-close) 就可以嚕，意思是：在按下提交的那一刻，點擊關閉的按鈕，就可以達到我們要的效果了

```js
// tweet_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  
  connect() {
    this.element.addEventListener('turbo:submit-end', () => {
      this.element.reset();
      document.querySelector('.btn-close').click()
    })
  }
}
```

總而言之，Turbo Frames 使得實現動態和局部更新的 Web 頁面變得更加簡單，減少了對傳統 Ajax 和 JavaScript 的依賴！

