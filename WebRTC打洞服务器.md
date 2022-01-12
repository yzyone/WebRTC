# WebRTC打洞服务器 #

1、安装依赖库

	yum install -y make gcc cc gcc-c++ wget

	yum install -y openssl-devel libevent libevent-devel mysql-devel mysql-server

2、找到合适的 turn stun 版本并安装

	wget http://turnserver.open-sys.org/downloads/v4.5.0.6/turnserver-4.5.0.6-CentOS7.2-x86_64.tar.gz

	tar -zxvf turnserver-4.5.0.6-CentOS7.2-x86_64.tar.gz
	cd turnserver-4.5.0.6

如果里面有install.sh文件，直接执行./install.sh

	./configure
	make
	make install

3.修改配置文件

监听端口可以不设置会默认的使用3478

```
listening-port=3478
#listening-ip,注意必须是你的内网IP地址如：
listening-ip=xx.xx.xx.xx
#relay-ip可以不设置，默认会使用你的外网ip地址作为转发包的中继地址,建议不设置，使用默认就可以：
relay-ip=xx.xx.xx.xx
#external-ip，注意必须使用你的外网IP地址如：
external-ip=xx.xx.xx.xx
#设置用户名及密码，这个是作为TURN服务器使用必须设置的,可以设置多个
user=user:password 或者使用ssh也是可以的
user=user:passKey
#realm，目前没发现有什么用，可设置可不设置：
realm=companyName.com.cn
#turndb数据库位置，/var/db/turndb
```

以上就是配置的主要内容，更详细的配置可以直接查看turnserver.conf，里面的注释很详细，可以设置tls，mysql，redis，mongodb等等内容这里不做详细解释了。

另外：STUN和TURN的区别，turn服务器是一个特殊的stun服务器，turn具备了stun的功能，并且具备stun不具备的中继转发功能，我们按照的服务可以不提供turn功能只作为stun使用，打开turnserver.conf中的stun-only即可。

原文链接：https://www.kancloud.cn/lengyueguang/linux/881703