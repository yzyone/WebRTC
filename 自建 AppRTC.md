# 自建 AppRTC #

AppRTC `https://github.com/webrtc/apprtc` 是 webrtc 的一个 demo。

自建 AppRTC 可以苦其心志劳其筋骨饿其体肤，更重要的是能学会 webrtc 服务器的搭建流程……

AppRTC 的组成部分是这样的：

1、AppRTC - 房间服务器。Github `https://github.com/webrtc/apprtc`

2、Collider - 信令服务器。上面 Github 工程里自带，在 src/collider 下

3、coTurn - 打洞(内网穿透)服务器。Google Code `https://code.google.com/p/coturn/`

4、还需要自己实现一个 coTurn 连接信息（主要是用户名、密码的配置）获取接口，通常叫做 TURN REST API。

## 零. 准备工作 ##

0、为避免其他操作系统的各种尿性问题，这里操作系统推荐使用 Ubuntu 14.04。

1、AppRTC 依赖 grunt 和 Google AppEngine SDK for Python 离线版本。

GAE SDK 安装很简单：下载、解压、添加到PATH环境变量即可完成。

grunt，是 Node.js 下的一套任务执行系统，经过 Gruntfile.js 的配置，可以做很多事情。首先安装 Node.js。使用 nvm 可以很方便的为自己的 Linux 账户安装并设置好 Node.js。（而后，你可以选择安装 cnpm，这样就可以使用国内的缓存节点，npm install 命令会快许多，如果你只用这一次 grunt，那么不装这个也是可以的。）接下来只需要执行一个 npm install -g grunt-cli 即可安装好 grunt。

2、Collider 依赖 golang。尿性的问题来了：墙……安装 golang 和日后使用 golang 所需的包，几乎都要FQ。所以这里解决掉墙这个问题后，再使用 gvm 安装 golang(当前版本 go1.4.2) 即可完成我们的准备工作。

## 一. coTurn，打洞服务器，安装需 sudo ##

0、这货是一个基本依赖，而且不需要改代码，只要安装，然后改配置文件，再启动服务、开好端口就行了。

1、下载，去 这里`http://turnserver.open-sys.org/downloads/` ，下载最新版，turnserver-*-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz

2、创建个目录，然后，tar xzf <下载的文件名> -c <目录名>。

3、进入目录，cat INSTALL，查看安装须知，并根据指引安装所需软件，而后安装该目录下的deb包。

4、安装完成后，修改配置文件 /etc/turnserver.conf 成如下内容：

```
# 不多讲
listening-device=eth0
listening-port=3478
relay-device=eth0
# 记得开防火墙
min-port=59000
max-port=65000
# 更繁杂的输出日志
Verbose
# WebRTC 的消息里会用到
fingerprint
# WebRTC 认证需要
lt-cred-mech
# REST API 认证需要
use-auth-secret
# REST API 加密所需的 KEY
# 这里我们使用“静态”的 KEY，Google 自己也用的这个
static-auth-secret=4080218913
# 用户登录域
realm=<填写你自己的域名>
# 可为 TURN 服务提供更安全的访问
stale-nonce
# SSL 需要用到的, 生成命令:
# sudo openssl req -x509 -newkey rsa:2048 -keyout /etc/turn_server_pkey.pem -out /etc/turn_server_cert.pem -days 99999 -nodes
cert=/etc/turn_server_cert.pem
pkey=/etc/turn_server_pkey.pem
# 屏蔽 loopback, multicast IP地址的 relay
no-loopback-peers
no-multicast-peers
# 启用 Mobility ICE 支持(具体干啥的我也不晓得)
mobility
# 禁用本地 telnet cli 管理接口
no-cli
```

配置文件保存成功后，使用命令 service coturn start ，启动 coTurn 服务。

如果服务器在内网，还需要做好端口映射：3478,3479:tcp&udp，外加配置文件里的 59000~65000:tcp&udp。

## 二. coturn 连接信息接口(TURN REST API) ##

0、由于 coTurn 没有自带该接口，所以需要我们码农自行实现。关于该接口的实现方式，详细的内容请参考 coTurn原始文档。我这里只做简要说明：

```
* TURN 服务器自身没有必要实现该接口，所以需要服务提供方自行实现及部署。
* 此 TURN REST API 有标准的参考文档，包括了请求参数以及返回内容，都有明确规定。码农们需要按照该文档内描述的内容来实现该接口。
* 文档里提到的内容有：
* ① 请求信息，方式 GET：
*     需要包含：service(“turn”)、username、key
*     例如：/turn?username=392938130&key=4080218913
* ② 响应信息，内容格式 JSON Dictionary：
*     需要包涵：username、password、ttl(可选, 单位秒. 推荐86400)、uris(数组)
*     例如：

{"username": "1433065916:392938130", "password": "***base64 encoded string***", "ttl":86400, "uris": ["turn:104.155.31.139:3478?transport=udp", "turn:104.155.31.139:3478?transport=tcp", "turn:104.155.31.139:3479?transport=udp", "turn:104.155.31.139:3479?transport=tcp"]}
```

