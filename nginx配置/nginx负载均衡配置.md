### nginx负载均衡配置

#### 负载均衡是什么

负载均衡，英文名称为Load Balance，其含义就是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如FTP服务器、Web服务器、企业核心应用服务器和其它主要任务服务器等，从而协同完成工作任务。

#### 负载均衡实例

##### 1、实现效果

浏览器地址栏输入地址 http://ip/edu/a.html，负载均衡效果，平均 到8080 和 8081 端口中。

##### 2、实现步骤

（1）准备两台 tomcat 服务器，一台 8080，一台 8081 

（2）在两台 tomcat 里面 webapps 目录中，创建名称是 edu 文件夹，在 edu 文件夹中创建 页面 a.html，用于测试 

（3）在 nginx 的配置文件中进行负载均衡的配置

```
vi conf/nginx.conf
```

```
upstream MyServer{
	server 192.168.106.128:8080;
	server 192.168.106.128:8081;
}
server {
    listen       80;
    server_name  localhost;
    #access_log  logs/host.access.log  main;

    location / {
    	proxy_pass http://MyServer;
    	root   html;
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

​	(4) 在浏览器中输入 http://192.168.106.128/edu/a.html

#### 负载均衡算法

##### 1.round robin（默认）

轮询方式，依次将请求分配到各个后台服务器中，默认的负载均衡方式。 
适用于后台机器性能一致的情况。 
挂掉的机器可以自动从服务列表中剔除。

##### 2.weight

根据权重来分发请求到不同的机器中，指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。  

例如： 

```
upstream MyServer {    
	server server1 weight=10;    
	server server2 weight=10;    
}  
```

##### 3. IP_hash

根据请求者ip的hash值将请求发送到后台服务器中，可以保证来自同一ip的请求被打到固定的机器上，可以解决session问题。

例如：

```
upstream MyServer {    
	ip_hash;    
	server server1;    
	server server2;    
}   
```

##### 4.url_hash（第三方）

根据请求的url的hash值将请求分到不同的机器中，当后台服务器为缓存的时候效率高。

例如：

在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法 

```
upstream MyServer {    
	server server1;    
	server server2;    
	hash $request_uri;    
	hash_method crc32;    
}  
```

tips: 

```
upstream bakend{#定义负载均衡设备的Ip及设备状态    
	ip_hash;    
	server 127.0.0.1:9090 down;    
	server 127.0.0.1:8080 weight=2;    
	server 127.0.0.1:6060;    
	server 127.0.0.1:7070 backup;    
} 
```

在需要使用负载均衡的server中增加 

```
proxy_pass http://bakend/; 
```

每个设备的状态设置为: 

1. down ：表示当前的server暂时不参与负载 。
2. weight ：默认为1，weight越大，负载的权重就越大。
3. max_fails ：允许请求失败的次数默认为1，当超过最大次数时返回proxy_next_upstream 模块定义的错误 。
4. fail_timeout:max_fails次失败后，暂停的时间。 
5. backup：其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻；nginx支持同时设置多组的负载均衡，用来给不用的server来使用；
   client_body_in_file_only 设置为On 可以讲client post过来的数据记录到文件中用来做debug ；
   client_body_temp_path 设置记录文件的目录 可以设置最多3层目录 ；
   location 对URL进行匹配，可以进行重定向或者进行新的代理负载均衡。

##### 5. fair（第三方）

根据后台响应时间来分发请求，响应时间短的分发的请求多。

例如：

```
upstream MyServer {    
	server server1;    
	server server2;    
	fair;    
}  
```

##### 6.最少连接

最少连接方式可以更公平的将负载分配到多个机器上面。使用最少连接，nginx不会将请求分发到繁忙的机器上面，而且将新的请求分发到较清闲的机器上面。

```
upstream MyServer {
  least_conn;
  server server1 weight=1;
  server server2 weight=1;
}
```

