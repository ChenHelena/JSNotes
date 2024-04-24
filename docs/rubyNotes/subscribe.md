---
description: 訂閱電子報
---

# 訂閱電子報

## 連結 stimulus
創建一個 `subscribe_controller.js`檔案，控制 input 名叫 email 的目標元素，在按鈕設置 addSubscribe 監聽事件
```js
<section class="subscribe section has-background-light is-centered has-text-centered" data-controller="subscribe">
  <h3 class="has-text-centered is-size-4 mb-5">訂閱電子報</h3>

  <div class="container">
    <form action="#">
      <div class="field has-addons">
        <div class="control has-icons-left is-expanded">
          <input type="email" placeholder="請輸入您的 Email 信箱" name="email" class="input is-rounded" data-subscribe-target="email" >
          <span class="icon is-small is-left">
            <i class="fas fa-envelope"></i>
          </span>
        </div>
        <div class="control">
          <input type="submit" value="訂閱電子報" class="button is-dark is-rounded" data-action="subscribe#addSubscribe" >
        </div>
      </div>
    </form>
  </div>
</section>
```

成功連結 stimulus，取得 key in 在 input 裡的值

```js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "email" ]

  addSubscribe(e){
    e.preventDefault();

    let email = this.emailTarget.value.trim()
    console.log(email)
  }
}
```

## 創建 routes
創建的目的是為了讓 api 能夠到達相應的 controller
```js
namespace :api do
  namespace :v1 do
    post 'subscribe', to: 'utils#subscribe'
  end
end
```

## 創建 controller
```js
rails g controller api/v1/utils
``` 
**確保後端收到 post 請求**：創建 subscribe 方法，按下按鈕，會呈現 'ok'，確認有連結到方法
```js
class Api::V1::UtilsController < ApplicationController
  def subscribe
    render json: { status: 'ok' }
  end
end
```

## 把資料送去後端
* 導入 `import Rails from "@rails/ujs"`，將表單提交時，使用ajax 將數據傳給後端， ajax 需要的資料為：`url, type, datatype, data`
* trim(): 去除空格

首先，建立 new FormData() 新對象，用於儲存要發送的數據，使用ajax 請求到 `/api/v1/subscribe`，並將儲存的數據（data）代入，傳送過去
```js
import { Controller } from "stimulus"
import Rails from "@rails/ujs"

export default class extends Controller {
  static targets = [ "email" ]

  addSubscribe(e){
    e.preventDefault();

    let data = new FormData()
    let email = this.emailTarget.value.trim()
    data.append("subscribe[email]", email)

    Rails.ajax({
      url:'/api/v1/subscribe',
      type:'POST',
      dataType: 'json',
      data: data
    })
  }
}
```

## 後台數據
在畫面上按下 subscribe 按鈕，後台數據會出現：
```
Parameters: {"subscribe"=>{"email"=>"123@gmail.com"}}
```

## 存放資料
可以獲取 input 的值之後，就要寫進資料庫，所以要開一個 model 來存放資料，確保 email 為唯一值
```js
rails g model Subscribe email:string:uniq
```
在 Subscribe model 設定驗證：

```js
validates :email, uniqueness: true
```

```js
rails db:migrate
```

## 寫進資料庫 
回到 controller，運用 subscribe 方法寫進資料庫

```js
class Api::V1::UtilsController < ApplicationController
  def subscribe
    email = params['subscribe']['email']
    sub = Subscribe.new(email: email)

    if sub.save
      render json: { status: 'ok', email: email }
    else
      render json: { status: 'duplicate', email: email }
    end

  end
end
```

## 畫面顯示
這時候在 Stimulus（subscribe_controller.js） 建立成功及失敗的畫面
```js
Rails.ajax({
  url:'/api/v1/subscribe',
  type:'POST',
  dataType: 'json',
  data: data, 
  success: function(res) {
    switch(res.status){
      case 'ok':
        alert('完成訂閱');
        this.emailTarget.value = ''
        break
      case 'duplicate':
        alert(`${res.email}已經訂閱過`);
        break
    }
  }, 
  error: function(err) {
    console.log(err);
  }
})
```

這時候按下按鈕會顯示 "完成訂閱" 的提示訊息，但是並不會把裡面的值清空，原因是因為這時候用 function，this 會指向的是函數本身，因此這時候的 this.emailTarget 是 undefined

:::danger note
把 function 改成 箭頭函式就可以解決
:::

```js
success: (res) => {
    switch(res.status){
      case 'ok':
        alert('完成訂閱');
        this.emailTarget.value = ''
        break
      case 'duplicate':
        alert(`${res.email}已經訂閱過`);
        break
    }
  }, 
  error: (err) => {
    console.log(err);
  }
```