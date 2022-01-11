# webrtc服务器环境搭建 #

(基于公网环境)

Last Modified Date: 2017/8/2

https://blog.csdn.net/gladsnow/article/details/77900578

目录

1. 搭建平台 
2. 软件安装 
3. 搭建房间服务器(Room Server) 
4. 搭建信令服务器(Collider Server) 
5. 搭建STUN\TURN服务器 
6. 配置Nginx服务器 
7. 运行测试 
8. 附录

## 1. 搭建平台 ##

- 操作系统： Ubuntu 16.04 server(64bits)
- Google webrtc的服务器Demo：详见https://github.com/webrtc/apprtc
- 域名： webrtc.olcms.com
- IP地址： 需要是公网地址

## 2. 软件安装 ##

安装JDK：

	add-apt-repository ppa:openjdk-r/ppa 
	apt-get update 
	apt-get install openjdk-8-jdk

安装nodejs相关包：

	apt-get install nodejs
	apt-get install npm  
	apt-get install nodejs-legacy 
	npm -g install grunt-cli

安装Python和Python-webtest：

	apt-get install python 
	apt-get install python-webtest

注： 若已安装过上述软件，可忽略；如上述未提及的软件需要安装，请自行安装。

## 3. 搭建房间服务器(Room Server) ##

下载apprtc源码（操作所在目录/root/）

    git clone  https://github.com/webrtc/apprtc.git   
    cd apprtc
    npm install

若npm install报错，请自行解决。

修改文件 

1.修改/root/apprtc/src/app_engine/constants.py

```
TURN_BASE_URL = 'https://webrtc.olcms.com'; #本机域名webrtc.olcms.com 
TURN_URL_TEMPLATE = '%s/turn.php?username=%s&key=%s'; #如果turn.php未实现，可使用默认配置
CEOD_KEY = 'inesadt'   #此处后面turn配置的用户名保持一致

ICE_SERVER_BASE_URL = 'https://webrtc.olcms.com';
ICE_SERVER_URL_TEMPLATE = '%s/iceconfig.php?key=%s'; #如果iceconfig.php未实现，可用默认配置，但是Android Apk会有问题

WSS_INSTANCE_HOST_KEY = 'webrtc.olcms.com:8089'  #信令服务器端口号8089  
WSS_INSTANCE_NAME_KEY = 'vm_name'
WSS_INSTANCE_ZONE_KEY = 'zone'
WSS_INSTANCES = [{
        WSS_INSTANCE_HOST_KEY: 'webrtc.olcms.com:8089',
        WSS_INSTANCE_NAME_KEY: 'wsserver-std',
        WSS_INSTANCE_ZONE_KEY: 'us-central1-a'  
        }, {  
        WSS_INSTANCE_HOST_KEY: 'webrtc.olcms.com:8089',
        WSS_INSTANCE_NAME_KEY: 'wsserver-std-2', 
        WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
        }]
```

编译（在apprtc目录下进行）

	grunt build

编译完成之后，会生成out目录，房间服务器编译完成。

注（编译成功可忽略）：此处编译需要FQ，若编译时无法FQ，可下载手动下载https://api.callstats.io/static/callstats.min.js，并把文件callstats.min.js放到目录/root/apprtc/out/app_engine/third_party/callstats/下。然后修改/root/apprtc/build/build_app_engine_package.py文件：

```
    # Download callstats.
    ......
    ......
    response = requests.get(urls[fileName])
    #if response.status_code == 200:    #把此处注释掉
    print 'Downloading %s to %s...' % (urls[fileName], path)
    with open(path + fileName, 'w') as to_file:
       to_file.write(response.text)
    #else:  #把此处注释掉
    #  raise NameError('Could not download: ' + filename + ' Error:' + \ #把此处注释掉
    #str(response.status_code))  #把此处注释掉 
```

然后继续进行编译即可。

安装和配置google app engine

1.下载google app engine 

需FQ，下载地址https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.50.zip)，或者通过其他地方下载。

2.配置google app engine 路径 

解压google_appengine_1.9.50.zip

	unzip google_appengine_1.9.50.zip  

编辑/etc/profile文件，在文件最后添加语句：

	export PATH="$PATH:/root/google_appengine/"

（当前安装目录是/root/google_appengine,请根据自己的安装目录进行配置） 

保存profile文件，进行以下操作生效

	source /etc/profile

运行房间服务器（room server) 

在目录/root/google_appengine目录下找到dev_appserver.py脚本,执行以下语句

	./dev_appserver.py --host=webrtc.olcms.com /root/apprtc/out/app_engine

若想后台运行，则执行

	nohup ./dev_appserver.py --host=webrtc.olcms.com /root/apprtc/out/app_engine &

在浏览器中访问房间服务器

	https://webrtc.olcms.com

