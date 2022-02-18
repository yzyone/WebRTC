# 安装 Python 2.7 在Ubuntu 20.04 LTS #

**打开终端Terminal**

在ubuntu desktop 也可以采用快捷键Ctrl+Alt+T进入Terminal；

**增加Universe repo以便可以安装python2**

![](./python2/0558d8c387da423eb53027c5298ab6ee.png)

	sudo apt-add-repository universe
	sudo apt update 

**Install Python2.7 on Ubuntu 20.04 LTS**


	sudo apt install python2-minimal

**检查Python2 version**


	python -V

**系统中可以用的python版本**

![](./python2/3ec17a7dbc574a10b98103f150d8033c.png)

    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
    update-alternatives --config python


**安装 Pip 2 on Ubuntu 20.04**

![](./python2/3a7da340c8e84012aaadf7b7ed583dae.png)

    curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
    sudo python2 get-pip.py

**卸载 (optional)**

	sudo apt remove python2-minimal

————————————————

版权声明：本文为CSDN博主「我想我是一只蜗牛」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/weixin_42030858/article/details/120509227