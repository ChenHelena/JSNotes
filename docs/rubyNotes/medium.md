---
description: medium 實作、postgresql
---

# medium 實作
## Postgresql
* 一個開源的關聯型數據庫管理系統（RDBMS），基於 SQL 語言來管理和查詢數據

新增一個 rails 包含 postgresql，`-d` 代表 database

`rails new my_medium -d postgresql`
我在安裝的期間出現的錯誤：
1. **pg gem**
    * pg gem 包含了一個原生擴展，需要 PostgreSQL 開發庫提供必要的文件
2. **安裝 PostgreSQL 的開發庫(以 mac 為例)**
    * 安裝 PostgreSQL 開發庫，對於成功構建 pg gem 是必要的
```js
brew install postgresql
```

尚未安裝 Homebrew，可以使用以下命令安裝：
```js
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
安裝完成後，確保你已經進入了你的 Rails 專案的根目錄，執行 `bundle install`

3. **無 webpack.yml 的檔案**

先確保 gemfile 裡存在：
```js
gem 'webpacker', '~> 5.0'
```

使用 rails new 命令時，如果指定 `-d postgresql` 選項，Rails 會生成一個使用 PostgreSQL 作為數據庫的新應用程序，但不會自動生成 Webpacker 的配置文件 webpacker.yml，所以要手動新增：
```js
rails webpacker:install
```

也可以在建立專案的時候自動產生：
`rails new project_name -d postgresql --webpack`

以上問題都解決了，但還是報錯的話：
可以使用以下命令檢查 PostgreSQL 伺服器的運行狀態：

```js
pg_ctl status
```

而我遇到的問題是，pg_ctl 命令缺少必要的參數，因為它需要知道 PostgreSQL 數據目錄的位置

```
pg_ctl: no database directory specified and environment variable PGDATA unset
Try "pg_ctl --help" for more information.
```
### PostgreSQL 配置文件
* 目標：更改 data_directory 的設置
  * 是為了指定 PostgreSQL 數據目錄的位置，從**相對路徑改為絕對路徑**
  
找到 PostgreSQL 的配置文件，通常是 postgresql.conf 這個文件( 通常在第一筆找到的資料 )
```js
sudo find / -name "postgresql.conf"
```
使用文本編輯器打開文件
```js
sudo nano /usr/local/var/postgresql@14/postgresql.conf
```
文件中修改 data_directory 的設置，把指向替換成實際的 PostgreSQL 數據目錄（用 sudo find 找到的文件目錄）
```js
data_directory = '/usr/local/var/postgresql@14/data'
```
在 postgresql.conf 文件中看到了 data_directory = 'ConfigDir' 這一行，是 PostgreSQL 設定使用了相對路徑 'ConfigDir' 作為數據目錄，是導致 PostgreSQL 未能找到正確的數據目錄的原因

保存並退出文本編輯器後，重新啟動 PostgreSQL 伺服器
```js
pg_ctl restart
```

PostgreSQL 在嘗試重新啟動時遇到了一些問題的話：

確認 PostgreSQL 未在運行：
```js
pg_ctl status -D /usr/local/var/postgresql@14/
```
再啟動伺服器(記得把開著的 localhost 也一併關閉)
```js
pg_ctl start -D /usr/local/var/postgresql@14/
```

最後在專案上執行 `rails db:create` 會出現這兩個檔案，接著在網頁搜尋打上 localhost:3000 就可以開啟囉
```
Created database 'my_medium_development'
Created database 'my_medium_test'
```

## 下載 devise 套件
* 建立會員系統的畫面（ 加入會員、編輯會員、註冊、忘記密碼等等，大部分的畫面都已經做好了，適合新手開發使用 ）

先到 [ rubygems ](<https://rubygems.org/> "rubygems") 裡面搜尋 devise 套件，複製 Gemfile 欄位 `gem 'devise', '~> 4.2'` 貼到專案下 Gemfile 的檔案，再到專案底下的終端機執行 `bundle install`

用於安裝 devise，它會生成一個 `config/initializers/devise.rb` 文件，其中包含 devise 的初始設定
```js
rails generate devise:install
```

生成 User model 
```js
rails generate devise User //( model 名稱 )
```

將 devise 所需的表格和欄位添加到資料庫中
```js
rails db:migrate
```

### 客製化 devise:controllers
用於生成定制化的 controller 文件，其中 [scope] 是 Devise 設置的作用域（例如，:users，:admins等）。如果不指定作用域，預設為 :users
```js
rails generate devise:controllers [scope]
```

這些檔案將存放在你的應用程式的 app/controllers/users 目錄中，可以編輯這些文件，添加或修改相應的方法
```js
rails g devise:controllers users
```

在 routes.rb 指定了自定義的 controllers users/sessions 來處理使用者登入（sessions）相關的操作

* sessions: 'users/sessions': 這裡指定了一個自定義的控制器 users/sessions，用於處理使用者登入（sessions）相關的操作，也可以添加其他的 controllers 選項

```js
Rails.application.routes.draw do
  devise_for :users, controllers: {
    sessions: 'users/sessions'
  }
