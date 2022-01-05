# WebRTC-Android编译 #

## 一、利用docker的ubuntu镜像安装 ##

1、安装docker

2、下载docker镜像

	docker run --rm -v <WebRTC Android/Linux 代码库的绝对路径>:/webrtc  -it piasy/webrtc-build

上面的piasy/webrtc-build是docker上制作的一个ubuntu镜像。
<WebRTC Android/Linux 代码库的绝对路径>这个括号里的目录是自己电脑上的真实目录，后面的:/webrtc意思是会docker上会有个webrtc目录映射到刚才那个真实目录。以后改真实目录的文件也会体现在docker上的，在docker上改动也会体现在电脑上，非常方便。

首次执行会下载Docker镜像，需要等会儿。启动成功后，命令行会变成Docker镜像实例的命令行，这个命令行里已经配置好了WebRTC Android/Linux开发所需要的环境了。

3、更新depot_tools代码

	cd /depot_tools && git pull
 
4、务必确保翻墙能力来下载源码

```
cd webrtc
fetch --nohooks webrtc_android
#上条命令执行完后，可以编辑当前目录下的.gclient文件
#在taget_os里加上linux，这样这套代码也就可以用于Linux开发了
```

5、如果中途失败，就执行以下命令gclient sync

6、执行./build/install-build-deps.sh，这里会下载很多依赖

## 二、利用虚拟机的Ubantu安装 ##

1、安装VMware Workstation

2、下载Ubantu镜像，选择18.04版本的

3、利用VMware安装虚拟机，注意：磁盘容量一定要40G以上，因为系统本身就很大，加上源码下载完就要20G，随便就超过30G了。

4、下载depot_tools

	git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

下载完之后需要设置环境变量vi ~/.bashrc，增加以下内容

```
export PATH=$PATH:/path/depot_tools  #在当前环境变量追加路径，
#path是自己的depot_tools文件夹的上级路径
```

完了，执行source ~/.bashrc，然后执行which gn或者which gclient测试下，有过有内容说明设置成功

5、务必确保在翻墙环境下载源码

```
cd webrtc
fetch --nohooks webrtc_android
#上条命令执行完后，可以编辑当前目录下的.gclient文件
#在taget_os里加上linux，这样这套代码也就可以用于Linux开发了
```

6、如果中途失败，就执行以下命令gclient sync

7、执行./build/install-build-deps.sh，这里会下载很多依赖

8、当所有依赖都下载完成的界面是这样

```
Syncing projects: 100% (307/307), done. 

Hook 'vpython src/build/landmines.py --landmine-scripts src/tools_webrtc/get_landmines.py --src-dir src' took 11.30 secs
Running hooks:  50% (11/22) clang                         
________ running 'vpython src/tools/clang/scripts/update.py' in '/webrtc'
Downloading https://commondatastorage.googleapis.com/chromium-browser-clang/Linux_x64/clang-n356235-f7f1abdb-1.tgz .......... Done.
Hook 'vpython src/tools/clang/scripts/update.py' took 27.31 secs
Running hooks:  86% (19/22) test_fonts        
________ running 'download_from_google_storage --no_resume --extract --no_auth --bucket chromium-fonts -s src/third_party/test_fonts/test_fonts.tar.gz.sha1' in '/webrtc'
0> Downloading src/third_party/test_fonts/test_fonts.tar.gz@cd96fc55dc243f6c6f4cb63ad117cad6cd48dceb...
0> Extracting 33 entries from src/third_party/test_fonts/test_fonts.tar.gz to src/third_party/test_fonts/test_fonts
Downloading 1 files took 34.387538 second(s)
Hook 'download_from_google_storage --no_resume --extract --no_auth --bucket chromium-fonts -s src/third_party/test_fonts/test_fonts.tar.gz.sha1' took 34.47 secs
Running hooks:  90% (20/22) msan_chained_origins
________ running 'vpython src/third_party/depot_tools/download_from_google_storage.py --no_resume --no_auth --bucket chromium-instrumented-libraries -s src/third_party/instrumented_libraries/binaries/msan-chained-origins-trusty.tgz.sha1' in '/webrtc'
0> Downloading src/third_party/instrumented_libraries/binaries/msan-chained-origins-trusty.tgz@92ab0ac3915bc2b50e6a8d7a990522d7355d1ef0...
Downloading 1 files took 96.771148 second(s)
Hook 'vpython src/third_party/depot_tools/download_from_google_storage.py --no_resume --no_auth --bucket chromium-instrumented-libraries -s src/third_party/instrumented_libraries/binaries/msan-chained-origins-trusty.tgz.sha1' took 97.55 secs
Running hooks:  95% (21/22) msan_no_origins     
________ running 'vpython src/third_party/depot_tools/download_from_google_storage.py --no_resume --no_auth --bucket chromium-instrumented-libraries -s src/third_party/instrumented_libraries/binaries/msan-no-origins-trusty.tgz.sha1' in '/webrtc'
0> Downloading src/third_party/instrumented_libraries/binaries/msan-no-origins-trusty.tgz@b02322f976be38aa177febfc8616546b7a23b25e...
Downloading 1 files took 67.776867 second(s)
Hook 'vpython src/third_party/depot_tools/download_from_google_storage.py --no_resume --no_auth --bucket chromium-instrumented-libraries -s src/third_party/instrumented_libraries/binaries/msan-no-origins-trusty.tgz.sha1' took 68.71 secs
Hook 'download_from_google_storage --directory --recursive --num_threads=10 --no_auth --quiet --bucket chromium-webrtc-resources src/resources' took 14.28 secs
Running hooks: 100% (22/22), done.
```

