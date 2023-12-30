---
description: follow 功能實作
---

# 追蹤功能 - Follow
## 使用 Stimulus 套件

* data 屬性用於指定 Stimulus controller 的行為
  * action: 'user#follow' : 指定 controller 為 user、follow 為 user 裡面的方法
  * user: @story.user.id: 包含了用戶 ID 的數據。會在 Stimulus controller 中使用
  * target: 'user.followBtn' : 指定了 user 中的 target，並且可以操作它
```js
<span class="profile" data-controller="user">
  <%= link_to '#',class:'follow-button', 
                  data: { action: 'user#follow' , 
                  user: @story.user.id, 
                  target: 'user.followBtn' } %>
</span>
```
### 起手式
創建一個 JavaScript 文件，例如 user_controller.js，放入 app/javascript/controllers

這是一個起手式，要有 target 及 方法（follow）對應 views 中寫的資訊去創建

```js
// app/javascript/controllers/user_controller.js
import { Controller } from "stimulus";

export default class extends Controller {
  static targets = ["followBtn"];

  follow(e) {
    e.preventDefault();
    let user = this.followBtnTarget.dataset.user
  }
}
```
this.followBtnTarget 代表了 followBtn 目標元素，可以通過它的 dataset.user 來獲取 HTML 元素上的 data-user（也就是 user: @story.user.id） 屬性的值

## axios 套件
接著引入 axios 來進行與後端的 HTTP 請求，Axios 發送 POST 請求時，需要指定要發送請求的目標 URL：`/users/:id/follow`，因為 this.followBtnTarget.dataset.user 可以抓取 user.id 的值，所以可以用這個方式帶入進去（ js用法 ）
```js
// app/javascript/controllers/user_controller.js
import { Controller } from "stimulus";
import axios from "axios";

export default class extends Controller {
  static targets = ["followBtn"];

  follow(e) {
    e.preventDefault();
    let user = this.followBtnTarget.dataset.user
    axios.post(`/users/${user}/follow`)
      .then((res)=>{
        console.log(res.data);
      })
      .catch((err)=>{
        console.log(err);
      })
  }
}
```
## 建立路由
因為現在目前 routes 設定中，/users/:id/follow 沒有這個路由，所以我們要自己去創建
```js
// routes.rb
esources :users, only: [] do
  member do
    post :follow
  end
end
```
:::danger note
only: [] 表示在 :users 資源上生成的路由將為空，即不生成任何標準的 RESTful 路由 (CRUD)
:::

創建完，可以來終端機打上
```
rails routes | grep follow
```

查看現在的 follow 路由是不是我們需要的 /users/:id/follow
```js
follow_user POST  /users/:id/follow(:format)               
                  users#follow
```
這樣 user_controller.js 裡打的路徑就是正確的，可以使用的，只是這時候還是會出現 404 頁面找不到，因為 /users/:id/follow 這個頁面還沒有存在，報錯是正常的

## 建立 controller
沒有頁面的話我們就新增一個給它，第一步先新增一個 controller，想要實現 follow 動作，就需要先定義 follow 方法
```js
rails g controller users
```
生成的 controller 長這樣
```js
class UsersController < ApplicationController
end
```
新增 follow 方法，而 follow 動作是我要找到某個使用者追蹤他，或者是我被追蹤，都需要相對應的 id 才能找到使用者是誰 `@user = User.find(params[:id])`，把他包裝成一個方法，以便日後取用
```js
class UsersController < ApplicationController
  before_action :find_user
  skip_before_action :verify_authenticity_token, only: [:follow]
  def follow
  end

  private
  def find_user
    @user = User.find(params[:id])
  end
end
```
* before_action 方法，用於執行在某些動作之前的預先處理
* skip_before_action :verify_authenticity_token, only: [:follow]：
這表示在執行 follow 方法時，不要執行 Rails 預設的 CSRF token 驗證


寫 follow 方法，在畫面按下 follow 按鈕會呈現 ok ，同時也會檢查有無登入，沒有的話會在檢查裡看到 sign_in_first 的提示，那這時候就確定已經前後端連結起來了
```js
class UsersController < ApplicationController
  before_action :find_user
  skip_before_action :verify_authenticity_token, only: [:follow]
  def follow
    if user_signed_in?
      render json:{status: 'ok'}
    else
      render json:{status: 'sign_in_first'}
    end
  end

  private
  def find_user
    @user = User.find(params[:id])
  end
end
```
## 建立 model
再來，因為我們要做的是追蹤別人，同時別人也會追蹤自己，這時候就需要一個 model 來記錄誰追蹤誰
```js
rails g model Follow user:references following_id:integer:index
```

建立出來的 model 會長這樣：
```js
class CreateFollows < ActiveRecord::Migration[6.1]
  def change
    create_table :follows do |t|
      t.references :user, null: false, foreign_key: true
      t.integer :following_id

      t.timestamps
    end
    add_index :follows, :following_id
  end
end
```

