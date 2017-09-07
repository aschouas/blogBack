---
title: js获取字符长度
date: 2017-09-07 17:11:06
tags: javascript
toc: true
categories: 技术研究
---

<blockquote class="blockquote-center">心跳</blockquote>

js获取实际字符长度

<!--more-->

```js
var xintiao = {};
xintiao.GetLength = function(str) {
    var realLength = 0, len = str.length, charCode = -1;
    for (var i = 0; i < len; i++) {
        charCode = str.charCodeAt(i);
        if (charCode >= 0 && charCode <= 128) realLength += 1;
        else realLength += 2;
    }
    return realLength;
 

};	
xintiao.GetLength('xintiao心跳')
```

**路漫漫**