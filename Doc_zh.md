# pjax = pushState + ajax


            .--.
           /    \
          ## a  a
          (   '._)
           |'-- |
         _.\___/_   ___pjax___
       ."\> \Y/|<'.  '._.-'
      /  \ \_\/ /  '-' /
      | --'\_/|/ |   _/
      |___.-' |  |`'`
        |     |  |
        |    / './
       /__./` | |
          \   | |
           \  | |
           ;  | |
           /  | |
     jgs  |___\_.\_
          `-"--'---'
##  [English DOC](./README.md)



## 简介

pjax 是用ajax 和pushState 实现的一个jquery插件 
pjax 的工作原理是通过ajax方式从服务器获取html片段然后填充到指定的容器中. 燃后用pushState更新浏览器当前的URL，不用重新加载整个页面。

对于 [不支持pushState的浏览器][compat] pjax 完全降级处理.

## 概述

pjax 是完全自动化的. 你只要指定用来替换的容器 就能实现页面无刷新导航.

例如：

``` html
<!DOCTYPE html>
<html>
<head>
  <!-- styles, scripts, etc -->
</head>
<body>
  <h1>My Site</h1>
  <div class="container" id="pjax-container">
    Go to <a href="/page/2">next page</a>.
  </div>
</body>
</html>
```

我们希望pjax 获取 URL `/page/2` 然后用返回的html片段替换 `#pjax-container`中的内容. 样式和脚本不会重载，  包括`<h1>`
可以保留其中的内容。 - 我们仅仅想改边`#pjax-container` 元素.

我们是通过监听`#pjax-container` 容器中的 `a` 标签来实现的：

``` javascript
$(document).pjax('a', '#pjax-container')
```

现在如果在支持pjax的浏览器中点击 "next page" `#pjax-container`容器将会被`/page/2`请求的html片段替换。

你还需要配置后台服务器，让他们根据pjax 选择性的返回html或者html片段

pjax  的请求会发送  `X-PJAX` 头 ，所以在这个案例中 (大多数情况下) 我们仅仅想加载容器中的部分，  所以就带这种X-PJAX 头请求.

在 Rails 中可能是下面的用法:

``` ruby
def index
  if request.headers['X-PJAX']
    render :layout => false
  end
end
```

如果你需要更多pjax 与 Rails 的相关信息 请查阅 [Turbolinks][].

或者 [RailsCasts #294: Playing with PJAX][railscasts].

## 安装

### bower

通过 [Bower][]:

```
$ bower install jquery-pjax
```

或者添加 `jquery-pjax` 到你的应用下的 `bower.json`文件.

``` json
  "dependencies": {
    "jquery-pjax": "latest"
  }
```

### 单独使用

下载 jquery.pjax.js 将jquery.pjax.js 的引用直接放在Jquery之后.

```
curl -LO https://raw.github.com/defunkt/jquery-pjax/master/jquery.pjax.js
```

**警告** 不要直接引用上述js地址. GitHub 不是 CDN.

## 依赖

需要 jQuery 1.8.x 或者更高版本.

## 适用范围

pjax 仅用于 [支持 `history.pushState`
API 的浏览器][compat]. 当这个API不支持时，进入回滚流程:
`$.fn.pjax` 的回调函数将失效 并且 `$.pjax` 将直接进入该URL地址.

为了调试这个特性, 你可以手动干预，如果你的浏览器支持 `pushState`. 方法是调用 `$.pjax.disable()`. 想要看看pjax是支持 `pushState`方式改变URL地址, 控制台输入`$.support.pjax`.

## 用法

### `$.fn.pjax`

最基本的使用方法:

``` javascript
$(document).pjax('a', '#pjax-container')
```

这表示所有的a链接都应用pjax 并且 指定`#pjax-container`作为容器.

如果你正在迁移一个已经存在的站点，你可能不想把所有的a链接启用pjax. 那么你可以给想启用pjax的链接加上`data-pjax`属性, 然后使用`'a[data-pjax]'` 作为你的选择器.

或者尝试使用这种选择器来匹配  `<div data-pjax>` 容器中的`<a data-pjax href=>`.

``` javascript
$(document).pjax('[data-pjax] a, a[data-pjax]', '#pjax-container')
```

#### 参数

`$.fn.pjax` 基本用法:

``` javascript
$(document).pjax(selector, [container], options)
```

1. `selector` 定义点击click[事件代理]的字符串[$.fn.on].
2. `container` 定义pjax 容器的唯一标识.
3. `options` 是一个对象参数集合，具体如下.

##### pjax options

key | 默认值 | 描述
----|---------|------------
`timeout` | 650 | ajax 请求超时的毫秒数，超过这个时间将会URL重载的方式进入
`push` | true | 导航的时候，使用 [pushState][] 来添加浏览器历史纪录
`replace` | false | 替换URL地址，不修改浏览器历史记录
`maxCacheLength` | 20 | 容器的最大缓存值
`version` | | 当前pjax的版本号
`scrollTo` | 0 | 导航之后垂直方向滚动到的值. 设为`false` 不改变滚动位置.
`type` | `"GET"` | see [$.ajax][]
`dataType` | `"html"` | see [$.ajax][]
`container` | | CSS selector for the element where content should be replaced
`url` | link.href | a string or function that returns the URL for the ajax request
`target` | link | eventually the `relatedTarget` value for [pjax events](#events)
`fragment` | | CSS selector for the fragment to extract from ajax response

You can change the defaults globally by writing to the `$.pjax.defaults` object:

``` javascript
$.pjax.defaults.timeout = 1200
```

### `$.pjax.click`

This is a lower level function used by `$.fn.pjax` itself. It allows you to get a little more control over the pjax event handling.

This example uses the current click context to set an ancestor as the container:

``` javascript
if ($.support.pjax) {
  $(document).on('click', 'a[data-pjax]', function(event) {
    var container = $(this).closest('[data-pjax-container]')
    $.pjax.click(event, {container: container})
  })
}
```

**NOTE** Use the explicit `$.support.pjax` guard. We aren't using `$.fn.pjax` so we should avoid binding this event handler unless the browser is actually going to use pjax.

### `$.pjax.submit`

Submits a form via pjax.

``` javascript
$(document).on('submit', 'form[data-pjax]', function(event) {
  $.pjax.submit(event, '#pjax-container')
})
```

### `$.pjax.reload`

Initiates a request for the current URL to the server using pjax mechanism and replaces the container with the response. Does not add a browser history entry.

``` javascript
$.pjax.reload('#pjax-container', options)
```

### `$.pjax`

Manual pjax invocation. Used mainly when you want to start a pjax request in a handler that didn't originate from a click. If you can get access to a click `event`, consider `$.pjax.click(event)` instead.

``` javascript
function applyFilters() {
  var url = urlForFilters()
  $.pjax({url: url, container: '#pjax-container'})
}
```

### 事件

所有的pjax事件除了`pjax:click` 和 `pjax:clicked` ，都是通过 pjax container 容器解除绑定的, 而不是你点击的元素.

<table>
<tr>
  <th>event</th>
  <th>cancel</th>
  <th>arguments</th>
  <th>notes</th>
</tr>
<tr>
  <th colspan=4>event lifecycle upon following a pjaxed link</th>
</tr>
<tr>
  <td><code>pjax:click</code></td>
  <td>✔︎</td>
  <td><code>options</code></td>
  <td>fires from a link that got activated; cancel to prevent pjax</td>
</tr>
<tr>
  <td><code>pjax:beforeSend</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>can set XHR headers</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:send</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:clicked</code></td>
  <td></td>
  <td><code>options</code></td>
  <td>fires after pjax has started from a link that got clicked</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>before replacing HTML with content loaded from the server</td>
</tr>
<tr>
  <td><code>pjax:success</code></td>
  <td></td>
  <td><code>data, status, xhr, options</code></td>
  <td>after replacing HTML content loaded from the server</td>
</tr>
<tr>
  <td><code>pjax:timeout</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>fires after <code>options.timeout</code>; will hard refresh unless canceled</td>
</tr>
<tr>
  <td><code>pjax:error</code></td>
  <td>✔︎</td>
  <td><code>xhr, textStatus, error, options</code></td>
  <td>on ajax error; will hard refresh unless canceled</td>
</tr>
<tr>
  <td><code>pjax:complete</code></td>
  <td></td>
  <td><code>xhr, textStatus, options</code></td>
  <td>always fires after ajax, regardless of result</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <th colspan=4>event lifecycle on browser Back/Forward navigation</th>
</tr>
<tr>
  <td><code>pjax:popstate</code></td>
  <td></td>
  <td></td>
  <td>event <code>direction</code> property: &quot;back&quot;/&quot;forward&quot;</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>before replacing content</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>right before replacing HTML with content from cache</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>after replacing content</td>
</tr>
</table>

`pjax:send` & `pjax:complete` are a good pair of events to use if you are implementing a
loading indicator. They'll only be triggered if an actual XHR request is made,
not if the content is loaded from cache:

``` javascript
$(document).on('pjax:send', function() {
  $('#loading').show()
})
$(document).on('pjax:complete', function() {
  $('#loading').hide()
})
```

An example of canceling a `pjax:timeout` event would be to disable the fallback
timeout behavior if a spinner is being shown:

``` javascript
$(document).on('pjax:timeout', function(event) {
  // Prevent default timeout redirection behavior
  event.preventDefault()
})
```

### Server side

Server configuration will vary between languages and frameworks. The following example shows how you might configure Rails.

``` ruby
def index
  if request.headers['X-PJAX']
    render :layout => false
  end
end
```

An `X-PJAX` request header is set to differentiate a pjax request from normal XHR requests. In this case, if the request is pjax, we skip the layout html and just render the inner contents of the container.

[Check if there is a pjax plugin][plugins] for your favorite server framework.

#### Response types that force a reload

By default, pjax will force a full reload of the page if it receives one of the
following responses from the server:

* Page content that includes `<html>` when `fragment` selector wasn't explicitly
  configured. Pjax presumes that the server's response hasn't been properly
  configured for pjax. If `fragment` pjax option is given, pjax will simply
  extract the content to insert into the DOM based on that selector.

* Page content that is blank. Pjax assumes that the server is unable to deliver
  proper pjax contents.

* HTTP response code that is 4xx or 5xx, indicating some server error.

#### Affecting the browser URL

If the server needs to affect the URL which will appear in the browser URL after
pjax navigation (like HTTP redirects work for normal requests), it can set the
`X-PJAX-URL` header:

``` ruby
def index
  request.headers['X-PJAX-URL'] = "http://example.com/hello"
end
```

#### Layout Reloading

Layouts can be forced to do a hard reload when assets or html changes.

First set the initial layout version in your header with a custom meta tag.

``` html
<meta http-equiv="x-pjax-version" content="v123">
```

Then from the server side, set the `X-PJAX-Version` header to the same.

``` ruby
if request.headers['X-PJAX']
  response.headers['X-PJAX-Version'] = "v123"
end
```

Deploying a deploy, bumping the version constant to force clients to do a full reload the next request getting the new layout and assets.

[compat]: http://caniuse.com/#search=pushstate
[$.fn.on]: http://api.jquery.com/on/
[$.ajax]: http://api.jquery.com/jQuery.ajax/
[pushState]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history#Adding_and_modifying_history_entries
[plugins]: https://gist.github.com/4283721
[turbolinks]: https://github.com/rails/turbolinks
[railscasts]: http://railscasts.com/episodes/294-playing-with-pjax
[bower]: https://github.com/twitter/bower