end
```
### devise 取得 permit
在 app/controllers/users/registrations_controller.rb 裡面全都幫你打好了只要把它打開即可

```js
class Users::RegistrationsController < Devise::RegistrationsController
  before_action :configure_account_update_params, only: [:update]

  def configure_account_update_params
    devise_parameter_sanitizer.permit(:account_update, keys: [:username, :intro])
  end
end
```
### devise 可設定不添加現有密碼

在 app/controllers/users/registrations_controller.rb 裡面添加設定：
```js
class Users::RegistrationsController < Devise::RegistrationsController
  def update_resource(resource, params)
    resource.update_without_password(params)
  end
end
```

### 登入、登出、註冊按鈕
devise 裡有個 `user_signed_in?` 這個方法，用於檢查使用者是否已經登入的方法，而前面的 `user` 跟建立的 model 有對應，例如建立 member model 就是 `member_sign_in?` 這個方法，`current_user` 也跟 `user_signed_in?` 有同樣特性

```js
<% if user_signed_in? %>
  <%= current_user.email %>你好
  <%= link_to '登出', destroy_user_session_path, method: 'delete', class: 'button is-danger' %>
<% else %>
  <%= link_to '註冊' , new_user_registration_path ,  class: 'button is-primary' %>
  <%= link_to '登入' , new_user_session_path, class: 'button is-light' %>
<% end %>
```

### 表單客製化
會在 app/views 目錄下生成一組 devise 的 views 文件，這些文件包括用於註冊、登入、密碼重設等功能的頁面，這樣就可以自定義它們，滿足客製化需求
```js
rails generate devise:views
```


## 安裝 Bulma

先到 [ Bulma ](<https://bulma.io/> "Bulma") 下載資料
再到 /vendor 去新增資料夾，把 bulma.css 檔案放進去

接著在 /config/initializers/assets.rb 這個文件新增：
```js
Rails.application.config.assets.paths += Dir["#{Rails.root}/vendor/assets/*"]
```
這樣一來，就可以把 `/vendor/assets/*` 路徑底下的所有文件指定給 assets `/app/assets` 的路徑

最後，在 app/assets/stylesheets/application.css 引入 bulma.css
```js
/*
 *= require bulma/bulma.css
 *= require_self
 */
```

### 添加 bulma 報錯驗證

這行顯示所有錯誤消息，使用 `full_messages `方法獲取每個錯誤的完整消息，然後使用 `to_sentence` 方法將它們轉換為一個人類可讀的句子形式
```js
<% if story.errors.any? %>
  <div class="notification is-danger">
    <%= story.errors.full_messages.to_sentence %>
  </div>