## 四、编译 ##

1、设置编译参数，生成ninja文件

```
gn gen out/build --args='target_os="android" target_cpu="arm" is_debug=false'
#out/build ： 编译生成文件的目录，随意指定
#target_os ： 编译目标平台 android ios 等
#target_cpu ： CPU架构平台 arm arm64 x86 x64等
i#s_debug : Release模式或者Debug模式
```

2、编译

```
#全量编译
ninja -C out/build
#debug编译
ninja -C out/Debug AppRTCMobile
```

3、生成aar

```
cd webrtc/src
python tools_webrtc/android/build_aar.py --output "libwebrtc.aar" --arch "armeabi-v7a" "arm64-v8a" --build-dir out/Release
#成功后你会在src目录下看到libwebrtc.aar文件，里面就是Android开发需要用到的SDK了。
#out/Release目录是编译目录，第一编译会全量编译速度很慢（预计30~40分钟），以后就增量编译很快（预计10s内）。
```

4、生成动态库.so文件，生成so文件首次会全量编译，后续增量编译速度非常快。如果以后只改C层代码不生成Java或者Object-C的API，这种方法非常适合测试。

```
ninja -C out/Debug sdk/android:libjingle_peerconnection_so
```

5、生成静态库.a文件

```
ninja -C out/Debug
生成的.a文件在out/Debug/obj/libwebrtc.a
```

## 五、编译错误锦集 ##

1、No such file or directory: '/webrtc/src/build/util/LASTCHANGE.committime

解决方式如下：

	python build/util/lastchange.py build/util/LASTCHANGE

2、third_party/llvm-build/Release+Asserts/bin/clang++: No such file or directory

```
ninja: Entering directory `out/ios'
[1/2732] CXX obj/api/audio_codecs/audio_codecs_api/audio_codec_pair_id.o
FAILED: obj/api/audio_codecs/audio_codecs_api/audio_codec_pair_id.o 
../../third_party/llvm-build/Release+Asserts/bin/clang++ -MMD -MF obj/api/audio_codecs/audio_codecs_api/
/bin/sh: ../../third_party/llvm-build/Release+Asserts/bin/clang++: No such file or directory
...
/bin/sh: ../../third_party/llvm-build/Release+Asserts/bin/clang++: No such file or directory
...
isystem../../buildtools/third_party/libc++abi/trunk/include -fvisibility-inlines-hidden -Wnon-virtual-dtor -Woverloaded-virtual -c ../../api/audio_codecs/ilbc/audio_encoder_ilbc.cc -o obj/api/audio_codecs/ilbc/audio_encoder_ilbc/audio_encoder_ilbc.o
/bin/sh: ../../third_party/llvm-build/Release+Asserts/bin/clang++: No such file or directory
ninja: build stopped: subcommand failed.
```

解决方式如下：

	python src/tools/clang/scripts/update.py

3、E: Unable to locate package lib32gcc-s1

```
E: Unable to locate package lib32gcc-s1
The following command failed:  apt-get --just-print install libasound2:i386 
libcap2:i386 libelf-dev:i386 libfontconfig1:i386 libglib2.0-0:i386 libgpm2:i386 
libncurses5:i386 libnss3:i386 libpango-1.0-0:i386 libpci3:i386 libssl-dev:i386 
libssl1.0.0:i386 libtinfo-dev:i386 libudev1:i386 libuuid1:i386 libx11-xcb1:i386
 ...
