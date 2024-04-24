---
description: Nested Forms
---
# Nested Forms 巢狀表單
需要一個表單內嵌套另一個表單，需要在同一個表單中同時建立與編輯兩者來自不同 model 的資料

## 建立 model
```js
rails g model Sku product:references spec quantity:integer deleted_at:datetime:index
```

### 相應的 migration
```js
class CreateSkus < ActiveRecord::Migration[6.1]
  def change
    create_table :skus do |t|
      t.belongs_to :product, null: false, foreign_key: true
      t.string :spec
      t.integer :quantity
      t.datetime :deleted_at

      t.timestamps
    end
    add_index :skus, :deleted_at
  end
end
```

### 建立資料表
```js
rails db:migration
```

### 設定 model 關聯
```js
class Product < ApplicationRecord
  has_many :skus
  accepts_nested_attributes_for :skus, reject_if: :all_blank, allow_destroy: true
end
```

在表單裡面再添加表單，要使用 `form.fields_for`
```js
<%= form_with model: product do |form| %>
  <%= form.fields_for :skus do |sku| %>
    <%= render 'sku_fields', form: sku %>
  <% end %>
<% end %>
```

## controller 參數添加

在 controller 添加允許的參數，使用 `accepts_nested_attributes_for` 關聯的 model，在表單提交的時候可以添加 `_destroy` 屬性來標記是否需要刪除，在提交的 `_destroy` 中設置 1，rails 就會知道要刪除這個 sku

```js
def product_params
  params.require(:product).permit(skus_attributes: [:id, :spec, :quantity, :_destroy])
end
```

## 新增自動跳出表單

### 連結 stimulus

* `data-controller` 與要建立的 stimulus 檔案同名：product_controller.js
* `data: { action: 'product#addSku' }`: 在 product_controller.js 檔案建立 addSku() 的點擊事件
* `data-product-target`: 建立名為 link 的目標元素，可以使用 `this.linkTarget` 來獲取元素

```js
<div data-controller="product">
  <%= form.fields_for :skus do |sku| %>
    <%= render 'sku_fields' , form: sku %>
  <% end %>

  <div data-product-target="link">
    <%= link_to '新增品項' , '#' , class:'button is-light is-small', data: { action: 'product#addSku' } %>
  </div>
</div>
```

```js
import { Controller } from "stimulus"

export default class extends Controller {

  addSku(e){
    e.preventDefault();
  }
}
```

### 在表單內插入 template
* template: 定義 html 的內容模板，在畫面上不會顯示
* child_index: 用於設置子模型在表單中的索引值

```js
<div data-controller="product">

  <template data-product-target="template">
    <%= form.fields_for :skus, Sku.new, child_index: 'NEW_RECORD' do |sku| %>
      <%= render 'sku_fields' , form: sku %>
    <% end %>
  </template>

  <%= form.fields_for :skus do |sku| %>
    <%= render 'sku_fields' , form: sku %>
  <% end %>

  <div data-product-target="link">
    <%= link_to '新增品項' , '#' , class:'button is-light is-small', data: { action: 'product#addSku' } %>
  </div>
</div>
```
### 監聽事件
要確保 child_index 的值是唯一值，所以要在 product_controller.js 內把 NEW_RECORD 改成當前的時間戳記，並且在新增品項之前用 insertAdjacentHTML 插入 `templateTarget` 的新內容，這樣就可以按下新增品項後，跳出可以填寫的 input 框
```js
addSku(e){
  e.preventDefault();
  let content = this.templateTarget.innerHTML.replace(/NEW_RECORD/g, new Date().getTime())
  this.linkTarget.insertAdjacentHTML('beforebegin', content)
}
```

### 刪除 input
接下來要做的是按下刪除鍵：
這個是資料的 input 框，在最外框自定義 `data-new-record`，在裡面插入 `form.object.new_record?` 方法，判斷是否為新紀錄，並且在表單建立一個隱藏字段 `_destroy`，來刪除紀錄

```js
<div class="columns nested-fields" data-new-record="<%= form.object.new_record? %>">
  <div class="column is-6">
    <div class="field">
      <%= form.text_field :spec, class: 'input', placeholder:'規格' %>
    </div>
  </div>
  <div class="column is-2">
    <div class="field">
      <%= form.number_field :quantity, class: 'input', placeholder:'數量' %>
    </div>
  </div>
  <div class="column is-1">
    <div class="field">
      <%= link_to '刪除', '#', class:'button is-danger', data: { action: 'product#removeSku'}%>
    </div>
  </div>
  <%= form.hidden_field :_destroy %>
</div>
```

### 刪除紀錄
* closest: 尋找最接近的元素
* dataset.newRecord: 在畫面上自定義的 `data-new-record`
* input[name*='_destroy']: 畫面上建立的 `_destroy` 隱藏字段，值設置為 1，rails 就會根據這個值來刪除對應的元素，也就是 nested-fields 的 input 框
```js
removeSku(e){
  e.preventDefault();
  let wrapper = e.target.closest('.nested-fields')
  if (wrapper.dataset.newRecord == 'true'){
    wrapper.remove()
  }else{
    wrapper.querySelector("input[name*='_destroy']").value = 1
    wrapper.style.display = 'none'
  }
}
```