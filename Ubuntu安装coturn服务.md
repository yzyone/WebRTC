# [DevOps](https://www.dazhuanlan.com/topics/node5) Ubuntu安装coturn服务 ~ 王诛魔

2020年03月15日 · 188 次阅读

**coturn project**

This project evolved from rfc5766-turn-server project (https://code.google.com/p/rfc5766-turn-server/).

这是一个穿透服务，为 webrtc 的链接做服务,按照 rfc5766-turn-server 实现。

**项目地址**：https://github.com/coturn/coturn

**安装 Wiki**: https://github.com/coturn/coturn/wiki/CoturnConfig

**依赖库 Third-party libraries**

libevent

```
$ wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz

# 需要sodu权限
$ tar xvfz libevent-2.0.21-stable.tar.gz
$ cd libevent-2.0.21-stable
$ ./configure
$ make
$ make install
```

安装完毕 `/usr/local/lib`

coturn install

**下载地址**：https://github.com/coturn/coturn/wiki/Downloads

下载

方式一：

我选择了最新的版本，4.5.0.8

```
$ wget https://coturn.net/turnserver/v4.5.0.8/turnserver-4.5.0.8.tar.gz
```

一直下载失败..所以我在本地下载了，上传到服务器 `home/wangzhumo/source`

方式二：

```
$ cd /home/wangzhumo/source
$ git clone https://github.com/coturn/coturn
```

编译

```
# 解压
$ tar -xvf turnserver-4.5.0.8.tar
$ cd turnserver-4.5.0.8
$ ./configure
$ make
$ sudo make install
```

```
# 记录安装位置
install -d /usr/local
install -d /usr/local/bin
install -d /usr/local/var/db
install -d /usr/local/man/man1
install -d /usr/local/etc
install -d /usr/local/lib
install -d /usr/local/share/examples/turnserver
install -d /usr/local/share/doc/turnserver
install -d /usr/local/share/turnserver
install -d /usr/local/include/turn

1) If your system supports automatic start-up system daemon services, 
then to enable the turnserver as a system service that is automatically
started, you have to:

    a) Create and edit /etc/turnserver.conf or 
    /usr/local/etc/turnserver.conf . 
    Use /usr/local/etc/turnserver.conf.default as an example.

    b) For user accounts settings: set up SQLite or PostgreSQL or 
    MySQL or MongoDB or Redis database for user accounts.
    Use /usr/local/share/turnserver/schema.sql as SQL database schema,
    or use /usr/local/share/turnserver/schema.userdb.redis as Redis
    database schema description and/or 
    /usr/local/share/turnserver/schema.stats.redis
    as Redis status & statistics database schema description.

    If you are using SQLite, the default database location is in 
    /var/db/turndb or in /usr/local/var/db/turndb or in /var/lib/turn/turndb.

    c) add whatever is necessary to enable start-up daemon for the 
    /usr/local/bin/turnserver.

2) If you do not want the turnserver to be a system service, 
   then you can start/stop it "manually", using the "turnserver" 
   executable with appropriate options (see the documentation).

3) To create database schema, use schema in file 
/usr/local/share/turnserver/schema.sql.

4) For additional information, run:

   $ man turnserver
   $ man turnadmin
   $ man turnutils
```

coturn 配置

turnserver.conf

```
$ sudo vim /usr/local/etc/turnserver.conf.default
```

要修改的地方只有 5 处：

    listening-port=3478
    
    external-ip=[服务器的 ip]
    
    min-port=50001
    
    max-port=65535
    
    user=wangzhumo:webrtcpwd
    
    realm=stun.wangzhumo.com

```
$ sudo mv /usr/local/etc/turnserver.conf.default /usr/local/etc/turnserver.conf
```

防火墙

1.打开云服务器的 安全组 ，加入 3478 ，50001—65535

2.本机的防火墙

coturn 开启

```
$ turnserver -o -c /usr/local/etc/turnserver.conf
```

测试

测试地址：https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/

1.输入：

STUN or TURN URI: turn:stun.wangzhumo.com

TURN username: wangzhumo

TURN password: webrtcpwd

2.Add Server

3.Gather candidates

测试结果：

```
Time    Component    Type    Foundation    Protocol    Address    Port    Priority
0.003    rtp    host    2749607730    udp    172.30.21.85    60720    126 | 30 | 255
7.794    rtp    srflx    842163049    udp    221.221.255.18    33338    100 | 30 | 255
7.825    rtp    relay    1676812102    udp    171.51.157.190    65032    2 | 30 | 255
7.825    Done
7.827
```

看到 relay 类型的就说明成功了



END
