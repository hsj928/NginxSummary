### nginx动静分离配置

#### 什么是动静分离

通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏 览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源 设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可， 所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件， 不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一 个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码 304， 如果有修改，则直接从服务器重新下载，返回状态码 200。



#### 动静分离实例

##### 1、实现步骤

（1）在 liunx 系统中准备静态资源（如：html文件，图像文件），用于进行访问

（2）在 nginx 配置文件中进行配置

```
vi conf/nginx.conf
```

```
server {
    listen       80;
    server_name  localhost;
    #access_log  logs/host.access.log  main;

	location ~ /www/ {
    	root   /data/;
    	index  index.html index.htm;
    }

    location ~ /image/ {
    	autoindex on;
    }
    location / {
    	root   /data/;
    	index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    	root   html;
    }    
}
```

(3) 浏览器中输入http://192.168.106.128/www/index.html或者http://192.168.106.128/image/