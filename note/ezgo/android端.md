[TOC]



### 1. Unable to run Ionic app after update to Android Studio 3.0

### 2. Dependency failing: com.nimbusds:nimbus-jose-jwt:5.1 -> net.minidev:json-smart@[1.3.1,2.3], but json-smart version was 2.3

https://github.com/invertase/react-native-firebase/issues/1676

修改 google-services 4.2.0 版本为 4.1.0 或 4.3.3

I had this problem with `com.google.gms:google-services:4.2.0`
downgrading to 4.1.0 or upgrading to 4.3.3 will fix this issue

无法解决，在 app 的 `build.gradle` 最后加上：

```
googleServices.disableVersionCheck = true
```



### 3. 导入 android stadio

https://www.jianshu.com/p/451993967237，导入已有项目，双击build.gradle

### 4. Unable to resolve dependency for ':app@debug/compileClasspath': Could not resolve com.android.support:support-v4:24.1.1+

https://www.tfzx.net/article/2748926.html

```
debugCompile project(path: ":CordovaLib", configuration: "debug")
releaseCompile project(path: ":CordovaLib", configuration: "release")
```

Replace them with

```
implementation project(path: ":CordovaLib")
```

https://www.cnblogs.com/shmilyGWT/p/8425213.html

### 5. uses-sdk:minSdkVersion 19 cannot be smaller than version 22 declared in library

gradle.properties 修改 

```
cdvMinSdkVersion=22
```

以上无效，可以在 `AndroidManifest.xml` 中添加

```
    <uses-sdk tools:overrideLibrary="org.apache.cordova" />
```

需要在根目录加上命名空间 

```
xmlns:tools="http://schemas.android.com/tools"
```

### 6. MACOS - 打包 apk

https://blog.csdn.net/angles1/article/details/89842929

https://segmentfault.com/a/1190000013755356

### 7.推送服务提示 Got Resources NotFoundException

重新安装 phonegap-plugin-push 插件即可

```shell
cordova plarform remove android
cordova plugin remove phonegap-plugin-push
cordova plugin add phonegap-plugin-push
cordova platform add android
```





