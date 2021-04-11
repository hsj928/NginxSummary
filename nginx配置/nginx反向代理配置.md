## nginx反向代理配置

### 代理

#### 正向代理

在客户端(浏览器)配置代理服务器，通过代理服务器进行互联网访问

#### 反向代理

反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后,在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。



##### 反向代理实例1

###### 1、实现效果 

（1）打开浏览器，在浏览器地址栏输入地址nginx服务器IP，跳转到 liunx 系统 tomcat 主页面中 

###### 2、实现步骤 

（1）在 liunx 系统安装 tomcat，使用默认端口 8080；tomcat 安装文件放到 liunx 系统中，解压进入 tomcat 的 bin 目录中，./startup.sh 启动 tomcat 服务器 

（2）修改配置文件

```
vi conf/nginx.conf
```

```
server {
    listen       80;
    server_name  localhost;
    #access_log  logs/host.access.log  main;

    location / {
    	root   html;
    	proxy_pass http://127.0.0.1:8080;  #添加这一句即可
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

（3）在 windows 系统中通过浏览器访问 nginx服务器，它会跳转到tomcat的主页。

##### 反向代理实例2

###### 1、实现效果

使用nginx反向代理，根据访问的路径跳转到不同端口的服务中。

访问http://ip/edu/ 直接跳转到 tomcat1

访问http://ip/vod/ 直接跳转到 tomcat2

###### 2、实现步骤

（1）准备两个 tomcat 服务器，一个 8080 端口，一个 8081 端口 ，创建文件夹和测试页面。

修改配置文件

```
vi conf/nginx.conf
```

```
server {
    listen       80;
    server_name  localhost;
    #access_log  logs/host.access.log  main;

    location / {
    	root   html;
    	index  index.html index.htm;
    }

    location ~ /edu/ {
    	proxy_pass http://127.0.0.1:8080;
    }

    location ~ /vod/ {
    	proxy_pass http://127.0.0.1:8080;
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

（3）在 windows 系统中通过浏览器根据不同路径访问 nginx服务器，它会跳转到不同的tomcat服务的测试页面。









