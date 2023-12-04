---
description: ActiveRecord、資料表關聯、ORM、CRUD
---

# ActiveRecord
在 rails 中的 model 層，把資料表的一筆資訊包裝成一個物件，並可在物件上增加額外的邏輯操作，讓資料存取更便利
* 與資料庫的互動主要透過 ActiveRecord 提供的方法，這些方法包括查詢資料、新增、更新、刪除**( CRUD )**等操作
* 是 **Object Relational Mapping（ORM）** 框架，用在關聯型資料庫和應用程式之間建立、操作、處理資料，透過 ORM 可以更容易以物件的方式存取資料庫中的資料而不需直接撰寫 SQL

## CRUD
關於 Rails 的 CRUD，指的是操作資料庫的四種基本操作， **Create、Read、Update、Delete**

### Create
  * new : 不會寫進資料庫
  * create : 會寫進資料庫
  * create! : 嚴格模式

### Read
  * first
  * last
  * find_by(id: 1) : 找不到時出現 nil
  * find : 找不到時出現 exception（異常）
  * find_each : 預設一次跑 1000 筆資料
    ```js
    Product.find_each do |product|
      //...
    end
    ```
  * all
  * select : 只選取 name 欄位
    ```js
    Product.select('name')
    ```
  * where : 找出所有 name 欄位是 ruby 的資料
    ```js
    Product.where(name: 'ruby')
    ```
  * order : 依照 id 大小反向排序
    ```js
    Product.order('id DESC')
    Product.order(id: :desc)
    ```
  * limit : 只取出五筆資料
    ```js
    Product.limit(5)
    ```
  * count
  * sum
  * average
  * maximum
  * minimum

### Update
  * save
  * update
  * update_attributes
  * update_all
  * increment
  * decrement
  * toggle
### Delete
  * delete
    * 直接刪除
  * destroy
    * 刪除過程會有 callback
  * destroy_all(condition = nil)
    * 可以附加條件
    ```js
    Product.destroy_all("price < 10")
    ```

## rails 支援的關聯
* has_one
  * 不是設定是一個**類別方法**
  * 表示 model 一對一的關係

如果有一個 User model，並且讓每個使用者只擁有一個檔案，可以使用 has_one 關聯，並且確保 profile 表有 user_id 的欄位，才能使用 belongs_to 聲明它屬於 User model
```js
# models/user.rb
class User < ApplicationRecord
  has_one :profile
end

# models/profile.rb
class Profile < ApplicationRecord
  belongs_to :user
end
```

執行 `has_one :profile`，會動態產生好幾個方法：`profile`, `profile=`, `build_profile`, `create_profile`

* belongs_to
  * 是一個**類別方法**
  * 表示 model 一對一或多對一的關係

執行 `belongs_to :user`，會動態產生好幾個方法：`user`, `user=`


* has_many
  * 是一個**類別方法**
  * 表示 model 一對多的關係

* has_one :through
  * 表示 model 建立間接一對一關係

在這個例子中，User 通過 Profile 這個中介 model，建立了和 Address 的一對一關係。這樣一來，你可以透過 user.address 直接訪問 Address model，而不需要直接在 User model 中定義 belongs_to 關係
```js
# app/models/user.rb
class User < ApplicationRecord
  has_one :profile
  has_one :address, through: :profile
end

# app/models/profile.rb
class Profile < ApplicationRecord
  belongs_to :user
  has_one :address
end

# app/models/address.rb
class Address < ApplicationRecord
  belongs_to :profile
end

```
* has_many :through
  * 表示 model 多對多的關係
  * 需要第三個資料表來儲存兩邊資訊

假設有學生（Student）、課程（Course），以及一個中介模型 Enrollment 來建立學生和課程之間的多對多關係。可以這樣定義 model ：
```js
# app/models/student.rb
class Student < ApplicationRecord
  has_many :enrollments
  has_many :courses, through: :enrollments
end

# app/models/course.rb
class Course < ApplicationRecord
  has_many :enrollments
  has_many :students, through: :enrollments
end

# app/models/enrollment.rb //第三個資料表
class Enrollment < ApplicationRecord
  belongs_to :student
  belongs_to :course
end
```

* has_and_belongs_to_many (HABTM)
  * 表示 model 多對多的關係
  
