---
description: 純手刻 ruby
---
# 純手刻 ruby
- 目標：建立一個投票網站
* 第一步：在畫面顯示

建立一個資料夾，並且運行他
```js
rails _6.1.4.6_ new rubyproject --webpack
rails s
```


接著到 `config/routes.rb` 新增來源（ 可以自訂 ）
```js
resources :candidates
```

到 controllers/ 新增 `candidates_controller.rb` 檔案，建立 CandidatesController 的類別並且繼承最上層
```js
class CandidatesController < ApplicationController
  
  def index //建立方法
  end
  
end
```

那接下來剩下畫面的問題，在 views 資料夾在建立一個資料夾 candidates，裡面放的是 erb 檔案，可以寫 ruby 語言的 html：index.html.erb( 也就是建立方法的名稱 )
```js
<h1>這是畫面顯示</h1>
```

進入 localhost:3000/candidates 這個網址，結果就會出現『 這是畫面顯示 』這段文字


* 第二步驟：建立 model 以及建立 migrate
```js
rails g model Candidate name party age:integer politics:text votes:integer
```

model 檔案
```js
class Candidate < ApplicationRecord
end
```
migrate：描述資料表 -> 記得要在終端機打上 `rails db:migrate`
```js
class CreateCandidates < ActiveRecord::Migration[6.1]
  def change
    create_table :candidates do |t|
      t.string :name
      t.string :party
      t.integer :age
      t.text :politics
      t.integer :votes, default:0

      t.timestamps
    end
  end
end
```

而 migrate 會建立在 `development.sqlite3` 這個檔案裡，效能雖然不好，但是簡單易用的檔案型資料庫


## form 裡的 token
在建立表單的地方，需要有一組 token，每次點擊每次都不同，這是為了防止有心人士灌水，所以 ruby 有內建的表單輸出自動產生 token : form_for
```js
<%= form_for(@candidate) do |form| %> 
//Candidate.new建立物件應交給 controller 來處理，view 就負責畫面的事情就好（已經在 controller 新增一個 new 方法存取在實體變數 ＠candidate）
<%= form.label :name %>
<%= form.text_field :name %>

<%= form.label :party %>
<%= form.text_field :party %>

<%= form.label :age %>
<%= form.text_field :age %>

<%= form.label :politics %>
<%= form.text_area :politics %>
<%= form.submit %>
<% end %>
```

新增一個 new 方法存取在實體變數 ＠candidate
```js
class CandidatesController < ApplicationController
  
  def index //建立方法
  end

  def new
    @candidate = Candidate.new //建立實體變數
  end
  
end
```

## 如何取得表單資料
因為在按下按鈕的同時，需要 create 的方法，這時在 create 方法新增一個 debugger，再到 rails 的環境中打上 `params` 會出現一堆資料，有 token、candidate 的 Hash 等等，我們需要的資料，也就是表單資料，就在這個 candidate 的 Hash 裡 ( params[:candidate] )
```js
class CandidatesController < ApplicationController
  
  def index
  end

  def new
    @candidate = Candidate.new
  end

  def create
    // debugger

    @candidate = Candidate.new(params[:candidate])

    if @candidate.save
      flash[:notice] = '建立成功' //快閃訊息
      redirect_to '/candidates'
    else

    end
  end
end
```

但是 ruby 內建有一個保護機制，只打這些還是會報錯，為了不讓他人隨意丟資料進去，所以在提交的同時，除了需要 token 之外，還需要確認資料的種類

```js
class CandidatesController < ApplicationController
  
  def index
  end

  def new
    @candidate = Candidate.new
  end

  def create
    // debugger

    @candidate = Candidate.new(candidate_params) //建立方法

    if @candidate.save
      flash[:notice] = '建立成功' //快閃訊息
      redirect_to '/candidates'
    else
      render :new 
    end
  end

  private //因為外部不需取用，所以使用私有方法
  def candidate_params
    params.require(:candidate).permit(:name, :party, :age, :politics) //資料只允許這四種
  end
end
```

`render :new` 重新渲染 new 頁面（也就是表單頁面），但是所 key 的資料一樣會保留，因為 create 裡的 @candidate 是帶有資料的

## link_to
這個方法像是 html 裡的 a 標籤的超連結，因為裡面要塞很多資料的話，使用 a 標籤會讓程式碼看起來很亂
```js
<%= link_to '想要顯示的名字',  '對照 routes 裡面的 path' %>

<%= link_to candidate.name,  candidate_path(candidate.id)%>
```

對照 routes 的方法，在專案終端機打上 `rails routes` 會出現 routes 對照表

而且使用 link_to 的話，如果在專案結束後，想要統一把網頁路由改名的話可以這麼做，在 config/routes.rb 新增 path 即可
```js
Rails.application.routes.draw do
  resources :candidates, path: 'member'
end
```

## find_by
用於在資料庫中查找滿足指定條件的第一條記錄

`:id` 是指查詢的條件，而 `params[:id]` 是從 HTTP 請求中獲取的參數，通常是 URL 中的路由參數，然後再把這項資料存進實體變數中
```js
@candidate = Candidate.find_by(id: params[:id])
```

