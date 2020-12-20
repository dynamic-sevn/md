* 运行 docker 镜像

  ```shell
  docker run -p 8080:80 -e DB_DRIVER="postgres" -e DB_DSN="host=127.0.0.1 user=sunhaoran password= dbname=sunhaoran sslmode=disable"  toolbox:test -s command server
  ```

* 司机端运行

  ```shell
  # 浏览器运行
  npm run cordova-serve-browser 
  ```

  ```shell
  # cordova 打包
  npm run build:cordova
  ```

* 存在的问题

  * 司机端上传图片，COS_SECRET_ID 是什么？
  * 司机端白屏，没找到 PLATFORM？
  * twillo 申请, 找回 RZ7fATG56Vt2K3dY63tm1l5y-VQ-JqGScjy4yZiT 

* ios 打包

  * 使用命令行打包

    ```shell
    npm run cordova-build-ios
    ```

  * 使用命令行 run

    ```shell
    npm run cordova-serve-ios
    ```

    报错

    ```shell
    No platforms installed in '/Users/sunhaoran/WebstormProjects/toolbox-driver/src-cordova', please execute "cordova platform add ios" in /Users/sunhaoran/WebstormProjects/toolbox-driver/src-cordova
    ```

  * 执行 cordova platform add ios ，卡在 `Using cordova-fetch for cordova-ios@^5.0.1` 不动

    可能因为已经添加了，但是不报错很蛋疼。

    

  * 新建了一个目录，重新进行进行 cordova ios build 和安装

    起初直接进行安装，跑起来之后白屏，使用 alert 调试，发现在使用到插件的时候走不下去了，原因是没有安装插件。

    把原来的 package.json 和其他文件拷贝到新文件里，重新进行 remove，然后 add，此时在下载 cordova-plugin-googlemaps 卡住了，但看 iterm 状态栏的速度有大几百 K，卡了好久。。。

    于是想速度更快一点，虽然本机已经搭了梯子，但全局代理对于 iterm2 这类终端是不生效的

    ![image-20200827102313173](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200827102313173.png)

    google 之，https://github.com/mrdulin/blog/issues/18 这个可以，然后终端执行 `proxy` , 下载速度大到 M 级别，但仍然等了很久才下完。。

    然后报了很多依赖安装不了的错误，具体内容就是 GitHub: invalid username or password，然后 google 之 https://stackoverflow.com/questions/29297154/github-invalid-username-or-password，我把 package.json 包里的 http 协议全改成了 ssh 协议，**但具体 http 怎么整仍不知道**，然后下载成功了。

    

    以为可以运行了，但打包成 xcode 项目后，报了编译错误。。

    ```shell
    Module 'FirebaseInstanceID' not found
    ```

    

    google 之，因为 CocoaPods 没有安装，按 https://github.com/phonegap/phonegap-plugin-push/issues/1825 说的做了

    

    安装好了再打包运行，报 

    ```shell
    No known instance method for selector 'userAgent' in CDVFileTransfer.m
    ```

    google 之，https://github.com/apache/cordova-plugin-file-transfer/issues/258 删掉了107-110 行

    

    再 build 报

    ```shell
    library not found for -lGoogleToolboxForMac
    ```

    再 google https://github.com/google/google-toolbox-for-mac/issues/137

    原来是打开姿势不对 Please close any current Xcode sessions and use MyApp.xcworkspace for this project from now on. 要打开 .xcworkspace 文件而不是 .xcodeproj 文件。

    

    然后再在 xcode 运行 ，这次可以了，报了一个 `GOOGLE_MAPS_IOS_KEY is not registry` , 原来漏了原src-cordova里的配置文件，拷贝出来加上，再 build 运行，但进去之后报了一个无法使用推送消息的错就又白屏了。。。不过至少说明已经开始加载主程序了。。。

  * 服务器部署

    安装 docker

    

    安装 postgis https://yq.aliyun.com/articles/591859

    ```shell
    postgres && cd postgres
    docker run --name=postgis -d -e POSTGRES_USER=shr -e POSTGRES_PASS=admin -e ALLOW_IP_RANGE=0.0.0.0/0 -p 5432:5432 --restart=always kartoza/postgis:9.6-2.4
    ```

    安装 git 

    ```shell
    yum install git
    ```

    直接跑 deploy.sh

    ```shell
    toolbox/deploy.prod.sh
    ```

    失败了，报没有 go 命名

    下载 go

    ```shell
    wget https://dl.google.com/go/go1.15.linux-amd64.tar.gz
    ```

    解压

    ```shell
    tar -C /usr/local -xzf go1.15.linux-amd64.tar.gz
    ```

    打包报错

    ```shell
    cannot find package "bitbucket.org/gsoes/toolbox/cmd" in any of:
    	/usr/local/go/src/bitbucket.org/gsoes/toolbox/cmd (from $GOROOT)
    	/usr/src/src/bitbucket.org/gsoes/toolbox/cmd (from $GOPATH)
    ```

    因为 gopath 没有配置好，gopath 为当前代码所在的根目录，goroot 为go包所在的位置

    因此 export 一下 gopath 到正确位置

    打包又报错

    ```shell
    graphql/driver/mutation.go:8:2: cannot find package "github.com/luaxlou/goutils/tools/logutils" in any of:
    	/usr/src/bitbucket.org/gsoes/toolbox/vendor/github.com/luaxlou/goutils/tools/logutils (vendor tree)
    	/usr/local/go/src/github.com/luaxlou/goutils/tools/logutils (from $GOROOT)
    	/usr/src/github.com/luaxlou/goutils/tools/logutils (from $GOPATH)
    ```

    因为依赖了 github 包，没有找到

    所以 

    ```shell
    go get [没找到的域名]
    ```

    然后运行

    ```shell
    docker run -d -p 80:80 -e DB_DRIVER="postgres" -e DB_DSN="host=47.254.27.52 user=shr password=admin dbname=gis sslmode=disable" -e POSTGRES_DB="gis" -e POSTGRES_PASSWORD="admin" -e POSTGRES_USER="shr" -e SMS_EXPIRE_IN_MINUTE="10" -e SMS_PREFIX="EZGO" -e SMS_TWILIO_PHONE="+15614755784" -e SMS_TWILIO_SID="AC5d1c3e42e64b17548b23dab9c3057c96" -e SMS_TWILIO_TOKEN="70b3b87e5447f29662692e5549ba4f8a" docker.io/masterxsun/toolbox:prod -s command server
    ```

    其中 d 为以守护进程运行，web 一般采用该方式

    >1. 前端怎么部署？
    >2. 怎么能够 http 访问到容器？
    >3. tracker 部署在一个里面是怎么部署的，会不会端口占用呢

    由于原项目分了两个模块，因此无法直接两个都使用 80 端口部署，所以，两个端口要不同, 容器里两个端口都是80，可以运行两个docker镜像将虚拟机不同的端口都映射到 80 端口。目前是开了两个端口。

  * 下面部署前端 docker 项目

    下载安装 node 

    ```
    wget https://nodejs.org/dist/v12.18.3/node-v12.18.3-linux-x64.tar.xz
    ```

    解压

    ```
    xz -d node-v9.8.0-linux-x64.tar.xz
    tar -xvf node-v9.8.0-linux-x64.tar
    ```

    >```shell
    >docker exec -it 63805d14f2cd /bin/bash #进入容器
    >```

    然后环境变量配置...

    运行 npm install，提示没有权限，使用 sudo，提示没有命令

    所以，使用如下方法将 node 和 npm 命令添加到 sudo

    参考 https://www.jianshu.com/p/efce6216dc8e，将 npm 和 node 软链到全局sudo下, 首先要进入 /usr/bin 目录

    ```shell
    cd /usr/bin$
    sudo ln -s /opt/node-v8.16.0-linux-x64/bin/npm
    sudo ln -s /opt/node-v8.16.0-linux-x64/bin/node
    ```

    再 npm install 安装依赖，然后运行文件目录下的 ./deploy.prod.sh，进行打包，由于没有 nginx 环境，是没发成功的。所以下面安装 nginx

  * 安装 nginx

    参考

    使用命令

    ```shell
    sudo yum install nginx
    ```

    安装完成后使用 `nginx` 命令执行是没法成功的，因为之前我们占用了 80 端口，nginx 默认使用 80 端口

    因此修改 nginx 的配置文件，让其监听 8081 端口

    具体位置在

    ```shell
    /etc/nginx/nginx.conf
    ```

  * 此时我们通过 ip:8081 应能访问 nginx 的主页，其html静态文件可以在配置文件中配置，默认位置为：

    ```
    /usr/share/nginx/html/
    ```

    我们要做的是使用我们自己的静态文件替换原有的默认页面，实际上项目里的 deploy文件已经写了这部分批处理命令，但是执行了之后不生效，不知为何

    因此自己手动 npm run build 了一把将文件生成然后复制过去，不知道有没有问题

    ```shell
    cp -r dist/. /usr/share/nginx/html/
    ```

    复制之后执行 nginx，然后前端可以访问页面了。

    > unalias cp