<% end %>
```

## 如何更新首頁畫面
新增一個 controller `rails g controller Pages`

在 routes.rb 把首頁指定給 pages 這個 controller 裡面的 index
```js
Rails.application.routes.draw do
  root 'pages#index'
end
```

在 controller 裡新增 index 方法 
```js
class PagesController < ApplicationController
  def index
  end
end
```
最後別忘記也要在 views/pages 新增一個同名 index.html.erb 檔案唷



## 連結指向首頁
使用 link_to 方法生成一個連結，該連結將指向 root_path，並且具有 navbar-item 這個 CSS 樣式
```js
<%= link_to root_path, class:'navbar-item' do %>
  <img src="https://bulma.io/images/bulma-logo.png" width="112" height="28">
<% end %>
```

## routes 的命名慣例
在 Rails 中，resources 方法會生成一組標準的 RESTful 路由，其中包括新增（create）、顯示（show）、修改（update）、刪除（destroy）等操作
```js
Rails.application.routes.draw do
  resources :stories 
end
```
* 新增(New) Story - new_story_path（顯示創建新故事的表單頁面）
* 創建(Create）Story - stories_path（提交表單）
* 編輯（Edit）Story - edit_story_path
* 顯示（Show）Story - story_path

## references 常常打錯怎麼辦
在建立關聯的時候，常常忘記加上 s，但是又已經 `rails db:migrate`，直接刪除在建立一個又會報錯，這時可以用 [ remove_reference ](<https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/remove_reference>)

首先，新增一個 migration：想從 stories 表中，移除 user 的引用，再新增一次（因為 reference 沒加 s）
```js
rails generate migration remove_and_add_user_reference_to_stories user:references
```
這時的 migrate 檔長這樣，執行 `rails db:migrate` 就可以把 `t.references :user` 更新成功囉
```js
class RemoveAndAddUserReferenceToStories < ActiveRecord::Migration[6.1]
  def change
    remove_reference :stories, :user, index: true
    add_reference :stories, :user, index: true
  end
end

```

這是更新後的 migrate 檔：
```js
class CreateStories < ActiveRecord::Migration[6.1]
  def change
    create_table :stories do |t|

      t.references :user
      
      t.timestamps
    end
  end
end
```
## 
## 轉換時間戳
將時間戳轉換成，例如：兩天前、三天前

只要在 views 中，這樣使用：
```js
<p>
  最後編輯時間：<%= time_ago_in_words(@story.updated_at) %>前
</p>

```

## 時區
在 application.rb 的檔案加入下列設定：
```jsx title="config/application.rb"
config.time_zone = 'Asia/Taipei'
```

## 軟刪除
向 stories 表添加一個 deleted_at 欄位，並在該欄位上創建一個索引，以提高查詢性能，數據類型為 datetime，即日期時間類型
```js
rails g migration add_deleted_at_to_story deleted_at:datetime:index
```

執行 `rails db:migrate`

migrate 檔
```js
class AddDeletedAtToStory < ActiveRecord::Migration[6.1]
  def change
    add_column :stories, :deleted_at, :datetime
    add_index :stories, :deleted_at
  end
end
```
* default_scope : 這是一個預設的作用域，默認情況下會應用於查詢模型的所有地方。在這裡，**它是一個條件**，只查詢 deleted_at 欄位為 nil 的記錄。換句話說，已經被標記為軟刪除的記錄**在查詢時會被隱藏，不會出現在默認的查詢結果中**

* destroy 方法 : 這個方法實際上並沒有執行真正的刪除操作，而是更新了 deleted_at 欄位為當前時間。這就是所謂的軟刪除。更新 deleted_at 欄位後，**這條記錄在正常查詢時就會被排除在外**，因為 default_scope 限制了查詢條件

```jsx title="story.rb"
class Story < ApplicationRecord
  //在 model 中加入預設作用域，只查詢 deleted_at 為 nil 的記錄
  default_scope{ where(deleted_at: nil)}
  
  //定義 destroy 方法，實際上是更新 deleted_at 欄位為當前時間
  def destroy
    update(deleted_at: Time.now)
  end