```js
rails db:migrate
```

follow 的 model 目前長這樣 :

```js
class Follow < ApplicationRecord
  belongs_to :user
  belongs_to :following, foreign_key: 'following_id', class_name: 'User'
end
```
`belongs_to :following, foreign_key: 'following_id', class_name: 'User'` 這句話的意思是 :

Follow model 中的 following_id 用於建立到 User model 的關聯，這個關聯的名稱是 following

* following 不存在所以可以自訂名稱，但因為要關聯 following_id，所以依照慣例可以設定名稱為 following


user 的 model 目前長這樣 :

```js
class User < ApplicationRecor
  has_many :follows, foreign_key: 'user_id'
  has_many :followers, through: :follows, source: :following
end
```
`has_many :follows, foreign_key: 'user_id'`: 

這表示一個用戶擁有多個 Follow，可以獲取用戶正在追蹤的對象

`has_many :followers, through: :follows, source: :following` : 

透過 follows 關聯，followers 會從 Follow 模型的 following 字段找到相對應的用戶

* user_id: 這是追蹤者（follower）的 ID
* following_id: 這是被追蹤者（following）的 ID

:::danger note
透過 user.followers 可以直接獲取一個用戶的追蹤者列表
:::

## 定義方法
在 user model 建立方法：
```js
# methods
def follow?(user)
  follows.exists?(following: user)
end

def follow!(user)
  if follow?(user)
    follows.find_by(following: user).destroy
    return 'Follow'
  else
    follows.create(following: user)
    return 'Followed'
  end
end
```

* 問號方法：true/false
* 驚嘆號方法：會做出行為
* follow?(user) : 當前用戶是否已經追蹤了這個使用者（user）
* follows.exists?(following: user) : 在這個 follows 是否存在你的追蹤對象（user）
* follow!(user) : 當前用戶要去追蹤這個使用者（user），或是取消追蹤
  * 如果追蹤了：follows.find_by(following: user).destroy 就去找到這個 user 取消追蹤
  * 如果沒有追蹤：follows.create(following: user) 就去新增使用者（user）

回到 user_controller，調用 model 中的 follow! 方法，檢查用戶是否已經登錄
```js
def follow
  if user_signed_in?
    render json:{status: current_user.follow!(@user)}
  else
    head :unauthorized // 返回 401 未授權狀態碼
  end
end
```

## 點擊事件
再回到 user_controller.js 處理按鈕的點擊事件，根據後端的狀態更新前端的顯示
```js
// app/javascript/controllers/user_controller.js
import { Controller } from "stimulus"
import axios from "axios";

export default class extends Controller {
  static targets = [ "followBtn" ]

  follow(e){
    e.preventDefault();
    
    let user = this.followBtnTarget.dataset.user
    let button = this.followBtnTarget
    console.log(user);
    axios.post(`/users/${user}/follow`)
      .then((res)=>{
        let status = res.data.status
        switch(status){
          case 'Follow':
            // 如果是 'Follow'，添加深色樣式
            button.classList.remove('followed-button');
            button.classList.add('follow-button');
            break;
          case 'Followed':
            // 如果是 'Followed'，添加淺色樣式
            button.classList.remove('follow-button');
            button.classList.add('followed-button');
            break;
        }
        button.innerHTML = status;
      })
      .catch((err)=>{
        if(err.response && err.response.status == 401){
          // 未登入的處理
          alert('你必須先登入')
        }else{
          console.log(err);
        }
      })
  }
}
```

:::danger note
寫 err.response 主要是因為在處理 Axios 錯誤時，並不是每次都有 response 對象。如果請求本身就沒有收到響應（例如，網絡錯誤、伺服器未回應等情況），err.response 就會是 undefined，所以加入這句是為了避免 undefined 的錯誤
:::

## 畫面與資料同步
最後回到一開始寫的 views，根據用戶登錄狀態和關注狀態而動態改變的關注按鈕

```js
<span class="profile" data-controller="user">
  <%= link_to '#',  class:("followed-button" if user_signed_in? && current_user.follow?(@story.user)) || "follow-button", 
                    data: { action: 'user#follow' , 
                    user: @story.user.id, 
                    target: 'user.followBtn' } do %>
    <%= (user_signed_in? && current_user.follow?(@story.user) ? 'Followed' : 'Follow')%>
  <% end %>
</span>
```

* class:("followed-button" if user_signed_in? && current_user.follow?(@story.user)) || "follow-button" : 查看用戶是否已經登入，且有沒有追蹤對方，來切換按鈕顏色
* (user_signed_in? && current_user.follow?(@story.user) ? 'Followed' : 'Follow') : 有登入且沒有追蹤對方會呈現文字 Followed，沒有的話則呈現 Follow

