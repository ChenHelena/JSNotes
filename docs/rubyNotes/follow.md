---
description: follow 功能實作
---

# follow
## 使用 Stimulus 套件

* data 屬性用於指定 Stimulus controller 的行為
  * action: 'user#follow' : 指定 controller 為 user、follow 為 user 裡面的方法
  * user: @story.user.id: 包含了用戶 ID 的數據。會在 Stimulus controller 中使用
  * target: 'user.followBtn' : 指定了 user 中的 target，並且可以操作它
```js
<%= link_to '#',class:'follow-button', 
                data: { action: 'user#follow' , 
                user: @story.user.id, 
                target: 'user.followBtn' } %>
```

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