end
```

當你在 controller 中調用 @story.destroy 時，它會觸發 model 中的 destroy 方法，更新了 deleted_at 欄位，而不是真正地從數據庫中刪除記錄

```js
def destroy
  @story.destroy
  redirect_to stories_path,notice: '刪除成功'
end
```

## 404 - rescue_from
處理當 ActiveRecord 查找資料庫時發生的 RecordNotFound 錯誤。通常發生在嘗試根據 ID 查找資料庫中的一條記錄，但找不到該記錄時

* application_controller.rb 是一種全局設定
* `:not_found` 是代表 HTTP 的狀態碼

```js
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
  private
  def record_not_found
    render file:"#{Rails.root}/public/404.html",
           status: :not_found,
           layout: false
  end
end
```

## name 屬性
這裡的 `name` 屬性用於區別表單中的不同按鈕，以便在 controller 中處理相應的動作。當提交表單時，根據按鈕的 name 屬性， controller 可以採取不同的行動。例如，在 controller 中，你可以透過 params[:save_as_draft] 或 params[:publish] 來檢查哪個按鈕被點擊

```js
<%= form.submit '儲存草稿', name:'save_as_draft', class:'button is-light'%>
<%= form.submit '發布文章', name:'publish', class:'button is-link'%>
```

## if
將條件語句放在操作語句之後，這種寫法在一行的情況下可以讓代碼看起來更簡潔，意思是，當 params[:publish] 存在（即 "發布文章" 按鈕被點擊），則將 @story.status 設置為 'published'
```js
@story.status = 'published' if params[:publish] 
```

### &&
`&&` 短路運算符是從左到右運行的，如果最左邊的條件為假（false），則不會評估後面的條件，如果 user 為 nil，那麼 `user.avatar.attached?` 就不會被評估，這樣可以避免 nil 對象引發的錯誤
```js
if user && user.avatar.attached?
```

## aasm
 "Acts As State Machine"，是一個用於 Ruby 和 Rails 應用程式的狀態機庫

 在 [ rubyGems ](<https://rubygems.org/>) 查詢 aasm ，複製它的 gemfile `gem 'aasm', '~> 5.5'`
 貼在專案底下的 `Gemfile` 檔案裡
 ```
 gem 'aasm', '~> 5.5'
 ```
 
 開啟終端機執行 `bundle install` 以確保相應的 gem 被安裝

 使用 AASM gem 定義了一個狀態機，將其應用於 `Story` model
 ```js
include AASM
aasm(column: 'status',no_direct_assignment: true) do
  state :draft, initial: 'true'
  state :published

  event :publish do
    transitions from: :draft, to: :published
  end

  event :unpublish do
    transitions from: :published, to: :draft
  end
