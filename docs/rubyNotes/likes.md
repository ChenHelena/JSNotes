---
description: 按讚功能
---
# 按讚功能 - Like
## Views 畫面

透過 data-action 和 data-controller 的屬性，告訴 stimulus 哪個 controller 及哪個方法應該在按鈕點擊時觸發

* slug : 是透過 data 屬性傳遞給 controller 的參數，可以類似 id
```js
<footer data-controller="story">
  <%= link_to '#' , class: 'btn btn-primary like_btn' , data: { action: 'story#toggleLike', slug: @story.slug } do %>
    <i class="fa-regular fa-thumbs-up" data-story-target="likeIcon"></i>  
    <span data-story-target="likesCount">0</span>
  <% end %>
    <span> 
      <span data-story-target="likesCountsText">@username</span>
      <span>和</span>
      <%= link_to '#' do %>
        <span>其他</span>
        <span>0</span>
        <span>位</span>
      <% end %>
      <span>按讚</span>
    </span>
</footer>
```
畫面上想呈現這樣的狀態：username 和 其他 1 位 按讚


## Stimulus 前後端連結
因為後端需要有一個路由來處理點讚事件，那我想要我的路由長這樣：/stories/:id/like，將資料傳遞到這個路由。利用 axios 發送 post 請求到 /stories/${slug}/like（目前路由尚未創建)
```js
// story_controller.js
import { Controller } from 'stimulus';
import axios from 'axios';
 
export default class extends Controller {
  static targets = ['likesCount','likeIcon','likesCountsText'];

  toggleLike(e) {
  e.preventDefault();
  const slug = e.currentTarget.dataset.slug;
  axios.post(`/stories/${slug}/like`)
    .then(response => {
      console.log(response.data);
    })
    .catch(err => {
      console.log(err.response);
    });
  }
}
```
* static targets = ['likesCount','likeIcon','likesCountsText'] : 聲明要操作的 html，後續可以透過 this.likesCountTarget, this.likeIconTarget, this.likesCountsTextTarget 獲取 DOM 元素

可以用 rails routes 去查看，這樣創建的路由為：/stories/:id/like
```js
// routes.rb
resources :stories do
  member do
    post :like
  end
end
```

這時候應該會出現 404 狀態碼，此時頁面根本還沒有做，所以報錯是正常的

## Controller 連結
在 controller 的設置確保有連結成功：
```js
class LikesController < ApplicationController
  before_action :find_story
  def like
    if user_signed_in?
      render json: {status: 'ok'}
    else
      render json: {status: 'signed_in_first'}
    end
  end
end
```
之後回到頁面按下檢查，如果出現 ok 那就是連結成功嚕


在 Rails 中實現按讚功能通常需要兩個主要 model，例如 Story 和 User，以及一個連接它們的關聯 model，例如 Like

生成一個 Like model，並且連結 User 和 Story
```js
rails generate model Like user:references post:references
```

運行數據庫遷移：
```js
rails db:migrate
```

## Model 設置關聯
```js
// Like model
class Like < ApplicationRecord
  belongs_to :user
  belongs_to :story
end
```

* 每則文章可以被很多人按讚
* dependent: :destroy : 當刪除一則 story，與它關聯的 like 都會被刪除
```js
// Story model
class Story < ApplicationRecord
  has_many :likes, dependent: :destroy
end
```

* 每個使用者可以收到很多讚
同時把**方法寫在 model** 裡，之後在 controller 裡調用：

like? 先檢查這篇文章有沒有被按讚，like! 如果有的話就刪除顯示 Like，如果沒有的話就新增資料進去顯示 Liked
```js
// User model
class User < ApplicationRecord
  has_many :likes, dependent: :destroy
end

def like?(story)
  likes.exists?(story: story)
end

def like!(story)
  if like?(story)
    likes.find_by(story: story).destroy
    return 'Like'
  else
    likes.create(story: story)
    return 'Liked'
  end
end
```

我想要紀錄按讚數量的話 : 在 stories 表格新增一個 likes_count 的整數欄位：
```js
rails generate migration AddLikesCountToStories likes_count:integer
```

```js
rails db:migrate
```

避免 n+1 問題，在 Like model 新增 counter_cache :
```js
class Like < ApplicationRecord
  belongs_to :story, counter_cache: true
end
```

因為我在畫面上我想要顯示有哪些使用者按讚，不只有數字而已：

透過 liking_users 的方法取得所有點讚這篇文章的用戶
```js
// Story model
has_many :liking_users, through: :likes, source: :user
```
透過 liked_stories 的方法取得所有被該用戶喜歡的文章（可能這裡不會用到）
```js
// User model
has_many :liked_stories, through: :likes, source: :story
```

設置好關聯後，那我們就可以來寫前後端的邏輯了


## Controller 設置方法
回到 controller 的設置：
先檢查使用者有無登入
* 如果有的話：
  * 會顯示 like 或 liked(當前使用者有無按讚)
  * @liking_users 哪個使用者按了這篇文章的讚
* 如果沒有：
  * head :unauthorized : 就讓他顯示 401 狀態碼
