##### 前提条件

1. 已安装好php
2. php已运行

##### 修改配置文件

```
vi /opt/lnmp/nginx/conf/nginx.conf
location ~ \.php$ {
    root           html;
    fastcgi_index  index.php;
    #fastcgi_pass   127.0.0.1:9000;
    fastcgi_pass   unix:/opt/lnmp/php7.4/var/run/www.sock;
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include        fastcgi_params;
}

```

##### 最终配置

> 最终配置

```
server {
    listen       80;
    server_name  localhost;
    root   html;

    #access_log  logs/local.dmxy.access.log  main;

    location / {
            root   html;
            index  index.html index.htm;
        }

	#error_page  404              /404.html;

	# redirect server error pages to the static page /50x.html
	#
	#error_page   500 502 503 504  /50x.html;
	location = /50x.html {
	root   html;
	}

    location ~ \.php(.*)$ {
    	root           html;
        fastcgi_index  index.php;
        fastcgi_pass   unix:/opt/lnmp/php7.4/var/run/www.sock;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```