## 如何顯示錯誤訊息
運用 `rails console` 這個命令，去檢查錯誤
* `.errors.any?` 詢問有無錯誤
* `.errors.full_messages` 列出錯誤訊息

```js
<% if @candidate.errors.any? %>
  <ul>
    <%= @candidate.errors.full_messages.each do |message| %>
    <li><%= message%></li>
    <% end %>
  </ul>
<% end %>
```

## 檢視及刪除顯示相同頁面怎麼辦
在 routes 的對照表中，檢視及刪除都使用同個路由，這樣一來會顯示同個畫面
```js
<td><%= link_to candidate.name, candidate_path(candidate.id) %></td>
<td><%= link_to 'delete', candidate_path(candidate.id) %></td>
```

解決方法：

在第二個 link_to 中，使用了 method: 'delete'，以便生成一個能夠觸發 DELETE 請求的刪除按鈕，也新增一個確認提示， data: { confirm: 'Are you sure?' } 提醒使用者確認是否要刪除

```js
<td><%= link_to candidate.name, candidate_path(candidate.id) %></td>
<td><%= link_to 'delete', candidate_path(candidate.id), method: 'delete' %></td>
```

## 新增路由方法
```jsx title="routes.rb"
Rails.application.routes.draw do
  resources :candidates

  post '/candidates/:id/vote', to:'candidates#vote'
end
```

但還是建議使用這種方式，在 resources 底下做擴充，有兩種方式：member、collection，差別在於有無產生 id
```jsx title="routes.rb"
Rails.application.routes.draw do
  resources :candidates do
    member do
      post :vote
      get :abc
    end
    //candidates/:id/vote

    collection do
      post :vote
    end
    //candidates/vote
  end
end
```

## schema

schema.rb 文件是通過運行 `rails db:migrate` 命令生成的。這個命令會執行項目中 db/migrate 目錄下的檔案，並根據遷移文件中的定義更新數據庫的結構。schema.rb 文件的內容會反映當前數據庫的實際結構

```jsx title="db/schema.rb"
ActiveRecord::Schema.define(version: 2023_11_24_143043) do

  create_table "candidates", force: :cascade do |t|
    t.string "name"
    t.string "party"
    t.integer "age"
    t.text "politics"
    t.integer "votes", default: 0
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

end
```

## 複數化
Rails 使用一種稱為複數化（pluralization）的方式，將 Model 名稱轉換成數據庫表名稱。`VoteLog` 是 Model 的名稱，而對應的數據庫表名稱被 Rails 複數化為 `vote_logs`，這是 Rails 有助於維持程式碼的一致性和可讀性

以下兩行範例呈現相同模式：
```js
VoteLog.create(candidate: @candidate,ip_address: request.remote_ip)
@candidate.vote_logs.create(ip_address: request.remote_ip)
```

## N+1 問題
只有 2 筆資料，但卻對資料庫做了 3 次的查詢，這樣子非常浪費系統的效能，要解決這個問題，可以使用 Rails 提供的 Counter Cache 做法

```js
Started GET "/candidates" for ::1 at 2023-11-29 15:41:07 +0800
Processing by CandidatesController#index as HTML
  Rendering layout layouts/application.html.erb
  Rendering candidates/index.html.erb within layouts/application
  Candidate Load (0.2ms)  SELECT "candidates".* FROM "candidates"
  ↳ app/views/candidates/index.html.erb:14
   (0.4ms)  SELECT COUNT(*) FROM "vote_logs" WHERE "vote_logs"."candidate_id" = ?  [["candidate_id", 4]]
  ↳ app/views/candidates/index.html.erb:17
   (0.3ms)  SELECT COUNT(*) FROM "vote_logs" WHERE "vote_logs"."candidate_id" = ?  [["candidate_id", 8]]
  ↳ app/views/candidates/index.html.erb:17
  Rendered candidates/index.html.erb within layouts/application (Duration: 13.7ms | Allocations: 2685)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 23.3ms | Allocations: 6235)
Completed 200 OK in 25ms (Views: 23.3ms | ActiveRecord: 0.9ms | Allocations: 6629)
```

首先在 Candidate Model 開一個 integer 型態的欄位，名稱叫做 vote_logs_count，我們使用 Migration 來做這件事：
在專案終端機打上 `rails g migration add_counter_to_candidate vote_logs_count:integer`

這時候 migration 的內容長這樣：
```js
class AddCounterToCandidate < ActiveRecord::Migration[6.1]
  def change
    add_column :candidates, :vote_logs_count, :integer
  end
end
```

執行： `rails db:migrate`

在 VoteLog Model 的 belongs_to 後面加上 counter_cache: true 參數：

```jsx title="vote_log.rb"
class VoteLog < ApplicationRecord
  belongs_to :candidate, counter_cache:true
end
```

