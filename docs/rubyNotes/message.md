---
description: 訊息功能
---
# 訊息串功能
## **建立關聯**
### 建立 message model

```js
rails g model message user:references body:text
```

### message 與 user 建立關聯
#### 建立一對多關聯
```js
// user model
has_many :messages
```

```js
// message model
belongs_to :user
validates :body, presence: true
```

### 建立訊息串（message_thread） model

```js
rails g message_thread
```

### message_thread 與 user/message 建立關聯

```js
// user model
has_and_belong_to_many :message_threads
```

```js
// message_thread model
has_and_belong_to_many :users
has_many :messages
```

```js
// message model
belongs_to :message_thread
```

### 新增外鍵
#### 在 messages 表中添加 message_thread_id 外鍵，目的是在 messages 表中建立與 message_threads 表的關聯
```js 
rails g migration add_message_threads_reference_to_messages message_thread:references
```



### 建立訊息及訊息串的儲存表 （message_threads_user）

```js
rails g model message_threads_user user:references message_thread:references
```

```js
// message_threads_user model
class MessageThreadsUser < ApplicationRecord
  belongs_to :user
  belongs_to :message_thread
end
```

### 建立路由

#### 訊息串及訊息的路由設置

```js
resources :message_threads, only: :index, path: '/messages' 
resources :messages, only: :create 
```

### 建立 controller

```js
class MessagesController < ApplicationController
  before_action :authenticate_user!

  def create
    user = User.find(params[:user_id])
    user_message_thread_ids = MessageThreadsUser.where(user: user).pluck(:message_thread_id)
    my_message_thread_ids = MessageThreadsUser.where(user: current_user).pluck(:message_thread_id)

    common_message_thread_ids = user_message_thread_ids & my_message_thread_ids
    if common_message_thread_ids.empty?
      ActiveRecord::Base.transaction do
        message_thread = MessageThread.create
        user.message_threads << message_thread
        current_user.message_threads << message_thread
        Message.create(message_params.merge(user: current_user, message_thread: message_thread))
      end
    else
      message_thread = MessageThread.find(common_message_thread_ids.first)
      Message.create(message_params.merge(user: current_user, message_thread: message_thread))
    end
  end

  private
  def message_params
    params.require(:message).permit(:body)
  end 
end
```

```js
class MessageThreadsController < ApplicationController
  before_action :authenticate_user!

  def index
    message_thread_ids = MessageThreadsUser.where(user: current_user).pluck(:message_thread_id)
    @message_threads = MessageThread.includes(:users, :messages).where(id: message_thread_ids)
    if params[:user_id].present?
      @new_message_thread_user = User.find(params[:user_id])
    end
  end
end 
```

## 訊息串 （message_thread）畫面顯示

```js
<% @message_threads.each do |message_thread| %>
  <% other_user = message_thread.users.where.not(id: current_user.id).first %>
  <div class="py-3 px-2 d-flex">
    <div class="flex-shrink-0">
      <% if other_user.avatar.attached? %>
        <%= avatar(other_user, size: { x: 48, y: 48 }) %>
      <% else %>
        <%= image_tag 'user.jpeg' , class: 'rounded-circle' , size: '48x48' , alt: 'Default' %>
      <% end %>
    </div>
    <div class="flex-grow-1 ms-3 align-items-center d-flex">
      <span class="fw-semibol text-dark">
        <%= other_user.name %>&nbsp;
      </span>
      <span class="text-black-50">
        @<%= other_user.username %>
      </span>
    </div>
  </div>
<% end %>
```

## 用 stimulus 找到當前 user_id
#### 找到嵌在上面的 id 可以讓我們對當前對象添加點擊事件及讓按鈕添加樣式
先在畫面上儲存 user_id 的數據，以便在 javascript 使用
```js
data-messages-target-user-id="<%= user_id %>"
```

接著，再到 js 的文件裡面寫上：
```js
const queryString = window.location.search //獲取當前頁面的查詢字符串
const urlParams = new URLSearchParams(queryString)

if(urlParams.get('user_id') !== null){
  this.element.querySelector(`[data-messages-target-user-id="${urlParams.get('user_id')}"]`).click()
}
```
找到 id 並且點擊