```js
class LikesController < ApplicationController
  skip_before_action :verify_authenticity_token //略過身份驗證
  before_action :find_story
  def like
    if user_signed_in?
      status = current_user.like!(@story)//調用 model 裡的 like! 方法
      @liking_users = @story.liking_users.pluck(:username) //因為關聯有設置，所以可以使用 @story.liking_users
      @story.reload
      render json: { status: status, liked: current_user.like?(@story),likes_count: @story.likes_count, liking_users: @liking_users }
    else
      head :unauthorized //會返回 401 狀態碼
    end
  end

  private
  def find_story
    @story = Story.friendly.find(params[:id])
  end
end
```

## Stimulus 點讚邏輯設置
回到 story_controller.js，處理點讚功能
```js
import { Controller } from 'stimulus';
import axios from 'axios';
 
export default class extends Controller {
  static targets = ['likesCount','likeIcon','likesCountsText'];

  
  toggleLike(e) {
  e.preventDefault();

  const slug = e.currentTarget.dataset.slug;
  axios.post(`/stories/${slug}/like`)
    .then(response => {
      console.log(response.data);
      let likesCount = response.data.likes_count;
      let liked = response.data.liked;
      let likingUsers = response.data.liking_users;
      // 更新按讚圖標和讚數
        if (liked) {
          this.likeIconTarget.classList.remove('fa-regular');
          this.likeIconTarget.classList.add('fa-solid');
        } else {
          this.likeIconTarget.classList.remove('fa-solid');
          this.likeIconTarget.classList.add('fa-regular');
        }
        this.likesCountTarget.innerHTML = likesCount;

        //處理按讚使用者帳號顯示及不顯示
        if (likingUsers && likingUsers.length >= 2) {
          this.likesCountsTextTarget.classList.remove('is-hidden');
          this.likesCountsTextTarget.classList.add('is-inline');
        } else {
          this.likesCountsTextTarget.classList.remove('is-inline');
          this.likesCountsTextTarget.classList.add('is-hidden');
        }
        
    })
    .catch(err => {
      if(err.response && err.response.status == 401){
        alert("你必須先登入")
      }else{
        console.log(err.response);
      }
    });
  }
}
```

因為在 controller 有設置，head :unauthorized 狀態碼會呈現 401，這時候在 catch error 的時候，可以設定你必須先登入的 alert，這樣就可以提醒未登入的使用者


## Views 畫面切換
最後也要更改畫面上（view）的切換 :

```js
<footer data-controller="story">
  <%= link_to '#' , class: 'btn btn-primary like_btn' , data: { action: 'story#toggleLike', slug: @story.slug } do
    %>
    <i class="<%= ('fa-solid' if current_user&.like?(@story)) || 'fa-regular' %> fa-thumbs-up" data-story-target="likeIcon"></i>  
    <span data-story-target="likesCount"><%= @story.likes_count %></span>
  <% end %>
  <span>
    <% if @story.liking_users&.any? %>
      <%= @story.liking_users.first.username %>
        <span class="<%= ('is-inline' if @story.liking_users.count > 1 )|| 'is-hidden' %>" data-story-target="likesCountsText">
          <span>和</span>
          <%= link_to '#' do %>
            <span>其他</span>
            <span>
              <%= @story.liking_users.count - 1 %>
            </span> 位
          <% end %>
          <span>按讚</span>
        </span>
    <% else %>
      尚無點用戶
    <% end %>
  </span>
</footer>
```

  點讚部分：

  link_to 方法創建了一個連結，這個連結會觸發 story#toggleLike 在story_controller.js 裡面的 toggleLike 方法，同時傳遞文章的 slug 當作 id
  ```js
  <%= link_to '#' , class: 'btn btn-primary like_btn' , data: { action: 'story#toggleLike', slug: @story.slug } do %>
    <i class="<%= ('fa-solid' if current_user&.like?(@story)) || 'fa-regular' %> fa-thumbs-up" data-story-target="likeIcon"></i>  
    <span data-story-target="likesCount"><%= @story.likes_count %></span>
  <% end %>
  ```

  切換空心讚及實心讚
  ```js
  <i class="<%= ('fa-solid' if current_user&.like?(@story)) || 'fa-regular' %> fa-thumbs-up" data-story-target="likeIcon"></i>
  ```

點讚使用者顯示：
```js
<span>
  <% if @story.liking_users&.any? %>
    <%= @story.liking_users.first.username %>
    <span class="<%= ('is-inline' if @story.liking_users.count > 1 )|| 'is-hidden' %>" data-story-target="likesCountsText">
      <span>和</span>
      <%= link_to '#' do %>
        <span>其他</span>
        <span><%= @story.liking_users.count - 1 %></span> 位
      <% end %>
      <span>按讚</span>
    </span>
  <% else %>
    尚無點用戶
  <% end %>
</span>
```

* @story.liking_users&.any? : 檢查是否有使用者按讚
* & : 是安全符，處理可能為 nil 的情況


這樣做就可以出現是誰按讚，超過兩個人就會顯示：`username 和其他 1 位按讚` 囉