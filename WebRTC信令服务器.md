# WebRTC信令服务器 #

一.安装go语言环境

安装包下载地址为：https://golang.org/dl/。

	wget https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz

	tar -zxvf go1.11.4.linux-amd64.tar.gz

	mv go /usr/local/

	vim /etc/profile 添加如下内容

	export GOROOT=/usr/local/go #设置为go安装的路径

	export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

	go version #查看go环境是否配置正确

二.

1.

	mkdir -p ~/collider/src
	ln -s ~/apprtc/src/collider/{collider,collidermain,collidertest} ~/collider/src/

设置go编译环境：

	vim /etc/profile 添加如下内容
	export GOPATH=$HOME/collider

2.编辑$GOPATH/collidermain/main.go,修改房间服务器为我们前面的房间服务器：

    //var roomSrv = flag.String("room-server", "https://appr.tc", "The origin of the room server")
    var roomSrv = flag.String("room-server", "http://192.168.3.21:8080", "The origin of the room server")

3.cd ~/collider/src

    go get collidermain
    go install collidermain

若go get collidermain命令运行失败,那么则用下面这个麻烦的方法:

    cd $GOPATH/src
    wget http://www.golangtc.com/static/download/packages/golang.org.x.net.tar.gz
    tar xvf golang.org.x.net.tar.gz
    go install golang.org/x/net/websocket/

运行

	$GOPATH/bin/collidermain -port=8089 -tls=false

测试

	go test collider

原文链接：https://www.kancloud.cn/lengyueguang/linux/880343