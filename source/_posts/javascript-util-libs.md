---
title: JavaScript 实用工具库
tags:
  - utils
categories:
  - javascript
date: 2018-04-12 23:48:22
---

### 精确类型检查
```javascript
function typeOf (obj) {
  const toString  = Object.prototype.toString;
  return toString.call(obj).slice(8, -1)
}
```
