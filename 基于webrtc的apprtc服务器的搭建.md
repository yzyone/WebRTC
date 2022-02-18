# 基于webrtc的apprtc服务器的搭建 #

基于webrtc的apprtc示例发布在公网https://apprtc.webrtc.org上（需要FQ），本文在本地ubuntu14.04 32bit搭建该系统,需要搭建房间服务器，信令服务器，TURN穿透服务器。最好使用VPN搭建环境，否则会遇到网络引起的各种错误，相关资源如下:

1.房间服务器apprtc项目源码地址: https://github.com/webrtc/apprtc

a.房间服务器搭建参考链接中的步骤就可以，如果网络环境良好，搭建成功不成问题。

b.修改apprtc/out/app_engine下的constans.py,TURN_BASE_URL需要填写本机的ip地址（注意：使用localhost会有错误）。

```
TURN_BASE_URL = 'http://192.168.1.103'
TURN_URL_TEMPLATE =
'%s/turn.php?username=%s&key=%s'
CEOD_KEY = 'helloworld'
WSS_INSTANCE_HOST_KEY = 'host_port_pair'
WSS_INSTANCE_NAME_KEY = 'vm_name'
WSS_INSTANCE_ZONE_KEY = 'zone'
WSS_INSTANCES = [{
    WSS_INSTANCE_HOST_KEY: '192.168.1.103:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
}, {
    WSS_INSTANCE_HOST_KEY: '192.168.1.103:8089',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
}]
```

c.修改apprtc/out/app_engine下的apprtc.py

```
def get_wss_parameters(request):
  ws_host_port_pair = request.get('wshpp')
  ws_tls = request.get('wstls')

  if not ws_host_port_pair:
    ws_host_port_pair =
constants.WSS_HOST_PORT_PAIR
  if ws_tls and ws_tls == 'false':
    wss_url = 'ws://' + ws_host_port_pair +
'/ws'
    wss_post_url = 'http://' + ws_host_port_pair
  else:
    wss_url = 'ws://' + ws_host_port_pair +
'/ws'
    wss_post_url = 'http://' + ws_host_port_pair
  return (wss_url, wss_post_url)
```

2.信令服务器collider项目源码地址:

https://github.com/webrtc/apprtc/tree/master/src/collider

本人按照参考链接搭建没有成功，主要是在go get collidermain这一步出现错误。经过摸索按照如下步骤搭建成功:

a.安装Go语言运行环境

	sudo apt-get install golang-go

b.在Home目录下新建文件夹

	mkdir -p ~/collider_root，并在collider_root下新建src文件夹

c.设置GOPATH环境变量

	export GOPATH=~/collider_root

d.在~/collider_root/src下设置apprtc/src/collider下的三个文件夹的链接，或者直接将collider,collidermain,collidertest三个文件夹拷贝到~/collider_root/src下(本文apprtc在Home目录下，具体自己修改命令中的路径)：

```
cp ~/apprtc/src/collider/collider ~/collider_root/src
cp ~/apprtc/src/collider/collidermain ~/collider_root/src
cp ~/apprtc/src/collider/collidertest ~/collider_root/src
```

e.进入collider_root下的src：

cd ~/collider_root/src，编译安装collider：

    go get collidermain 
    go install collidermain

成功编译后会在collider_root目录下生成bin和pkg文件夹，可执行程序在bin中

f.修改main.go的代码填上自己的ip地址，var roomSrv = flag.String("room-server","http://192.168.1.103:8080/", "The origin of the room
server")

g.运行信令服务器~/collider_root/bin/collidermain -port=8089 -tls=false

3.TURN服务器:

http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html

参考文档中的TURN server安装在ubuntu 64bit机上，不实用于本文的32bit环境。32bit安装步骤如下:

a.下载资源

	wget http://rfc5766-turn-server.googlecode.com/files/turnserver-1.8.6.0-binary-linux-wheezy-ubuntu-mint-x86-32bits.tar.gz

b.解压资源

	tar -zxvf turnserver-1.8.6.0-binary-linux-wheezy-ubuntu-mint-x86-32bits.tar.gz 

c.安装服务器

    sudo apt-get update
    sudo apt-get install gdebi-core
    sudo gdebi *.deb  

d.编辑配置文件

	/etc/default/rfc5766-turn-server,TURNSERVER_ENABLED=1去掉注

e.编辑配置文件

```
/etc/turnserver.conf
listening-device=eth0
relay-device=eth0
Verbose
fingerprint
lt-cred-mech
use-auth-secret
static-auth-secret=helloword
user=helloword:0x06b2afcf07ba085b7777b481b1020391
user=helloword:helloword
stale-nonce
cert=/etc/turn_server_cert.pem
pkey=/etc/turn_server_pkey.pem
no-loopback-peers
no-multicast-peers
```

f.生成签名证书

	sudo openssl req -x509 -newkey rsa:2048 -service coturn
	start;keyout   /etc/turn_server_pkey.pem -out
	/etc/turn_server_cert.pem -days 99999 -nodes

g.启动TURN服务器

	service coturn start


标签: ubuntu, webrtc, apprtc服务端搭建, 房间服务器, 信令服务器, TURN穿透服务器