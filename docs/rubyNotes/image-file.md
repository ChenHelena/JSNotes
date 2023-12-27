---
description: Avtive Storge、image_tag、imagemagick、mini_imagek、Users_helper、resize_to_fill	
---
# 上傳圖片及縮圖
## 上傳圖片
[ Avtive Storge ](<https://guides.rubyonrails.org/active_storage_overview.html#transforming-images>) 是 Rails 中的一個內建庫，用於處理文件上傳和文件儲存的功能

用於生成 Active Storage 相關的數據庫文件和 model 文件。它會在 db/migrate 目錄下生成一個文件，用於創建 Active Storage 所需的數據庫表
```js
rails active_storage:install
rails db:migrate
```

在 model user 使用 has_one_attached 定義關聯
```js
class User < ApplicationRecord
  has_one_attached :avatar
end
```

然後在想要顯示的 views 檔案新增
```js 
<div class="field">
  <%= f.label :avatar, '大頭貼照片' %><br />
  <div class="file">
    <%= f.file_field :avatar %>
  </div>
</div>
```
### Devise 的參數過濾器（parameter sanitizer）

要記得在 controller 新增配置帳戶更新的參數，確保只有允許的屬性（:avatar）會被更新到數據庫
```js
def configure_account_update_params
  devise_parameter_sanitizer.permit(:account_update, keys: [:avatar])
end
```

### image_tag
image_tag 是 Rails 中一個輔助方法（helper method），用於生成 HTML `<img>` 標籤，用於在網頁上顯示圖片

* user-avatar : 新增一個 class 可以使用 css 來美化它
* user.avatar.attached?：條件語句，檢查 user 對象是否有附加（attached）了頭像（avatar），有的話才執行
```js
<%= image_tag user.avatar, class: "user-avatar" if user.avatar.attached? %>
```

## 圖片太大怎麼辦
MiniMagick 是一個基於 ImageMagick 的 Ruby gem，使用 Ruby 進行圖像處理。因此，如果使用 MiniMagick，系統就需要安裝 ImageMagick：

Gemfile 中添加了 mini_magick，
```js
gem 'mini_magick', '~> 4.12'
```

同時確保 image_processing 存在（Rails 中使用的內建圖像處理庫）
```js
# Use Active Storage variant
gem 'image_processing', '~> 1.2'
```

並且執行 `bundle install` 來安裝相依的 gem

再安裝 imagemagick( 以下是 mac 安裝方法 ）

```js
brew install imagemagick
```

只要在想縮小圖片的地方加上 `variant`，**variant** 方法是 Active Storage 提供的一個方法，並使用 **`resize_to_fill`** 方法，會將圖片調整為指定的大小，並填滿整個區域，不保持原始比例，圖片可能會被拉伸或壓縮以填滿整個指定區域
```js
<%= image_tag user.avatar.variant(resize_to_fill: [size[:x], size[:y]]), class: "user-avatar" if user.avatar.attached? %>
```

### Users_helper
可以在 /app/helpers 底下新增一個 **users_helper** 把方法包裝起來
```js
module UsersHelper
  def avatar(user, size: { x: 250, y: 250 })
    image_tag user.avatar.variant(resize_to_fill: [size[:x], size[:y]]), class: "user-avatar" if user.avatar.attached?
  end
end
```

只要在 views 中加入這段，current_user 當前對象的參數進去，如果有圖片就會顯示
```js
 
```