## 4. 搭建信令服务器(Collider Server) ##

安装go语言编译器

	apt-get install golang-go

复制collider源代码 

（此源码在房间服务器源码目录下/root/apprtc/src/collider/) 
在/root目录下新建文件夹

	mkdir -p goWorkspace/src

配置编译环境，此配置是暂时有效的

	export GOPATH=/root/goWorkspace/

把/root/apprtc/src/collider/目录下的三个目录（collider、collidermain、collidertest）复制到/root/goWorkspace/src/目录下

	cp -rf /root/apprtc/src/collider/* /root/goWorkspace/src

修改代码 

1.编辑文件/root/goWorkspace/src/collidermain/main.go，修改房间服务器的地址

	var roomSrv = flag.String("room-server", "https://webrtc.olcms.com", "The origin of the room server")

2.编辑文件/root/goWorkspace/src/collider/collider.go，修改下面这句：

	e = server.ListenAndServeTLS("/cert/cert.pem", "/cert/key.pem")

修改为

	e = server.ListenAndServeTLS("/etc/letsencrypt/live/webrtc.olcms.com/fullchain.pem", "/etc/letsencrypt/live/webrtc.olcms.com/privkey.pem")

其中fullchain.pem和privkey.pem为SSL证书，具体证书路径根据实际路径进行配置，SSL证书生成见后续Nginx配置。

编译信令服务器 

进入目录/root/goWorkspace/src/,此处编译需要FQ。

    go get collidermain
    go install collidermain


编译成功后，在/root/goWorkspace/下会生成bin和pkg目录。 

若此处编译时无法FQ，可手动下载需要的文件。在/root/goWorkspace/src/目录下，

    mkdir -p golang.org/x 
    cd golang.org/x/
    git clone https://github.com/golang/net

然后再进行编译即可。

运行信令服务器 

进入/root/goWorkspace/bin/目录，运行信令服务器

	./collidermain -port=8089 -tls=true

若想后台运行，则执行

	nohup ./collidermain -port=8089 -tls=true &

## 5. 搭建STUN\TURN服务器 ##

安装coturn

	apt-get install coturn

进行相关配置 

编辑文件/etc/default/coturn,把TURNSERVER_ENABLED=1的注释去掉。

编辑文件/etc/turnserver.conf,把以下内容加入到文件最后（或者在文件中找到相应的选项，进行配置）

```
    listening-device=eth0  #此处eth0是电脑网卡名称
    listening-port=3478     #turn服务器的端口号
    relay-device=eth0   #此处eth0是电脑网卡名称
    min-port=49152
    max-port=65535
    Verbose
    fingerprint
    lt-cred-mech
    use-auth-secret
    static-auth-secret=inesadt  #此处要和房间服务器配置时constants.py文件中的CODE_KEY保持一致。
    user=inesadt:0x7e3a2ed35d3cf7f19e2f8b015a186f54
    user=inesadt:inesadt
    stale-nonce
    cert=/usr/local/etc/turn_server_cert.pem
    pkey=/usr/local/etc/turn_server_pkey.pem
    no-loopback-peers
    no-multicast-peers
    mobility
    no-cli
```

上述文件中 0x7e3a2ed35d3cf7f19e2f8b015a186f54的生成方法：
 
	turnadmin -k -u inesadt -r north.gov -p inesadt 

- -k 表示生成一个long-term credential key 
- -u 表示用户名 
- -p 表示密码 
- -r 表示Realm域，（这个值的设置可能会有影响）。

coturn的证书生成（即配置文件中cert和pkey)

	sudo openssl req -x509 -newkey rsa:2048 -keyout /usr/local/etc/turn_server_pkey.pem –out /usr/local/etc/turn_server_cert.pem -days 99999 –nodes

启动coturn服务器
	
	service coturn start

## 6. 配置Nginx服务器 ##

安装Nginx

	apt-get install nginx

配置Nginx 

编辑配置文件/etc/nginx/sites-available/default

```
    server {
            listen 80 default_server;
            listen [::]:80 default_server;

            # SSL configuration
            #
            # listen 443 ssl default_server;
            # listen [::]:443 ssl default_server;
            #
            # Note: You should disable gzip for SSL traffic.
            # See: https://bugs.debian.org/773332
            #
            # Read up on ssl_ciphers to ensure a secure configuration.
            # See: https://bugs.debian.org/765782
            #
            # Self signed certs generated by the ssl-cert package
            # Don't use them in a production server!
            #
            # include snippets/snakeoil.conf;

            root /var/www/html;

            # Add index.php to the list if you are using PHP
            index index.html index.htm index.nginx-debian.html;

            server_name webrtc.olcms.com; #添加域名，如不添加，生成SSL证书时可能会有问题

            location / {
                    # First attempt to serve request as file, then
                    # as directory, then fall back to displaying a 404.
                 try_files $uri $uri/ =404;
            }
            .....
            ....
        }
```

生成SSL证书

1.使用let’s encrypt颁发的免费SSL证书，安装软件

    sudo apt-get update  
    apt-get install software-properties-common  
    add-apt-repository ppa:certbot/certbot  
    apt-get update 
    apt-get install python-certbot-nginx

2.生成SSL证书

    certbot --nginx certonly

SSL证书生成的目录/etc/letsencrypt/live/webrtc.olcms.com/,下面会有四个证书（cert.pem,chain.pem,fullchain.pem,privkey.pem）

PS：详见https://certbot.eff.org/#ubuntuxenial-nginx

安装php和php-fpm

    apt-get install php  
    apt-get install php7.0-fpm

编辑配置文件/etc/nginx/sites-available/default

```
upstream roomserver {
                server 114.215.131.216:8080;
        }
        server {
                listen 80 ;
                server_name webrtc.olcms.com;
                return  301 https://$server_name$request_uri;
        }
        server {
                # listen 80 default_server;
                # listen [::]:80 default_server;

                # SSL configuration
                #
                # listen 443 ssl default_server;
                # listen [::]:443 ssl default_server;
                listen 443;
                #
                # Note: You should disable gzip for SSL traffic.
                # See: https://bugs.debian.org/773332
                #
                # Read up on ssl_ciphers to ensure a secure configuration.
                # See: https://bugs.debian.org/765782
                #
                # Self signed certs generated by the ssl-cert package
                # Don't use them in a production server!
                #
                # include snippets/snakeoil.conf;

                root /var/www/html;

                # Add index.php to the list if you are using PHP
                index index.html index.htm index.nginx-debian.html index.php; #此处添加index.php

                server_name webrtc.olcms.com;

                # location / {
                        # First attempt to serve request as file, then
                        # as directory, then fall back to displaying a 404.
                        # try_files $uri $uri/ =404;
                # }

               # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
                #
                location ~ \.php$ {
                        include snippets/fastcgi-php.conf;
                        # With php7.0-cgi alone:
                        #fastcgi_pass 127.0.0.1:9000;
                        # With php7.0-fpm:
                        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                }
                location / {
                       proxy_pass http://roomserver$request_uri;
                       proxy_set_header Host $host;
                }

                # deny access to .htaccess files, if Apache's document root
                # concurs with nginx's one
                #
                #location ~ /\.ht {
                #       deny all;
                #}
                ssl on;
                ssl_certificate /etc/letsencrypt/live/webrtc.olcms.com/fullchain.pem;
                ssl_certificate_key /etc/letsencrypt/live/webrtc.olcms.com/privkey.pem;
        }
```

编写turn.php文件和iceconfig.php文件，并把文件放到目录/var/www/html/目录下
turn.php文件内容

```
 <?php  
            $request_username = $_GET["username"];  
            if(empty($request_username)) {  
                echo "username == null";  
                exit;  
            }  
            $request_key = $_GET["key"];  
            $time_to_live = 600;  
            $timestamp = time() + $time_to_live;//失效时间  
            $response_username = $timestamp.":".$_GET["username"];  
            $response_key = $request_key;  
            if(empty($response_key))  
            $response_key = "code_key"; //constants.py中CEOD_KEY  

            $response_password = getSignature($response_username, $response_key);  

            $jsonObj = new Response();  
            $jsonObj->username = $response_username;  
            $jsonObj->password = $response_password;  
            $jsonObj->ttl = 86400;
            //此处需配置自己的服务器
            $jsonObj->uris= array("stun:webrtc.olcms.com:3478","turn:webrtc.olcms.com:3478?transport=udp","turn:webrtc.olcms.com?transport=tcp");

            echo json_encode($jsonObj);  

        /**   
         * 使用HMAC-SHA1算法生成签名值   
         *   
         * @param $str 源串   
         * @param $key 密钥   
         *   
         * @return 签名值   
         */
        function getSignature($str, $key) {
        $signature = "";
        if (function_exists('hash_hmac')) {
        $signature = base64_encode(hash_hmac("sha1", $str, $key, true));
        } else {
        $blocksize = 64;
        $hashfunc = 'sha1';
        if (strlen($key) > $blocksize) {
        $key = pack('H*', $hashfunc($key));
        }
        $key = str_pad($key, $blocksize, chr(0x00));
        $ipad = str_repeat(chr(0x36), $blocksize);
        $opad = str_repeat(chr(0x5c), $blocksize);
        $hmac = pack(
        'H*', $hashfunc(
        ($key ^ $opad) . pack(
        'H*', $hashfunc(
        ($key ^ $ipad) . $str
        )
        )
        )
        );
        $signature = base64_encode($hmac);
        }
            return $signature;
            }

            class Response {  
                public $username = "";  
                public $password = "";  
                public $ttl = "";  
                public $uris = array("");  
            }  

        ?> 
