### nginx  location语法规则

语法规则： location [=|~|~*|^~] /uri/ {… }

首先匹配 =，其次匹配^~,其次是按文件中顺序的正则匹配，最后是交给 /通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。

| 符号                  | 含义                                                         |
| --------------------- | ------------------------------------------------------------ |
| =                     | = 开头表示精确匹配                                           |
| ^~                    | ^~开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格） |
| ~                     | ~ 开头表示区分大小写的正则匹配                               |
| ~*                    | ~* 开头表示不区分大小写的正则匹配                            |
| !~和!~*               | !~和!~*分别为区分大小写不匹配及不区分大小写不匹配的正则      |
| /                     | 用户所使用的代理（一般为浏览器）                             |
| $http_x_forwarded_for | 可以记录客户端IP，通过代理服务器来记录客户端的ip地址         |
| $http_referer         | 可以记录用户是从哪个链接访问过来的                           |

 匹配规则示例：

```html
      location = / {
          #规则A
      }

      location = /login {
          #规则B
      }

      location ^~ /static/ {
          #规则C
      }

      location ~ \.(gif|jpg|png|js|css)$ {
          #规则D
      }

      location ~* \.png$ {
          #规则E
      }

      location !~ \.xhtml$ {
          #规则F
      }

      location !~* \.xhtml$ {
          #规则G
      }

      location / {
          #规则H
      }
```

那么产生的效果如下：

1. 访问根目录/，比如http://localhost/将匹配规则A

2. 访问 http://localhost/login 将匹配规则B，http://localhost/register则匹配规则H

3. 访问 http://localhost/static/a.html 将匹配规则C

4. 访问 http://localhost/a.gif,http://localhost/b.jpg 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用，而http://localhost/static/c.png则优先匹配到规则C

5. 访问 http://localhost/a.PNG 则匹配规则E，而不会匹配规则D，因为规则E不区分大小写。

6. 访问 http://localhost/a.xhtml 不会匹配规则F和规则G，http://localhost/a.XHTML不会匹配规则G，因为不区分大小写。规则F，规则G属于排除法，符合匹配规则但是不会匹配到，所以想想看实际应用中哪里会用到。

7. 访问 http://localhost/category/id/1111 则最终匹配到规则H，因为以上规则都不匹配，这个时候应该是nginx转发请求给后端应用服务器，比如FastCGI（[PHP](http://lib.csdn.net/base/php)），tomcat（jsp），nginx作为方向代理服务器存在。