使用這種方式建立多對多關係時，必須建立一個中間表，以儲存兩者之間的關係。在這個例子中，中間表的名稱應該是 courses_students （migration），按照字母順序排列並使用複數形式。這個表會存儲 student_id 和 course_id 之間的對應關係
```js
# app/models/student.rb
class Student < ApplicationRecord
  has_and_belongs_to_many :courses
end

# app/models/course.rb
class Course < ApplicationRecord
  has_and_belongs_to_many :students
end

```

## Scope
* 簡化使用時的邏輯
* 簡化 Controller 的程式碼 
* 類別方法
  ```js
  class Article < ApplicationRecord
    def self.published
      where(published: true)
    end
  end

  ```

在 model 定義了一個名為 published 的 Scope，使用了 where 方法，表示我們只想獲取 published 屬性為 true 的文章
```js
class Article < ApplicationRecord
  // 定義一個名為 published 的 Scope
  scope :published, -> { where(published: true) }
end
```

在 Controller 中調用 Scope
```js
class ArticlesController < ApplicationController
  def index
    # 使用 Scope 獲取所有已發布的文章
    @published_articles = Article.published
  end
end
```

## Action Mailer
在 Rails 中，Mailer 是處理郵件發送的部分。可以使用 Mailer 來生成和發送電子郵件，那要怎麼查詢 rails 的內建功能呢？ 在終端機輸入 `rails g` 這樣就可以知道 rails 內建有什麼功能囉，接著在終端機輸入 `rails g mailer Vote` 會新增兩個檔案：`mailers/vote_mailer.rb` & `views/vote_mailer`

在 `mailers/vote_mailer.rb` 的檔案裡，建立 vote_notify 的方法
```js
class VoteMailer < ApplicationMailer
  def vote_notify(email)
    mail to: email, subject: 'test email'
  end
end
```

當你想發送郵件時，可以在 controller 或其他地方呼叫這個 vote_notify 方法
```js
class CandidatesController < ApplicationController
  def vote
    # send mail

    VoteMailer.vote_notify('cocolulu2327@gmail.com').deliver

  end
end
```
可以在 views/vote_mailer 目錄下創建與方法名對應的 view 文件
```jsx title="vote_notify.html.erb"
<h1>test email</h1>
```
因為寄發信件需要配置 Action Mailer 的 `delivery_method` 和相應的 `smtp_settings`，以指定郵件的發送方式和 SMTP 伺服器的設置

在 rails guides 的 ActionMailer Basic 可以找到相關文件需要的資料，並且確保在 config/environments/development.rb 文件中配置郵件寄發服務：
```js
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:         'smtp.gmail.com',
  port:            587,
  domain:          'example.com',
  user_name:       '<username>',
  password:        '<password>',
  authentication:  'plain',
  enable_starttls: true,
  open_timeout:    5,
  read_timeout:    5 
  }
```

delivery_method 設置為 :smtp，表示希望使用 SMTP 來發送郵件，這是因為 Action Mailer 可以使用不同的方式來發送郵件，例如 SMTP、Sendmail、Test（僅用於測試）等，而 smtp_settings 則包含了 SMTP 伺服器的詳細設置，包括伺服器地址、埠口、用戶名、密碼等

但由於安全性考慮，不可能將密碼放到程式碼中，所以需要寄發信件的套件，例如：mailgun，可以產生 username password port 及 hostname

## ActiveJob
有些程式在執行的時間較長，等了一陣子後他才會運作到下一個工作，這樣不僅會影響到使用者體驗，更會在效能上出現很大的問題，所以像這種會影響使用者體驗的工作，會把要執行的工作先存起來，然後跑下一步，等到空檔或是指定時間再去執行存下來的工作，可以使用 rails 的內建功能 ActiveJob 來處理這個問題

根據以上寄發信件的例子來看：

執行 `rails g job VoteMail` 命令，生成一個新的 job

會在 app / jobs 目錄下創建一個新的文件 `vote_mail_job.rb`

```js
class VoteMailJob < ApplicationJob
  queue_as :default //表示工作著急程度：low_priority、default、urgent

  def perform(*args)
    VoteMailer.vote_notify('cocolulu2327@gmail.com').deliver
    //把剛剛跑很慢的寄發信件貼過來
  end
end
```

然後在 controller 的地方調度 Job
```js
class CandidatesController < ApplicationController
  def vote
    # send mail

    VoteMailJob.perform_later

  end
end
```


