---
title: postman中form-data和x-www-form-urlencoded的区别
date: 2018-07-02 00:45:00
categories:
- 常见问题
tags:
- Spring MVC
- postman
- HTTP
---
### 对应的请求头中的content-type

* form-data 对应 multipart/form-data
* x-www-form-urlencoded 对应 application/x-www-form-urlencoded

### postman上的直观区别

![form-data](/images/form-data.png)
{% asset_img x-www-form-urlencoded.png x-www-form-urlencoded格式图片 %}