基本形式就是这样。需要提及的有：

1、响应的 username 字段，需要以【timestamp + ":" + username】的形式输出。注意：这里 timestamp 为密码失效的 UNIX 时间戳，即【当前 timestamp + ttl 时长】；而 username 则是请求时提交的 username，原封不动；中间使用半角冒号做字符串连接（默认设置下是这样，你也可以自己改，不过多蛋疼……）。为便于后文叙述，此处响应内容里的 username 字段，我们称之为 turn-username。

2、响应的 password 字段，需要以 HMAC-SHA1 算法计算得出，公式为：【base64_encode( hmac( key, turn-username ) )】。此处的 key，为 coTurn 服务器配置中的 “static-auth-secret”值。以 key 作为 hmac 算法的密钥，turn-username 为被计算的内容，得出的 hmac 摘要后，经 base64 编码得到最终密码。

3、就是这些。uris 一般根据 coTurn 服务器的假设规模来从后台配置好，而后再直接输出到前台。我用 Node.js 简单写了个接口代码，有需要可以参考：

```
var express = require('express');
var crypto = require('crypto');
var app = express();

var hmac = function(key, content){
    var method = crypto.createHmac('sha1', key);
    method.setEncoding('base64');
    method.write(content);
    method.end();
    return method.read();
};

app.get('/turn', function(req, resp) {

var query = req.query;
var key = '4080218913'; // 这里的 key 是事先设置好的, 我们把他当成一个常亮来看, 所以就不从HTTP请求参数里读取了

if (!query['username']) {
    return resp.send({'error':'AppError', 'message':'Must provide username.'});
} else {
    var time_to_live = 600;
    var timestamp = Math.floor(Date.now() / 1000) + time_to_live;
    var turn_username = timestamp + ':' + query['username'];
    var password = hmac(key, turn_username);

    return resp.send({
        username:turn_username,
        password:password,
        ttl:time_to_live,
        "uris": [
            "turn:turn.server.ip.address:3478?transport=udp",
            "turn:turn.server.ip.address:3478?transport=tcp",
            "turn:turn.server.ip.address:3479?transport=udp",
            "turn:turn.server.ip.address:3479?transport=tcp"
            ]
    });
}

});

app.listen('3033', function(){
    console.log('server started');
});
```

代码里没处理跨域的问题，所以虽说上面代码监听了 3033 端口，但我用 nginx 做了一个反代，以便能够使用同一个域名。这样也就不会出现跨域的问题了。

## 三. Collider 信令服务器 ##

0、现在可以摆好FQ的pose了，如果不知道怎么翻说明你还没有准备好……

1、首先，可能是因为WALL的原因亦或是我FQ的姿势不对，我在 get Collider 的依赖库的时候遇到了点麻烦，不过改了源代码就好了：

```
# $PWD = apprtc/src/collider/collider/
进入上面的这个目录
# $FILES = [ 'collider.go', 'collider_test.go' ]
把上面俩文件里的这行:
"code.google.com/p/go.net/websocket"
替换成:
"golang.org/x/net/websocket"
记得保存
```

2、然后就可以参照着 src/collider/README.md 里面的步骤执行了：


......

	# $PWD = apprtc/src/collider/


将 以下3个目录, 软链接到 $GOPATH/src/

注意，应当先检查自己的 golang 是否安装好，并检查 $GOPATH 是否不为空且路径 $GOPATH/src/ 已存在.

假设：当前现在就在 apprtc/src/collider/ 下

命令：

	ln -s $PWD/{collider,collidermain,collidertest} $GOPATH/src/

结果：会把 3 个目录软链接到 $GOPATH/src/ 下

软链接创建好后，执行：

命令：

	go get collidermain && go install collidermain

3、以上命令成功执行后，路径 $GOPATH/bin/collidermain 将会是一个可执行文件。

可以追加 --help 参数查看所有可用的命令行参数。这里我们使用这样的配置：

```
# $GOPATH/bin/collidermain -port=8089 -room-server=‘http://apprtc.domain.com:8083' -tls=false

# 到这里我们还没有接触过 AppRTC 的搭建，所以暂时我们并不知道 room-server 参数的值是什么。
# 这条命令应该是等下搞好了 AppRTC 后，再回来执行的。

-port = 表示 collider 监听的端口。我们这里用 8089。如果有防火墙，记得打开，如果在内网，记得在路由器上做映射。
-room-server = 表示 AppRTC 可访问的 URL。如果使用了非标准端口（80、443），则务必写好端口号。
-tls = false 表示关闭 TLS 加密。因为这个需要配置证书（"/cert/cert.pem", "/cert/key.pem"），所以以自己研究为目的的，关了是最方便的……
```

