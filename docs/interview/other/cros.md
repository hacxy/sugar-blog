# 什么是跨域？为什么浏览器要使用同源策略？你有几种方式可以解决跨域问题？了解预检请求嘛？

## 什么是跨域？
浏览器有同源策略,如果:``协议``,``端口``,``域名``有一个不同就是跨域

## 为什么浏览器要使用同源策略?
出于安全考虑,为了防止CSRF(跨站请求伪造)攻击:CSRF 攻击是利用用户的登录态发起恶意请求

## 跨域请求是否发出去了
请求是发出去了，但是浏览器拦截了响应。浏览器认为这不安全，所以拦截了响应。也就说明了跨域并不能完全阻止 CSRF，因为请求毕竟是发出去了。

## 你有几种方式可以解决跨域问题？
### jsonp
利用`` <script> ``标签没有跨域限制的漏洞。通过 ``<script>`` 标签指向一个需要访问的地址并提供一个回调函数来接收回调数据

**简单使用**
```html
<script>
    // jsonp的回调函数
    function jsonpCallback(data) {
    	console.log(data)
	}
</script>
<!-- 这个为发起的请求 -->
<script src="xxxxx?param1=1&param2=2&callback=jsonpCallback"></script>
<!-- 可以通过js脚本控制 -->
<script>
let $srcipt = document.createElement('script')
$srcipt.src = 'xxxxx?param1=1&param2=2&callback=jsonpCallback'
document.body.appendChild($srcipt)
</script>
```
**封装一个jsonp**

```html
<script>
    function jsonp(url, jsonpCallback, success) {
        const $script = document.createElement('script')
        $script.src = url + `&callback=${jsonpCallback}`
        $script.async = true
        $script.type = 'text/javascript'
        window[jsonpCallback] = function (data) {
            success && success(data)
        }
        document.body.appendChild($script)
    }
    btn.addEventListener('click', function (e) {
        jsonp(`http://xxx.xxx?param1=1&param2=2`, 'jsonpCallback', function (res) {
            console.log(res)
        })
    })
</script>
```

## CORS
服务端设置 Access-Control-Allow-Origin 就可以开启 CORS

以node为例子
```js
const http = require('http')

let server = http.createServer(async (req, res) => {
    //  -------跨域支持-----------
    // 放行指定域名
    res.setHeader('Access-Control-Allow-Origin', '*')
    //跨域允许的header类型
    res.setHeader("Access-Control-Allow-Headers", "*")
    // 允许的方法
    res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS')

    let { method, url } = req
    if (method === 'OPTIONS') {
        return res.end()
    }
    console.log(method, url)
    if (url === '/api/expires') {
        // 设置过期时间
        res.setHeader('Expires', 'Tue Mar 10 2020 11:11:28 GMT+0800')
        return res.end(`Expires---${new Date()}`)
    }
    if (url === '/api/cache') {
        // 设置10s后过期,需重新请求
        res.setHeader('Cache-control', 'max-age=10')
        return res.end(`Cache-control---${new Date()}`)
    }
    res.end('success')
})

// 启动
server.listen(3000, err => {
    console.log(`listen 3000 success`);
})
```

## Nginx
通过设置反向代理进行请求的转发

## document.domain
二级域名相同的情况下，比如 a.sugarat.top 和 b.sugarat.top 适用于该方式。

只需要给页面添加 document.domain = 'sugarat.top' 表示二级域名都相同就可以实现跨域