```

iceconfig.php文件内容

```
<?php  
            $request_username = "inesadt";  //配置成自己的turn服务器用户名
            if(empty($request_username)) {  
                echo "username == null";  
                exit;  
            }  
            $request_key = "inesadt";  //配置成自己的turn服务器密码
            $time_to_live = 600;  
            $timestamp = time() + $time_to_live;//失效时间  
            $response_username = $timestamp.":".$_GET["username"];  
            $response_key = $request_key;  
            if(empty($response_key))  
            $response_key = "CEOD_KEY";//constants.py中CEOD_KEY  

            $response_password = getSignature($response_username, $response_key);  

            $arrayObj = array();
            $arrayObj[0]['username'] = $response_username;
            $arrayObj[0]['credential'] = $response_password;
            //配置成自己的stun/turn服务器
            $arrayObj[0]['urls'][0] = "stun:webrtc.olcms.com:3478";
            $arrayObj[0]['urls'][1] = "turn:webrtc.olcms.com:3478?transport=tcp";
            $arrayObj[0]['uris'][0] = "stun:webrtc.olcms.com:3478";
            $arrayObj[0]['uris'][1] = "turn:webrtc.olcms.com:3478?transport=tcp";
            $jsonObj = new Response();  
            $jsonObj->lifetimeDuration = "300.000s";
            $jsonObj->iceServers = $arrayObj;
            echo json_encode($jsonObj);  

            /**   
            * 使用HMAC-SHA1算法生成签名值   
            *   
            * @param $str 源串   
            * @param $key 密钥   
            *   
            * @return 签名值   
            */
            function getSignature($str, $key) {
                $signature = "";
                if (function_exists('hash_hmac')) {
                    $signature = base64_encode(hash_hmac("sha1", $str, $key, true));
                } else {
                    $blocksize = 64;
                    hashfunc = 'sha1';
                    if (strlen($key) > $blocksize) {
                        $key = pack('H*', $hashfunc($key));
                    }
                    $key = str_pad($key, $blocksize, chr(0x00));
                    $ipad = str_repeat(chr(0x36), $blocksize);
                    $opad = str_repeat(chr(0x5c), $blocksize);
                    $hmac = pack(    
                    'H*', $hashfunc(    
                            ($key ^ $opad) . pack(    
                                    'H*', $hashfunc(    
                                            ($key ^ $ipad) . $str    
                                   )    
                            )    
                        )    
                    ); 
                    $signature = base64_encode($hmac);
                }
                return $signature;
           }

            class Response {
                    public $lifetimeDuration = "";
                    public $iceServers = array("");
            } 
        ?>
