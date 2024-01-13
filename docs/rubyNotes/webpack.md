---
description: webpack
---

# 如何將 Bootstrap 與 Webpack 和 Rails 結合使用
## 建立新專案
建立一個新的專案
```js
rails _6.1.4.6_ new twitter-clone
```

因為用 _6.1.4.6_ 這個版本新增的專案，默認使用 Webpacker 來管理 Webpack 這時候會看到 Gemfile 裡面：
```js
gem 'webpacker', '~> 5.0'
```
## 彈跳視窗及下拉選單的設定
接著在終端機打上：
```js
yarn add bootstrap jquery popper.js
```

:::danger note
目的是將 bootstrap 依賴的 javascript 庫（jquery 和 popper.js）添加進來，安裝到 node_modules
:::

打開專案目錄，在 config/webpack/environment.js 新增 webpack 的配置設定：
```js
const  webpack = require("webpack")

environment.plugins.append("Provide",new webpack.ProvidePlugin({
  $: 'jquery',
  jquery: 'jquery',
  Popper:['popper.js', 'default']
}))
```

* 使用 ProvidePlugin 的插件，以便在整個環境中可以使用 jquery 和 Popper.js
* popper 的設置是控制 bootstrap 的彈跳視窗及下拉選單

## 監聽對象
在 app/javascript/packs/application.js 新增：
```js
import "bootstrap"
import "../stylesheets/application" //此時尚未建立

document.addEventListener("turbolinks:load", () => {
  $('[data-toggle="tooltip"]').tooltip();
  $('[data-bs-toggle="popover"]').popover()
})
```

在 Turbolinks 的頁面加載完成後，確保整個專案都能使用 bootstrap，一併開啟 tooltip（提示框） 及 popover（彈跳框） 的功能

## 導入樣式
最後，在 app/javascript 新增 stylesheets 的資料夾，裡面放入 application.scss 來引入 bootstrap 的樣式
```js
@import "bootstrap/scss/bootstrap";
```

## 客製化樣式
如果要客製化 css 樣式，也另外開一個 style.scss 檔案放入 stylesheets 資料夾，這時候的 application.scss 這樣就可以客製化自己想要的樣式囉
```js
@import "bootstrap/scss/bootstrap";
@import "./style.scss";
```
## bootstrap 5 不支援 jquery
:::danger note
因為在 bootstrap 5 中，已經移除了對 jquery 的依賴，所以造成 .tooltip() is not a function 一直報錯
:::

**解決方式：**

在 application.js 裡修改成 javascript 原生的方式寫成，然後改變 bootstrap 引入的方式，`* as bootstrap`，這樣在使用時就可以透過 bootstrap 的對象來使用它的功能
```js
import * as bootstrap from "bootstrap"
document.addEventListener("turbolinks:load", () => {
  var popoverTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="popover"]'))
  var popoverList = popoverTriggerList.map(function (popoverTriggerEl) {
    return new bootstrap.Popover(popoverTriggerEl)
  })

  var tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'))
  var tooltipList = tooltipTriggerList.map(function (tooltipTriggerEl) {
    return new bootstrap.Tooltip(tooltipTriggerEl)
  })
})
```

那麼 environment.js 裡面的設定就都可以清除，因為他是為了要使用 jquery 而設定的，目前檔案裡只留下原本有的設定檔就可以囉
```js
const { environment } = require('@rails/webpacker')

module.exports = environment
```