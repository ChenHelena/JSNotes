---
description: 非同步 JavaScript 和 XML (AJAX) 是 Web 應用程式開發技術的組合
---

# 什麼是 AJAX？
每當您的使用者與 Web 應用程式互動時，瀏覽器就會與遠端伺服器交換資料。資料交換可能會導致頁面重新載入並中斷使用者體驗。AJAX 這個技術和後端的 Web API 溝通進行資料交換，AJAX 可以讓網頁能夠只更新需要的部分，而無須重新載入整個頁面。

## AJAX 實現方式
### XMLHttpRequest（XHR）
XMLHttpRequest（XHR）是一組API函式集，可被 JavaScript 呼叫。不過 XHR 算是一個舊的方法，且它不支援 Promise，因此會造成複雜的 callback hell。

### fetch（ IE 不支援）
fetch 是 JavaScript 的原生 API，透過 fetch 我們可以向 HTTP Request。fetch 是基於 Promise 的，所以用法上會接 .then，用法上比 XHR 精簡，但是資料回傳後，還要再加一步 response.json() 轉成 JSON 格式。

### Axios
* 底層是由 XMLHttpResquest 做出來的
* code 精簡

一樣支援 Promise 語法，且能夠自動轉換 JSON 數據，主流瀏覽器都能支援 Axios。


```js
axios.get('url')
    .then((res) => {
      console.log(res.data)
    })
    .catch((err) => {
      console.log(err)
    })
```
` .get : 請求方法`

` url : API 路徑`

`.then() : 伺服器成功回應`

`.catch() : 伺服器失敗回應`