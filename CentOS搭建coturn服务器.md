# CentOS搭建coturn服务器

**1.安装需要的环境**

`yum install openssl-devel
yum install libevent2
yum install libevent-devel
yum install mysql-devel
yum install mysql-server`

这里数据库使用mysql，也可以用其他数据库。其中libevent2可能会安装失败，建议先下载下来然后传到服务器手动安装。

**2.手动安装libevent2**

官网地址：`http://www.monkey.org/~provos/libevent/`

1.下载最新的版本上传到/usr目录下并解压

`tar zxvf libevent-2.0.10-stable.tar.gz`

2.进入解压后的目录

`cd libevent-2.0.10-stable`

3.安装gcc

`yum install gcc`

4.设置安装路径

`./configure -prefix=/usr`

5.编译、安装

`make
make install`

**3.安装coturn**

1.在/usr目录下下载coturn

`git clone https://github.com/coturn/coturn`

若没有安装git，执行yum install git安装git

2.安装

`cd coturn
./configure 
make 
make install`

查看是否安装成功

`which turnserver`

若显示了安装路径则表示安装成功

3.修改配置

在/usr/local/etc/下新建turnserver.conf文件。

注意：turnserver.conf配置文件放在哪个位置不重要，他会自动寻找，而且运行turnserver的时候会显示调用的配置文件的路径。

4.签名证书

cert和pkey配置的自签名证书用Openssl命令生成:

`openssl req -x509 -newkey rsa:2048 -keyout /etc/turn_server_pkey.pem -out /etc/turn_server_cert.pem -days 99999 -nodes`

执行命令后需要填写一些信息，随意填写即可

5.设置用户名和密码

使用命令生成密码turnadmin -k -u <用户名> -r north.gov -p <密码>，执行命令后屏幕会打印加密后的密码，请记住这个密码。这里以用户名zz，密码123456为例

6.在/etc目录下新建turnuserdb.conf文件，将用户名和上一步生成的密码填写进去然后保存退出。可以多生成几个用户名和密码

`vim /etc/turnuserdb.conf`

![](E:\GitHub\WebRTC\coturn\20190506214737454.png)

5.修改turnserver.conf配置文件

正确配置

`vim /usr/local/etc/turnserver.conf`

其中listening-ip与relay-ip采用内网ip，external-ip是外网的ip。

`relay-device=eth0 
listening-ip=内网ip 
listening-port=3478 
tls-listening-port=5349 
relay-ip=内网ip
external-ip=外网ip 
relay-threads=50 
lt-cred-mech 
cert=/etc/turn_server_cert.pem 
pkey=/etc/turn_server_pkey.pem 
pidfile=”/var/run/turnserver.pid” 
min-port=49152 
max-port=65535
userdb=/etc/turnuserdb.conf
user=xc:0x36f65151a0636f98c27dc77e50836675 //turnuserdb.conf中的用户名和密码，可以有多个`

6.运行coturn

执行命令运行coturn服务

`turnserver -v -r 外网地址:3478 -a -o -c /usr/local/etc/turnserver.conf`

在浏览器输入

`<外网ip>:3478，显示`

![](E:\GitHub\WebRTC\coturn\20181213101259309.png)

表示启动成功，如果访问不了可能是服务器防火墙没有开启3478端口

7.开启防火墙端口

`#tcp和udp都要打开
firewall-cmd --permanent --add-port=3478/tcp
firewall-cmd --permanent --add-port=3478/udp
#刷新防火墙
firewall-cmd --reload
#查看当前开放的端口
firewall-cmd --list-port`

如果还是不能访问，请考虑云服务的安全组策略是否同样开启了3478的tcp和udp端口。

8.网站检测穿透效果

访问：`https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/`

配置地址、用户名和密码后Gather candidates

9.停止turnserver

`ps -ef|grep turnserver
kill -9 xxxx`

————————————————

版权声明：本文为CSDN博主「非常暴躁的程序猿」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/hello123yy/article/details/84976086