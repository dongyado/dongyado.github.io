---
layout: post
title:  SharedArrayBuffer 以及在跨域中的使用
date: 2021-09-19
categories:
- webassembly
tags: [webassembly]
---
### 什么是跨域
跨域问题是前端开发人员面前绕不过去的问题。  
所谓跨域，通俗点表达，就是当前站点，请求或加载了其他域名的资源。
比如当前 app.a.com 请求了  file.b.com 的资源，就产生了跨域资源的加载。

浏览器在渲染当前页面时，校验网页请求的资源是否可以安全的在当前页面使用。
比如：
1. 是同站点下的资源，比如 app.a.com/logo.png，可以直接使用
2. 如果是同域名下的资源，比如 file.a.com/logo.png, 这时会校验文件的响应头，看有没有配置 Access-Control-Allow-* 相关的访问控制参数
3. 如果是 file.b.com/logo.png 同样也会去校验访问控制信息信息

如果资源的访问控制信息不能表面资源可以在当前站点使用，就会产生跨域的问题，浏览器会禁止资源的加载和展示。
也就是说跨域问题其实是浏览器对资源使用安全策略产生的，浏览器要保证站点访问资源是安全，合法访问的。

浏览器是否可以使用某个资源有两个限制：
1. 当前站点是否允许使用其他站点，域名的资源
站点可以配置是否允许访问其他域名的资源，比如在站点配置一些安全策略

```
# COEP 跨域嵌入策略，会阻止页面加载任何未明确授予文档许可权的跨域资源（使用CORP或CORS）。
Cross-Origin-Embedder-Policy: require-corp
# COOP 
Cross-Origin-Opener-Policy: same-origin
```

2. 请求的资源是否可以在当前站点使用
资源是否开启了跨域资源共享策略，决定资源在哪些站点可以被使用，一般解决跨域的时候，需要在资源的响应头中配置以下选项

```
# 跨域资源策略
Cross-Origin-Resource-Policy: same-site | same-origin | cross-origin
# cross-origin 表示该资源被任何站点使用

# 允许通过哪些方法访问
Access-Control-Allow-Methods: GET | POST | PUT | OPTIONS | HEAD

# 允许哪些域名访问， * 代表所有，一般来说不安全
Access-Control-Allow-Origin: *
```

### 什么是 SharedArrayBuffer
SharedArrayBuffer(以下简称 SAB) 是一个 javascript 对象，用于网站线程之间的内存数据共享，比如 worker。同样由于 WebAssembly 使用 worker 模拟了多线程，所以在这种情况下同样会使用到 SAB 做数据共享访问。

### 为什么使用 SAB 会遇到跨域问题
18年 spectre 曝光之前，已经有不少网站使用了 SAB。由于 spectre 是 cpu 层面的漏洞，不太可能通过cpu 升级去解决，因此当时所有浏览器直接禁用了 SAB，来防止 spectre 的旁路攻击。  

那么什么是 spectre 攻击，用通俗一点的话来说，就是 cpu 有一个机制，叫做预测机制，假如有一个恶意程序去查询一个没有权限的信息，操作系统会返回禁止的信息，本身来说这个逻辑当然没有问题。但是当恶意程序询问时，其实操作系统即使返回了禁止的信息，但是还是会因为预测机制会去执行正确答案的逻辑，会导致慢了一小段时间，比如几微秒，几毫米，这就被恶意程序注意到了，通过每隔一段时间不断去尝试询问，达到获取正确答案的效果。  

恶意程序可以通过  SAB 获取到其他页面的敏感信息。我们都知道 worker 在浏览器中有很大的限制，比如不能访问 window, document 对象， SAB 是用来和主线程进行数据交换访问的高效方法，也被大量应用，这就会导致 worker 是有办法通过 SAB 攻击获取到主线程的敏感信息。  

所以在 spectre 漏洞曝出时，几乎所有主流浏览器都默认关闭了 SAB 以应对这个漏洞的攻击。

### 如何解决
SAB 作为 WebAssembly(wasm) 模拟多线程的基础组件，如果不能使用，将会对 wasm 在某些应用场景的性能大幅下降。重新启用是势在必行。  
在 chrome 92 之前， chrome 是默认开启 SAB 的，但是已经弹出警告信息提示 SAB 将会在 92 版本必须启用以下选项：

```
# COEP 跨域嵌入策略，会阻止页面加载任何未明确授予文档许可权的跨域资源（使用CORP或CORS）。
Cross-Origin-Embedder-Policy: require-corp
# COOP 
Cross-Origin-Opener-Policy: same-origin
```

也就是说加载的资源必须有 corp 策略的头信息， 跨域打开者也是同域名

### 相关资料
- 关于 SharedArrayBuffer spectre 攻击的讨论
https://github.com/tc39/security/issues/3
- 使用 SharedArrayBuffer 进行 Spectre 攻击的原理
https://www.giantbranch.cn/2018/01/09/%E8%85%BE%E8%AE%AF%E7%8E%84%E6%AD%A6%E5%AE%9E%E9%AA%8C%E5%AE%A4%E6%A3%80%E6%B5%8B%E6%B5%8F%E8%A7%88%E5%99%A8spectre%E7%9A%84%E5%8E%9F%E7%90%86/
- 为什么需要“跨域隔离”才能获得强大的功能
- Same origin : https://web.dev/same-origin-policy/
- 跨域隔离启用指南： https://web.dev/cross-origin-isolation-guide/
- https://v8.dev/blog/spectre
- https://web.dev/coop-coep/
- https://www.cnblogs.com/Shepherdzhao/p/8253421.html