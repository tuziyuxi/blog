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

可以看到，form-data格式可以上传File,对的，这就是最大的区别。

### 请求体内容格式区别

```
    @RequestMapping(value = "/test")
    public void test(HttpServletRequest request) {
        System.out.println("请求头：");
        String header = request.getHeader("content-type");
        System.out.println("content-type:" + header);
        System.out.println("请求体：");
        try (BufferedReader reader = request.getReader()){
            String s;
            while ((s = reader.readLine()) != null) {
                System.out.println(s);
            }
            reader.close();
        } catch (Exception e){
            e.printStackTrace();
        }
    }
```

使用上面一段代码打印出请求类型和请求体，结果见下图：

{% asset_img form-data-request.png  %}

form-data请求的请求体格式复杂（下面一个是文件流，没有截全）

{% asset_img x-www-form-urlencoded-request.png  %}

x-www-form-urlencoded请求体中的格式就和get请求一样，参数键值对拼接放在请求体中

### Spring MVC中支持multipart/form-data格式

需要配置MultipartResolver

```
    @Bean
    public MultipartResolver multipartResolver(){
        CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
        multipartResolver.setMaxUploadSize(1000000);
        return  multipartResolver;
    }
```

### 用实体接收参数后，再从request中读取不到请求体

```
    @RequestMapping(value = "/test1")
    public void test1(DemoObj obj, HttpServletRequest request) {
        System.out.println(obj.toString());
        System.out.println("请求头：");
        String header = request.getHeader("content-type");
        System.out.println("content-type:" + header);
        System.out.println("请求体：");
        try(InputStream is = request.getInputStream()) {
            StringBuilder sb = new StringBuilder();
            byte[] b = new byte[4096];
            for (int n; (n = is.read(b)) != -1;) {
                sb.append(new String(b, 0, n));
            }
            System.out.println(sb.toString());
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
```

结果：

```
DemoObj{id=1, name='hello'}
请求头：
content-type:multipart/form-data; boundary=----WebKitFormBoundaryPh2Rl1iCPOcVhss9
请求体：

```

```
DemoObj{id=1, name='hello'}
请求头：
content-type:application/x-www-form-urlencoded
请求体：

```

再从request中读取请求体失败，是因为在这之前已经读过一次流，具体Controller如何接收参数，后续有时间再去研究源码吧。

也可参考下[这篇文章](https://www.cnblogs.com/sunny3096/p/7215906.html)


