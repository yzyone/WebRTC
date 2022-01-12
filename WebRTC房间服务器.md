# WebRTC房间服务器 #

一.java

1、下载jdk8 http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

2、tar -zxvf jdk-8u191-linux-x64.tar.gz

3、mv jdk1.8.0_191 /usr/local/java

4、vim /etc/profile 在末尾添加如下内容


    export JAVA_HOME=/usr/local/java
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH


二.Python

1.安装 python

	yum install -y python
	python -V 2.7.5

2.安装pip

    wget https://bootstrap.pypa.io/get-pip.py
    python get-pip.py
    pip install requests

3.安装sqlite-devel

	yum install sqlite-devel -y

三.nodejs,npm

下载nodejs8.12.0版本

    wget https://nodejs.org/dist/v8.12.0/node-v8.12.0-linux-x64.tar.gz
    mv node-v8.12.0-linux-x64 /usr/local/node
    vim /etc/profile, 添加 /usr/local/node/bin ,source /etc/profile
    node -v,npm -v
    npm install -g npm ,npm升级

2.淘宝npm镜像使用方法

    npm config set registry https://registry.npm.taobao.org 持久使用
    npm config get registry 验证是否成功
    npm install -g cnpm --registry=https://registry.npm.taobao.org 通过cnpm使用

3.grunt-cli

	cnpm -g install grunt-cli

4.apprtc

    git clone https://github.com/webrtc/apprtc.git
    cnpm install
    grunt build

5.google-cloud-sdk

网址：https://cloud.google.com/sdk/docs/quickstart-linux

下载地址：

https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-218.0.0-linux-x86_64.tar.gz

    tar -zxvf google-cloud-sdk-218.0.0-linux-x86_64.tar.gz
    mv google-cloud-sdk /usr/local/google-cloud-sdk

四.

    cd /usr/local/apprtc
    /usr/local/google-cloud-sdk/bin/dev_appserver.py --host 192.168.3.30 ./out/app_engine

原文链接：https://www.kancloud.cn/lengyueguang/linux/880264