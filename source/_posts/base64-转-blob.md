---
title: base64 转 blob
date: 2017-10-22 21:44:12
tags:
- JavaScript
categorles:
- 笔记📒
---

```javascript
var arr = base64.split(','), mime = arr[0].match(/:(.*?);/)[1],
    bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
    while (n--) {
        // charCodeAt() 方法可返回指定位置的字符的 Unicode 编码
        u8arr[n] = bstr.charCodeAt(n);
    }
// 'application/octet-binary' (默认值)
let blob = new Blob([u8arr], { type: mime })
```

#### Base64 的编码码和解码
使用atob 和 btoa 方法.

- atob() 解码

```javascript
atob("amF2YXNjcmlwdA==")
// 解码结果 "javascript"
```

- btoa() 编码

```javascript
window.btoa(javascript)
// 转码结果 "amF2YXNjcmlwdA=="
```

以上两种方法对于中文是有局限性的, 解决如下:

```javascript
var str = "China，中国";

// 先用 encodeURI() 编码
window.btoa(window.encodeURIComponent(str))
// "Q2hpbmElRUYlQkMlOEMlRTQlQjglQUQlRTUlOUIlQkQ="

// atob 解码 Base64 再用 decodeURI() 解码
window.decodeURIComponent(window.atob('Q2hpbmElRUYlQkMlOEMlRTQlQjglQUQlRTUlOUIlQkQ='))
// "China，中国"
```

Created on 2017-9-4 by Cara*