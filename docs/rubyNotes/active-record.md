---
description: 資料表關聯、ORM、CRUD
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

