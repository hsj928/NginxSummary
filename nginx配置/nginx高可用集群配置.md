### nginx高可用集群配置

#### 需要准备的工作

(1)  需要两台服务器192.168.106.128 和192.168.106.130
(2)  在两台服务器安装nginx。
(3)  在两台服务器安装keepalived。

#### 安装keepalived

（1）安装

```
sudo apt-get install keepalived
```

默认安装路径: /etc/keepalived
安装之后，在etc里面生成目录keepalived, 有配置文件keepalived.conf

（2）修改配置文件

```
vi /etc/keepalived/keepalived.conf
```

```
global_defs {
	notification_email {
	  acassen@firewall.loc
	  failover@firewall.loc
	  sysadmin@firewall.loc
	}
	notification_email_from Alexandre.Cassen@firewall.loc
	smtp_server 192.168.106.128
	smtp_connect_timeout 30
	router_id LVS_DEVEL	# LVS_DEVEL这字段在/etc/hosts文件中看；通过它访问到主机
}

vrrp_script chk_http_ port {
	script "/usr/local/src/nginx_check.sh"
	interval 2   # (检测脚本执行的间隔)2s
	weight 2  #权重，如果这个脚本检测为真，服务器权重+2
}

vrrp_instance VI_1 {
	state MASTER   # 备份服务器上将MASTER 改为BACKUP
	interface ens33 //网卡名称
	virtual_router_id 51 # 主、备机的virtual_router_id必须相同
	priority 100   #主、备机取不同的优先级，主机值较大，备份机值较小
	advert_int 1	#每隔1s发送一次心跳
	authentication {	# 校验方式， 类型是密码，密码1111
        auth type PASS
        auth pass 1111
    }
	virtual_ipaddress { # 虛拟ip
		192.168.106.50   # VRRP H虛拟ip地址
	}
}
```

#### 编写脚本

在路径/usr/local/src/ 下新建检测脚本 nginx_check.sh

```
#! /bin/bash
A=`ps -C nginx -no-header | wc - 1`
if [ $A -eq 0];then
	/opt/lnmp/nginx/sbin/nginx
	sleep 2
	if [`ps -C nginx --no-header| wc -1` -eq 0 ];then
		killall keepalived
	fi
fi
```

#### 两台服务器上启动keepalived

```
$ systemctl start keepalived.service		#启动keepalived
$ ps -ef I grep keepalived					#查看启动keepalived进程id，确认是否已启动
```

#### 最终测试

(1) 在浏览器地址栏输入虚拟ip地址192.168.106.50

(2) 把主服务器(192.168.106.128) nginx和keealived停止，再输入192.168.106.50

$ systemctl stop keepalived.service   #停止keepalived