```

重启Nginx服务器和php7.0-fpm

    service nginx restart 
    service php7.0-fpm restart


## 7. 运行测试 ##


**PC浏览器（Android手机浏览器）之间的视频通信测试**

访问 https://webrtc.olcms.com 

1.PC浏览器：Firefox 54.0.1(64bits)，Android手机浏览器：Firefox 54.0.1

测试OK

2.PC浏览器：Google Chrome 59.0.3071.115(64bits)，Android手机浏览器：Google Chrome 59.0.3071.125

测试OK


**Android APK客户端之间以及客户端与浏览器之间**

1.获取Android APK

下载webrtc源码，在源码目录下webrtc/examples/androidapp，进行编译即可生成Android APK

2.测试Android APK客户端之间

测试OK

3.测试Android APK客户端与浏览器（Chrome、Firefox)之间

测试OK

## 附录 ##

运行过程中的问题

Failed to start signaling: Failed to execute ‘pushState’ on ‘History’: A history state object with URL ‘http://192.168.6.54/r/198676628’ cannot be created in a document with origin ‘https://192.168.6.54’ and URL ‘https://192.168.6.54/

解决方法1： 

房间服务器编译完成后，在/root/apprtc/out/app_engine/js/apprtc.debug.js文件中找到window.history.pushState({‘roomId’: roomId, ‘roomLink’: roomLink}, roomId, roomLink)，把这句话注释掉，重新运行即可。（如果重新编译，需要重新修改）

解决方法2： 

在/root/apprtc/src/web_app/js/appcontroller.js文件中找到window.history.pushState({‘roomId’: roomId, ‘roomLink’: roomLink}, roomId, roomLink)，把这句话注释掉，然后重新编译，重新运行房间服务器即可。


参考： 

1.http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html 

2.http://www.jianshu.com/p/3a43233b9c39

原文链接：https://www.cnblogs.com/qcjd/p/9324877.html