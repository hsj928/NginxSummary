#### 启用SSL

##### 创建私钥和自签名的证书

创建一个私钥和自签名的证书,这对大多数不需要 CSR的安装来说足够了:

```
# mkdir .ssl
# cd .ssl
# openssl req -new -x509 -nodes -newkey rsa:4096 -keyout server.key -out server.crt -days 1095
# chmod 400 server.key
# chmod 444 server.crt
```

##### 修改配置文件

    server {
    	listen       443 ssl;
    	server_name  localhost;
        ssl_certificate      /opt/lnmp/nginx/.ssl/server.crt;
        ssl_certificate_key  /opt/lnmp/nginx/.ssl/server.key;
    
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
    
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
    
        location / {
            root   html;
            index  index.html index.htm;
        }
    }

```
重启服务：systemctl restart nginx
```


