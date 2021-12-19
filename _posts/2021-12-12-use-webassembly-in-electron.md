---
layout: post
title:  在 Electron 中使用 WebAssembly
date: 2021-12-12
categories:
- webassembly
tags: [webassembly]
---
Wasm 在浏览器的使用日趋常见，被经常用来做一些图像，音频，绘图，性能相关的处理。而 electron 本身就是使用 Chromium 内核的桌面应用 app 开发框架，自然也可以使用 wasm。

### 引入 wasm
Electron 打包之后访问的是本地 html 文件，一般编译完成后引入在 html 里面直接引入:
```
<script src="/path/to/wasmlib.js" ></script>
```
wasmlib.js 会自动加载 wasmlib.wasm 文件并初始化，通过 Module 提供的回调函数获取初始化结果的通知

```
Module.onRuntimeInitialized = async _ => {
    // wasm lib initialized 
}
```

### SharedArrayBuffer 报错
SharedArrayBuffer被经常用于 worker 中的数据共享，如果 wasm 中有使用 ，在高版本浏览器（chrome 92 开始)上不能使用，会直接提示并报错:

```
[Deprecation] SharedArrayBuffer will require cross-origin isolation as of M92, around July 2021. See https://developer.chrome.com/blog/enabling-shared-array-buffer/ for more details.
Uncaught ReferenceError: SharedArrayBuffer is not defined
    at wasmlib.js:1074
```

这是因为在高版本上，chrome 对 SharedArrayBuffer 使用启用了更严格的安全限制，必须在服务器上配置了以下选项才能被使用：
```
# COEP 跨域嵌入策略，会阻止页面加载任何未明确授予文档许可权的跨域资源（使用CORP或CORS）。
Cross-Origin-Embedder-Policy: require-corp
# COOP 
Cross-Origin-Opener-Policy: same-origin
```
以上配置一般只能在 web 服务器配置，所以最初的方案是在 electron 内部启用个 web 服务器增加以上的头信息，然后通过 http 协议加载应用文件（相当于打开个在线网站），但是会带来很明显的性能问题，首页打开会非常慢。

### 最终解决方案
1. 使用 electron 提供的 session 设置增加返回头信息:
```
 session.defaultSession.webRequest.onHeadersReceived((details, callback) => {
        callback({
            responseHeaders: {
                ...details.responseHeaders,
                'Cross-Origin-Opener-Policy': 'same-origin',
                'Cross-Origin-Embedder-Policy': 'require-corp',
            },
        });
    });
```
2. 通过 BrowserWindow.LoadFile("/path/to/app.html") 加载本地 html 文件。  
基于此，app 不需要内嵌一个 web 服务器，本地文件也可以使用SharedArrayBuffer 的功能，速度也得到很大的提升。