* IOS 打包

  * 出口审查，在xcode项目中plist里设置`App Uses Non-Exempt Encryption` 为 NO

* 状态栏被遮挡问题

  viewWillAppear:(**BOOL**)animated 中

  MainViewController.m 中

  ```
      if ([[[UIDevice currentDevice] systemVersion]floatValue] >= 7 && [[[UIDevice currentDevice] 	systemVersion]floatValue] < 10) {
          CGRect viewBounds = [self.webView bounds];
          viewBounds.origin.y = 20;
          viewBounds.size.height = viewBounds.size.height - 20;
          self.webView.frame = viewBounds;
      } else if ([[[UIDevice currentDevice] systemVersion]floatValue] >= 10) {
          CGRect viewBounds = [self.webView bounds];
          viewBounds.origin.y = 44;
          viewBounds.size.height = viewBounds.size.height - 44;
          self.webView.frame = viewBounds;
      }
  ```

  ![image-20200914013858134](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200914013858134.png)

* twilio

  ![image-20200915011308152](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011308152.png)

  

![image-20200915011353959](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011353959.png)

![image-20200915011404482](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011404482.png)

![image-20200915011418432](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011418432.png)

![image-20200915011432178](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011432178.png)

![image-20200915011443282](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011443282.png)

![image-20200915011453681](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011453681.png)

![image-20200915011531132](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011531132.png)

![image-20200915011541075](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011541075.png)

![image-20200915011553542](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011553542.png)

![image-20200915011606859](/Users/sunhaoran/Library/Application Support/typora-user-images/image-20200915011606859.png)

* gcp npm install 报了 `stack Error: not found: make`，需要安装 build-essential

  ```shell
  $ sudo apt-get install build-essential
  ```

  https://stackoverflow.com/questions/14772508/npm-failed-to-install-time-with-make-not-found-error

* 仍然报错，需要清理之前的安装包

  ```shell
  rm package-lock.json && rm -rf node_modules && rm -rf ~/.node-gyp
  ```

  https://github.com/nodejs/node-gyp/issues/1694

* 删除所有已停止的 docker container

  ```shell
  sudo docker rm `docker ps -a|grep Exited|awk '{print $1}'`
  ```

* `systemctl restart v2ray`

