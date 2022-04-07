# ubuntu16.04下安装配置coturn

一、安装软件包

```
apt update
apt install coturn
```

二、配置coturn

1、复制DTLS、TLS支持的证书文件：

```
cp /usr/share/coturn/examples/etc/turn_server_cert.pem /etc/turn_server_cert.pem
cp /usr/share/coturn/examples/etc/turn_server_pkey.pem /etc/turn_server_pkey.pem
```

2、编辑/etc/turnserver.conf文件：

```
listening-port=3478
tls-listening-port=5349
listening-ip=内网地址
relay-ip=内网地址
external-ip=外网地址
server-name=外网地址
realm=外网地址
lt-cred-mech
user=用户名:密码
userdb=/var/lib/turn/turndb
cert=/etc/turn_server_cert.pem
pkey=/etc/turn_server_pkey.pem
no-stdout-log
log-file=/var/tmp/turnserver.log
pidfile="/var/run/turnserver.pid"
```

3、编辑/etc/default/coturn文件：

	TURNSERVER_ENABLED=1

三、完成安装

1、重启coturn

	service coturn restart

2、端口开放

![](https://img2020.cnblogs.com/blog/1375113/202005/1375113-20200503153951618-1955275461.png)

3、测试验证

```
 turnadmin -a -u test -r 外网地址（域名） -p test
 turnutils_uclient 外网地址（域名） -u test -w test
```

测试结果

![](https://img2020.cnblogs.com/blog/1375113/202005/1375113-20200503154059140-33499484.png)

然后就可以去使用了，如下：

```
var configuration = {
              'iceServers': [{
                 'urls': 'turn:外网地址:3478',
                 'credential': "密码",
                 'username': "用户名"
              }]
};

```

结束了，谢谢！
