### ubuntu 18.04安装nginx + php7.4 + mysql5.7



#### nginx简介

```
Nginx ("engine x")是一个高性能的HTTP和反向代理服务器，特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好；
Nginx专为性能优化而开发，性能是其最重要的考量，实现上非常注重效率，能经受高负载的考验，有报告表明能支持高达50000个并发连接数。
```

#### ubuntu更换为国内源

##### 备份文件

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```



##### 更换为中科大源



```
sudo vi /etc/apt/sources.list

deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
```



#### nginx



##### 1.安装Nginx依赖包以及常用工具

###### 安装openssl：

​		

```
sudo apt-get install openssl libssl-dev
```



###### 安装pcre：

​		

```
sudo apt-get install libpcre3 libpcre3-dev
```



###### 安装zlib：

​		

```
sudo apt-get install zlib1g-dev
```



###### 常用工具：

​    

```
sudo apt-get install git
	sudo apt-get install make cmake
	sudo apt-get install build-essential
	sudo apt-get install zip unzip libbz2-dev


```



##### 2.下载Nginx源码

​	

```
wget http://nginx.org/download/nginx-1.19.9.tar.gz
```



##### 3.解压

```
tar -xzvf nginx-1.19.9.tar.gz
```



##### 4.创建用户

```
sudo useradd -M -s /sbin/nologin nginx
```



##### 5.configure & install

```
./configure --prefix=/opt/lnmp/nginx \
	--user=nginx --group=nginx \
	--with-http_stub_status_module \
	--with-http_ssl_module \
	--with-http_random_index_module \
	--with-http_sub_module \
	--with-http_gzip_static_module \
	--with-pcre   
    && make  && make install
```



也可简易参数：

```
./configure --prefix=/opt/lnmp/nginx \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-pcre 
&& make -j12 && make install
```




##### 6.参数注解

- --prefix=/opt/lnmp/nginx 安装目录
- --user=nginx --group=nginx 指定进程守护者
- --with-http_stub_status_module 监控模块
- --with-http_ssl_module https
- --with-http_random_index_module 随即首页
- --with-http_sub_module 字符串替换
- --with-http_gzip_static_module css等压缩
- --with-pcre 正则匹配 nginx 配置文件中的 url

##### 7.systemctl工具启动

```
vi /lib/systemd/system/nginx.service
```



```
[Unit]
Description=nginx service
After=network.target network-online.target syslog.target
Wants=network.target network-online.target

[Service]
Type=forking

ExecStart=/opt/lnmp/nginx/sbin/nginx
ExecReload=/opt/lnmp/nginx/sbin/nginx -s reload
ExecStop=/opt/lnmp/nginx/sbin/nginx -s stop

[Install]
WantedBy=multi-user.target
```

```
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
```

##### 8.修改 nginx 安装目录权限

```
sudo setfacl -m u:nginx:rwx -R /opt/lnmp/nginx
sudo setfacl -m d:u:nginx:rwx -R /opt/lnmp/nginx
```

#### php7.4

##### 1.依赖包

```
wget http://archive.ubuntu.com/ubuntu/pool/main/libp/libpng/libpng_1.2.54.orig.tar.xz
tar xf libpng_1.2.54.orig.tar.xz
cd libpng-1.2.54/
./autogen.sh
./configure
make -j12
sudo make install

sudo apt-get install autoconf automake libtool
wget https://github.com/kkos/oniguruma/archive/v6.9.4.tar.gz -O oniguruma-6.9.4.tar.gz
tar xf oniguruma-6.9.4.tar.gz
cd oniguruma-6.9.4/
./autogen.sh
./configure
make -j12
sudo make install

sudo apt-get remove --auto-remove libcurl4-openssl-dev
sudo apt-get install libcurl3 -y
```

#####  2. configure & install

```
./configure \
--prefix=/opt/lnmp/php7.4 \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--with-config-file-path=/opt/lnmp/php7.4/etc \
--disable-ipv6 \
--with-libxml-dir \
--enable-ftp \
--with-openssl \
--with-zlib \
--with-curl \
--enable-gd \
--with-jpeg \
--enable-mbstring \
--enable-sockets \
--with-xmlrpc \
--with-iconv-dir=/usr/local/libiconv \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-mysqlnd \
--enable-mysqlnd-compression-support \
--enable-static
```

##### 3.配置文件

- php.ini

```
cp php.ini-* /opt/lnmp/php7.4/etc/
cp php.ini-development /opt/lnmp/php7.4/etc/php.ini
```

- php-fpm.conf

```
cp php-fpm.conf.default php-fpm.conf
cd php-fpm.d
cp www.conf.default www.conf
```

- 修改www.conf配置文件

```
;listen = 127.0.0.1:9000
listen = /opt/lnmp/php7.4/var/run/www.sock
```

##### 4. php-fpm 运行有以下两种模式

- socket

```
listen = /opt/lnmp/php7.4/var/run/www.sock
```

- tcl

```
listen = 127.0.0.1:9000
```

##### 5. systemctl工具启动

- 修改php-fpm.conf配置文件

```
; Pid file
; Note: the default prefix is /usr/local/php/var
; Default Value: none
pid = /opt/lnmp/php7.4/var/run/php-fpm.pid
```

- systemctl

```
vi /lib/systemd/system/phpfpm.service

[Unit]
Description=phpfpm service
After=network.target syslog.target

[Service]
Type=forking
PIDFile=/opt/lnmp/php7.4/var/run/php-fpm.pid
ExecStart=/opt/lnmp/php7.4/sbin/php-fpm
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
```

- systemctl start phpfpm
- systemctl stop phpfpm
- systemctl restart phpfpm

##### 6. 授权 php 程序

```
sudo setfacl -m u:nginx:rwx -R /opt/lnmp/php7.4
sudo setfacl -m d:u:nginx:rwx -R /opt/lnmp/php7.4
```



























































