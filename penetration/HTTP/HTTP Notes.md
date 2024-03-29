## GET->POST

1、修改GET为POST

2、移除URL中的查询字符串

3、添加请求头Content-Type，例如：

```http

Content-Type:Content-Type: text/xml; charset=utf-8

//对于表单数据为：application/x-www-form-urlencoded
对于json数据为：application/json
```

4、添加Content-Length

5、添加POST表单内容

例如：

```http
GET /submit?name=John&age=30 HTTP/1.1
Host: example.com
```

```http
POST /submit HTTP/1.1
Host: example.com
Content-Type: text/xml; charset=utf-8
Content-Length: 15

name=John&age=30
```