```

这个是因为之前下载依赖的时候部分失败，解决方式：gclient sync，继续同步下载代码

4、ld.lld: error: ../../build/linux/debian_sid_amd64-sysroot/usr/lib/x86_64-linux-gnu/libdl.so:1: unknown directive: XSym

```
itcayman@ubuntu:~/webrtc/src$ ninja -C out/build sdk/android:libjingle_peerconnection_so 
ninja: Entering directory `out/build'
[1/533] LINK clang_x64/protoc
FAILED: clang_x64/protoc 
../../third_party/llvm-build/Release+Asserts/bin/clang++ -Wl,--fatal-warnings -Wl,--build-id -fPIC -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -Wl,-z,defs -Wl,--as-needed -fuse-ld=lld -Wl,--icf=all -Wl,--color-diagnostics -m64 -Wl,-O2 -Wl,--gc-sections -rdynamic -nostdlib++ --sysroot=../../build/linux/debian_sid_amd64-sysroot -L../../build/linux/debian_sid_amd64-sysroot/usr/local/lib/x86_64-linux-gnu -L../../build/linux/debian_sid_amd64-sysroot/lib/x86_64-linux-gnu -L../../build/linux/debian_sid_amd64-sysroot/usr/lib/x86_64-linux-gnu -pie -Wl,--disable-new-dtags -Werror -o "clang_x64/protoc" -Wl,--start-group @"clang_x64/protoc.rsp"  -Wl,--end-group  -ldl -lpthread -lrt
ld.lld: error: ../../build/linux/debian_sid_amd64-sysroot/usr/lib/x86_64-linux-gnu/libdl.so:1: unknown directive: XSym
>>> XSym
>>> ^
clang: error: linker command failed with exit code 1 (use -v to see invocation)
ninja: build stopped: subcommand failed.
```

这是因为clang链接需要的动态库不对，一般是从硬盘copy WebRTC源码到虚拟机会出现这个问题。解决方案：

```
#第一步，删除src/build/linux/debian_sid_amd64-sysroot目录
rm -rf build/linux/debian_sid_amd64-sysroot
#第二步，重新下载生成
build/linux/sysroot_scripts/install-sysroot.py --arch="amd64"
```

5、ERROR at //BUILD.gn:33:17: Can't load input file.

```
itcayman@ubuntu:~/webrtc/src$ gn gen out/Debug --args='target_os="android" target_cpu="arm" is_debug=false'
ERROR at //BUILD.gn:33:17: Can't load input file.
      deps += [ "examples" ]
                ^---------
Unable to load:
  /home/itcayman/webrtc/src/examples/BUILD.gn
I also checked in the secondary tree for:
  /home/itcayman/webrtc/src/build/secondary/examples/BUILD.gn

这个是因为编译重复依赖了examples，我们只需要去掉编译examples和test模块就行

gn gen out/Debug --args='target_os="android" target_cpu="arm" is_debug=false rtc_include_tests=false rtc_build_examples=false'
```

6、gclient sync过程中报错You have unstaged changes.Please commit, stash, or reset.

```
Error: 3> 
3> ____ src/build at a03951acb996e9cea78b4ab575896bf1bfcd9668
3>  You have unstaged changes.
3>  Please commit, stash, or reset.
```

这是因为同步下载依赖的过程中部分文件下载失败，只需要删除掉第一行出现的目录。再次执行gclient sync就行，像上面，只需要执行

```
# 第一步，删除对应目录
rm -rm src/build/
# 第二步，再次同步下载依赖
gclient sync
```

7、/usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.26' not found

	sudo sudo apt-get install software-properties-common && add-apt-repository ppa:ubuntu-toolchain-r/test && sudo apt update && sudo apt install gcc-9 && sudo apt-get install libstdc++6

## 六、切换分支 ##

1、查看所有分支

	git branch -a

2、切换到具体分支

```
#切换到4183分支举例
git checkout remotes/branch-heads/4183  
```

3、执行gclient sync，这个时候会报错

```
WARNING: Your metrics.cfg file was invalid or nonexistent. A new one will be created.
Syncing projects:   2% ( 7/307) src/base   

src/third_party (ERROR)
----------------------------------------
[0:00:02] Started.
[0:00:02] Finished running: git config remote.origin.url
[0:00:02] Finished running: git rev-list -n 1 HEAD
[0:00:02] Finished running: git rev-parse --abbrev-ref=strict HEAD
----------------------------------------
Error: 7> 
7> ____ src/third_party at e0df6e10adc084f88dda51c0cbab84645db6c135
7>  Your repo is locked, possibly due to a concurrent git process.
7>  If no git executable is running, then clean up '/webrtc/src/third_party/.git/index.lock' and try again.
```

然后我们根据提示删除掉对应目录重新执行glient sync就行

```
rm -rf src/thrid_party
rm -rf src/examples
rm -rf src/tools
rm -rf src/build
gclient sync
```

## 友情提示 ##

如果编译出现其它错误，一般都会在命令行写出解决方案的，只要执行上面的提示命令一般能解决问题。

2人点赞

WebRTC


作者：itcayman
链接：https://www.jianshu.com/p/a9a293a443fa
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。