最後在 index 檔案，顯示投票數的地方使用 size 方法：

  * size 方法可以應用在關聯的集合、查詢結果等地方
  * size 方法的使用方式類似於 count 方法
  * size 會儘可能在內存中計算集合的大小，而不去數據庫查詢，這對於已經加載到內存中的集合來說效率更高
  * 如果集合還沒有被加載，或者是一個條件較為複雜的查詢，Rails 可能會選擇使用 count 方法來發送一個查詢到數據庫
```js
<td><%= candidate.vote_logs.size %></td>
```

這樣一來，成功解決 n+1 的問題：只有抓取一次資料
```js
Started GET "/candidates" for ::1 at 2023-11-29 15:41:51 +0800
Processing by CandidatesController#index as HTML
  Rendering layout layouts/application.html.erb
  Rendering candidates/index.html.erb within layouts/application
  Candidate Load (0.1ms)  SELECT "candidates".* FROM "candidates"
  ↳ app/views/candidates/index.html.erb:14
  Rendered candidates/index.html.erb within layouts/application (Duration: 4.0ms | Allocations: 1230)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 11.8ms | Allocations: 4780)
Completed 200 OK in 13ms (Views: 12.4ms | ActiveRecord: 0.1ms | Allocations: 5174)
```


## 新增任務
lib/tasks 資料夾底下新增 `reset_counter.rake`檔案，相關的任務會放在 :db 命名空間下，而會使用 reset_counters 的原因是因為，目前的投票數跟畫面上的投票數沒有同步進行，需要手動同步計數器的值時，可以使用 reset_counters 方法

```jsx title="reset_counter.rake"
namespace :db do
  desc 'Reset Counter Cache!'
  task :reset_counter => :environment do
    Candidate.all.each do |candidate|
      Candidate.reset_counters(candidate.id, :vote_logs)
    end
  end
end
```
在專案終端底下輸入： `rails db:reset_counter` 這樣就可以同步了

## 方法重複如何整理

像這樣程式碼 有過多 `@candidate = Candidate.find_by(id: params[:id])` 這段程式碼，我們可以用方法包起來
```js
class CandidatesController < ApplicationController
  def index
    @candidates = Candidate.all
  end

  def show
    @candidate = Candidate.find_by(id: params[:id])
  end

  def new
    @candidate = Candidate.new
  end

  def create
    @candidate = Candidate.new(candidate_params)
  end

  def edit
    @candidate = Candidate.find_by(id: params[:id])
  end

  def update
    @candidate = Candidate.find_by(id: params[:id])
  end

  def destroy
    @candidate = Candidate.find_by(id: params[:id])
  end
end
```
像這樣用方法包起來，在每個方法裡加入方法，但這樣的方式還是讓程式碼有重複的文字存在
```js
class CandidatesController < ApplicationController
  def index
    @candidates = Candidate.all
  end

  def show
    find_candidate
  end

  def new
    @candidate = Candidate.new
  end

  def create
    @candidate = Candidate.new(candidate_params)
  end

  def edit
    find_candidate
  end

  def update
    find_candidate
  end

  def destroy
    find_candidate
  end

  def find_candidate
    @candidate = Candidate.find_by(id: params[:id])
  end
end
```

為了解決重複，可以使用 before_action 的方法使下面程式碼變得乾淨，除了 only 的用法之外，還有 except 的方法可以使用（與 only 相反）
```js
class CandidatesController < ApplicationController

  before_action :find_candidate, only: [:show, :edit, :update, :destroy, :vote]

  def index
    @candidates = Candidate.all
  end

  def show
  end

  def new
    @candidate = Candidate.new
  end

  def create
    @candidate = Candidate.new(candidate_params)
  end

  def edit
  end

  def update
  end

  def destroy
  end

  def find_candidate
    @candidate = Candidate.find_by(id: params[:id])
  end
end
```

## 表單重複如何整理
當兩個頁面同時出現以下程式碼，可以新增一個有下底線的檔案，例如：_form.html.erb，把重複的程式碼丟進去
```js
<% if @candidate.errors.any? %>
  <ul>
    <% @candidate.errors.full_messages.each do |message| %>
    <li><%= message%></li>
    <% end %>
  </ul>
<% end %>

<%= form_for(@candidate) do |form| %>
<div>
  <%= form.label :name %>
  <%= form.text_field :name %>
</div>
<div>
  <%= form.label :party %>
  <%= form.text_field :party %>
</div>

<div>
  <%= form.label :age %>
  <%= form.text_field :age %>
</div>

<div>
  <%= form.label :politics %>
  <%= form.text_area :politics %>
</div>


<%= form.submit %>
<% end %>
```

然後在想要的檔案裡加入`<%= render 'form'%>`，這樣就可以取用它。再來因為 _form.html.erb 檔案裡有 @candidate 的實體變數，要如何把它變成區域變數呢？

就是在想要的檔案裡面加入：`<%= render 'form', candidate: @candidate%>`，這樣做的好處之一是，它**讓視圖（即 _form.html.erb）能夠更容易地重用，而不依賴於調用它的上下文**

在 _form.html.erb 的檔案中就可以把 ＠ 去掉，使用 candidate 的區域變數了