end
```

* column: 'status': 指定數據庫表中用於存儲狀態的為 status
* no_direct_assignment: true: 禁用直接手動更改狀態的操作

創建了一個具有 `:draft` 和 `:published` 兩種狀態的簡單狀態機，並定義了 `publish` 和 `unpublish` 兩個事件

### aasm 狀態標籤
#### controller

params[:publish] 的值，如果它存在並為真，則調用 publish! 方法將 @story 的狀態改為發布狀態

`!` 具有強制意思
```js
@story.publish! if params[:publish] 
```

#### views

使用 **content_tag** helper 方法來生成 HTML `<span>` 元素，其中包含 '已發佈' 的標籤，如果 `story.published?` 為真，則將 'tag is-success' 添加到該標籤的 class 屬性中，否則不添加

```js
<%= content_tag :span, '已發佈' , class: 'tag is-success' if story.published? %>
```

### aasm 提供的方法
`may_publish?` 和 `may_unpublish?` 這兩個 AASM gem 提供的問題式方法，以動態地顯示或隱藏發布和下架文章的按鈕
```js
<%= form.submit '發布文章', name:'publish', class:'button is-link' if story.may_publish? %>
<%= form.submit '下架文章', name:'unpublish', class:'button is-danger' if story.may_unpublish? %>
```

## 網址自定義 [ friendlyid ](<https://github.com/norman/friendly_id>)

在 Gemfile 貼上這行
```js
gem 'friendly_id', '~> 5.4.0'
```
運行
```js
bundle install
```
Story 表格中新增一個叫做 "slug" 的欄位，並給這個欄位加上唯一性約束（uniq），也可以ToUser、ToMember
```js
rails g migration AddSlugToStory slug:uniq
```
生成一個 migration 檔案以及一些其他的配置選項
```js
rails generate friendly_id
```
運行
```js
rails db:migrate
```
編輯 app/models/story.rb 檔案添加：
```js
class Story < ApplicationRecord
  extend FriendlyId
  friendly_id :name, use: :slugged
end
```

`friendly_id :name` 可以使用預設值也可以自己建立一個：
```js
class Story < ApplicationRecord
  extend FriendlyId
  friendly_id :slug_candidate, use: :slugged

  def slug_candidate
    [:title,
      [:title,ResureRandom.hex[0,6]]
    ]
  end
end
```
* [:title, SecureRandom.hex[0, 6]]：如果 title 不可用或不合適，這個選項會使用 `SecureRandom.hex` 生成一個隨機的十六進位字符串，然後取其前 6 個字符

在 controller 的 find 方法前添加 `friendly`
```js
class UserController < ApplicationController
  def find_story
    @story = current_user.stories.friendly.find(params[:id])
  end
end
```

如果是已經有很多文章了，但是需要一個一個儲存才會變更，可以使用：
```js
Story.find_each(&:save)
```

這樣一來，網頁的網址就可以呈現自定義啦！但是如果標題是中文的話可能會出線亂碼，所以要下載 babosa

### [ babosa ](<https://github.com/norman/babosa>)
處理網址是中文的亂碼問題

一樣也是到 RubyGems 下載

在 app/models/story.rb 檔案添加並搭配 friendly :

```js
def normalize_friendly_id(input)
  input.to_s.to_slug.normalize(transliterations: :russian).to_s
end
```
會呈現這樣：
```js
class Story < ApplicationRecord
  extend FriendlyId
  friendly_id :slug_candidate, use: :slugged

  def normalize_friendly_id(input)
    input.to_s.to_slug.normalize(transliterations: :russian).to_s
  end

  def slug_candidate
    [:title,
      [:title,ResureRandom.hex[0,6]]
    ]
  end
end
```
## webpacker

Webpacker 的目標是簡化在 Rails 中使用現代 JavaScript 和 CSS 的過程

在 Gemfile 貼上
```js
gem 'webpacker', '~> 5.4', '>= 5.4.4'
```

在專案底下運行
```js
bundle install
```
安裝 Webpacker gem，將創建一個 config/webpacker.yml 配置文件，以及一些其他相關的文件和目錄
```js
rails webpacker:install
```

在這個檔案 app/views/layouts/application.html.erb 新增：
```js
//用於引入 Webpack 打包的 JavaScript 文件
<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
```
這樣一來：就會去找到 app/javascript/packs/application.js 底下的 application(是 Webpack 打包中的入口文件的名稱)

接著運行 webpacker 在不刷新整個頁面的情況下查看和應用代碼更改，且運行的較為快速
```js
/bin/webpack-dev-server
```

### foreman 套件
同時運行 rails server 跟 webpacker，省下開兩個視窗的套件

將 foreman 放在 group :development 裡面
```js
gem 'foreman', '~> 0.87.2'
```

執行
```js
bundle install
```

在 Rails 項目的根目錄中創建一個名為 Procfile 的文件

在文件中寫入：
```js
web: bin/rails server -p 3000
webpack: bin/webpack-dev-server
```

使用以下命令啟動 Foreman：
```js
foreman start
```

## UJS
指的是 "Unobtrusive JavaScript"（** 不侵入式 JavaScript** ），使 JavaScript 代碼與 HTML 和 CSS 代碼盡可能分離，從而提高程式碼的可維護性和可讀性

### stimulus
提供一種輕量級的方式，將 JavaScript 添加到 Web 應用程序中，而無需過多的學習成本和複雜性

```js
rails webpacker:install:stimulus
```

使用 Stimulus 呈現時，你可以按照以下步驟來設置：

在 /app/javascript/controllers 創建一個 index_controller.js 的文件
```js
// app/javascript/controllers/index_controller.js

