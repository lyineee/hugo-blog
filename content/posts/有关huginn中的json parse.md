---
title: "有关huginn中的json parse"
date: 2022-01-17T21:50:00+08:00
tags: ['huginn']
---
huginn中 post agent 返回的 body 都是没有 json 格式的，这时候可以使用 javascript agent 中的 JSON.parse 函数来将字符串格式的 body 转换成 json ，但要注意，如果字符串中的 json 中有换行符(\n)可能会出现报错

`[00:00:00] ERROR -- : JavaScript error: SyntaxError: Unexpected token`

本来 JSON.parse 函数在报错时应该写出这个 `Unexpected token` 在哪里，但由于出错的符号是换行符就会出现这样的报错

## 解决方法

```javascript
JSON.parse(data.body.replace(/\n/g, '\n'))
```