4、这里的 8089 是服务端口，如果你在内网里并需要对外提供服务，需要将 8089:tcp 映射出外网。

## 四. AppRTC 房间服务器 ##

0、这是最后一步，只需要改几处配置就可以打开运行了。这里假设你所在的路径是 apprtc 项目的根目录。

1、由于前面我们的 Collider 使用了 tls = false 参数，且为了访问方便，这里需要做以下修改：

```
# $FILE = apprtc/src/app_engine/apprtc.py

搜索 "wss:" 和 "https:" （注意冒号）
可以在方法 get_wss_parameters 里搜索到，这里需要把 wss: 替换成 ws:、把 https: 替换成 http:，保存退出，就可以了。
```

2、编辑配置文件，文件位于 apprtc/src/app_engine/constants.py。

```
① 搜索 TURN_BASE_URL
　 将等号后面的字符串替换为 apprtc 可以访问到的地址，如：'http://apprtc.domain.com:8083'

② 搜索 WSS_INSTANCES
　 可以看到，这里被配置为了一个数组，不过我们只有单台服务器。所以先删掉数组的其他元素，只保留一个。
　 在保留下来的元素中，我们只修改 WSS_INSTANCE_HOST_KEY 对应的值即可。
　 将其改为上面 Collider 服务器的可访问地址。比如：apprtc.domain.com:8089。不需要协议，没有 URI。
```

改完以上两处，可以保存退出了

3、如果你使用了非标准端口，或者使用路由器上的端口映射为广域网提供测试和服务，还需要修改WEB前端的一些代码。否则会遇到浏览器的跨域访问问题（主要是 pushState 会产生一个错误），这么改：

```
# $FILE = apprtc/src/web_app/js/appcontroller.js

这个文件里有这样的两个函数：
AppController.prototype.pushCallNavigation_
AppController.prototype.displaySharingInfo_

分别在这两行函数内部的开头，加上如下代码：
roomLink = roomLink.replace(/apprtc\.domain\.com\//, 'apprtc.domain.com:8083/');
```

保存退出

4、现在再次确认当前目录是 apprtc 项目的根目录，而后执行：

```
$ grunt build; dev_appserver.py --host=0.0.0.0 ./out/app_engine

# 因为是为公网、或者局域网其他人提供服务，所以这里 host=0.0.0.0，以避免只监听 127.0.0.1 的状况。
```

5、需要注意的是，dev_appserver 默认监听了 8080 端口。但配置里到处都是 8083，因为还差了最后一步……

## 五. nginx，用来统一 HTTP 对外服务端口，以及最主要的反代 3033 TURN REST API 服务。 ##

0、先装好nginx：`apt-get install nginx -y`

1、默认配置文件在 /etc/nginx/sites-enabled/default 。打开后，新增：

```
server {
  listen 8083;
  server_name apprtc.domain.com;
  location / {
    proxy_pass http://ip.address.of.apprtc:8080;
    proxy_set_header Host $host;
  }
  location /turn {
    proxy_pass http://ip.address.of.turn-rest-api:3033;
    proxy_set_header Host $host;
  }
}
```

保存后退出

2、使用 service nginx reload 重载配置文件。

3、如果有需要，可以在路由器上做端口映射：8083:tcp。

4、此时，外部可以访问到的 AppRTC 服务的 URL 已经确定了，你可以检查并执行【三（3）】中的命令。注意如果前面运行过此命令，就不需要再次运行了。

## 六. 打开浏览器访问 http://apprtc.domain.com:8083/ 。 ##

输入房间号，并把房间URL发给你的好基友，就可以面基了……

--------------------------------------------------------------

由于我搭建整套环境是在内网虚拟机上操作的，所以文章里到处都会出现”在路由器上设置好端口映射“这样的句子。
在这里，统一做一下整理吧：

```
8083:tcp - AppRTC 的端口，提供 HTTP 服务。默认使用的是 8080:tcp，我把它改了。
8089:tcp - Collider 服务的端口，提供 WebSocket 服务。
3478、3479:tcp&udp - coTurn 的监听端口。
59000~65000:tcp&udp - coTurn 的服务端口。
```

TURN REST API 的端口（3033）由于被 nginx 反向代理了，所以并不需要再单独给它做端口映射。

就这样吧，EOF

原文链接：https://www.cnblogs.com/hujihon/p/4991137.html