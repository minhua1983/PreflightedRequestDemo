# PreflightedRequestDemo

关于自定义请求头的跨域ajax问题：

参考了如下文章：
http://levy.work/2016-09-01-why-got-options-request-via-ajax/

请求分为如下2种：  
sample request，普通请求  
preflighted request，预检请求（OPTIONS请求）

请求如果js做了跨域ajax请求，无论是post还是get。服务器端得设置：
Access-Control-Allow-Origin:*

如果js的跨域ajax使用了非标准的contentType（标准：application/x-www-form-urlencoded, multipart/form-data, 或text/plain），如：application/json，则请求时，会先发一个预检请求（OPTIONS请求），预检请求被正确响应后，才会发起后续真正的请求。服务器端得设置：
Access-Control-Allow-Origin:* 
Access-Control-Allow-Methods:GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers:Content-Type

如果js的跨域ajax使用了自定义请求头参数，则请求时，也会预先发一个预检请求（OPTIONS请求），预检请求被正确响应后，才会发起后续真正的请求，服务器端得设置（Access-Control-Allow-Methods:GET, POST, PUT, DELETE, OPTIONS）以便支持OPTIONS请求。服务器端得设置：
Access-Control-Allow-Origin:* 
Access-Control-Allow-Methods:GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers:自定义参数名1, 自定义参数名2, 自定义参数名3...

服务器端会收到2个请求
第一个为OPTIONS的请求
第二个为真正的API请求

如果我们在web.config或global.asax中统一设置：
Access-Control-Allow-Origin:* 
Access-Control-Allow-Methods:GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers:自定义参数名1, 自定义参数名2, 自定义参数名3...

那么遇到预检请求（OPTIONS）时，也会进行API逻辑处理（等于OPTIONS和真正的请求，做了两次一模一样的API逻辑处理，只是OPTIONS请求时无法带上请求体的内容和自定义请求头参数，可能导致结果和真正请求不一致的现象），2次处理API逻辑，这明显不是我们希望的。

那如何处理带有预检请求的OPTIONS请求呢？
其实只是很简单，只要对HttpApplction的BeginRequest事件进行拦截，然后输出输出如下：
```
if(HttpContext.Request.HttpMethod.ToUpper()=="OPTIONS")
{
  HttpContext.Response.Headers.Add("Access-Control-Allow-Headers", "access_token,id"); 
  HttpContext.Response.Headers.Add("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS"); 
  HttpContext.Response.Headers.Add("Access-Control-Allow-Origin", "*"); 
  HttpContext.Response.StatusCode = 200; 
  HttpContext.Response.SubStatusCode = 200; 
  HttpContext.Response.End();
}
```