import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "content","output" ]

  connect() {
    console.log('hi')
  }

  greet() {
    this.outputTarget.textContent = `Hello, ${this.contentTarget.value}!`;
  }
}
```

使用 data-controller 屬性將 controller 連結到元素：
```js
<!--HTML from anywhere-->
<div data-controller="index">
  <input data-index-target="content" type="text">

  <button data-action="click->index#greet">
    Greet
  </button>

  <span data-index-target="output">
  </span>
</div>
```

:::danger note
注意，data-index-target 的值應與控制器中的 static targets **一致**
:::


## controller 整理
在 controller 裡面如果有添加條件語句，讓方法變得太長影響閱讀
```js
//controller
@stories = Story.where(status:'published').order(created_at: :desc).includes(:user)
```

建議把它建立 scope 在 model，再從 controller 引用
```js
//model
scope :published_story, -> { where(status: 'published') }

```
```js
//controller
@stories = Story.published_story.order(created_at: :desc).includes(:user)
```

## N+1 問題
with_attached 是 **Active Storage** 提供的方法，主要用於在**檢索記錄的同時獲取附件**，避免多次查詢附件表，提高性能

舉例來說，如果你的 Story model 中有一個附加的 cover_image，想要顯示在首頁，可以使用 `with_attached ` 方法來在查詢時加載附件
```js
class PagesController < ApplicationController
  def index
    @stories = Story.published.with_attached_cover_image.order(created_at: :desc).includes(:user)
  end
end
```

## 解決文章中的超連結
simple_format 用於將文本轉換為 HTML，同時保留段落和換行符號
```js
<%= simple_format(story.content)%>
```

## 關閉自動生成
在 config/application.rb 中設置 config.generators 來配置生成器（Generators）
```js
config.generators do |g|
  g.assets false
  g.helpers false
  g.test_framework false
end
```

* g.assets false: 禁用生成器創建 assets 文件（例如 JavaScript 和 CSS 文件）。這表示在創建控制器或視圖等時，不會自動生成與 assets 相關的文件

* g.helpers false: 禁用生成器創建 helper 文件。這表示在創建控制器時，不會生成相應的 helper 文件

* g.test_framework false: 禁用生成器創建測試框架相關的文件。這表示在創建組件時，不會生成測試文件（例如單元測試或集成測試）

這樣的配置可以提供更大的靈活性和控制權，手動編寫這些文件，而不是依賴生成器自動創建

## form_with
form_with 是 Rails 5 新增的表單輔助方法，用於簡化在 Rails 中的表單建立，取代了先前版本中的 form_for 和 form_tag 方法

在特定故事（story）的評論（Comment）上建立新的評論：
使用 Comment model 建立新的評論，提交後數據應該發送到指定的位置（路由）

* local: false：啟用遠端（Ajax）表單。當 local 設為 false 時，表單會被設置為非同步（Ajax），頁面不會重新加載
```js
<%= form_with model: Comment.new, url: story_comments_path(story), local: false do |form| %>
  <%= form.text_area :content %>
  <%= form.button '送出' %>
<% end %>
```