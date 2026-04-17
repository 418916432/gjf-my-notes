## Basic
### 环境变量
```
系统环境变量: 
操作系统真正存在的环境变量

文件变量:
项目中存在的变量

一般都是先读默认值，再读配置文件，再读系统环境变量，后者覆盖前者。
```
### CORS
```
Cross orign resource sharing.

当前origin浏览器所在域名: https://www.a-site.com
请求目标: https://api.b-server.com

域名不同就会发生跨域。浏览器不会直接发post/get请求，会先发一个OPTION请求到server
后端如果收到，就会返回允许不允许跨域

如果配置了允许跨域，就会返回以下内容Access-Control-Allow-Origin，没配置没有这个Access-Control-Allow-Origin

HTTP/1.1 200 OK 
Access-Control-Allow-Origin: https://www.a-site.com ← 关键 
Access-Control-Allow-Methods: POST 
Access-Control-Allow-Headers: Content-Type

```