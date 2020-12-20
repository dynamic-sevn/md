# 1.     代码部分说明：

toolbox项目: 服务端Golang代码

docker方式发布，到服务器会有两个实例，一个是toolbox,一个是tracker

toolbox 和tracker 使用不同的启动参数分别运行，详见cmd/main.go

postgis数据库的搭建，详见docker-compose-prod.yml

数据结构: 使用的是gorm 的automigrate,程序运行后数据表会自动创建。

目前https使用cloudflare做代码，没有原来配置证书的方式

目前使用了firebase服务做推送，目前这部分代码有内存泄漏的情况，建议上前重写下。Firebase需要新申请。

使用了twilio 购买的号码进行发短信

启动需要配置的环境变量（目前由stack脚本传入）：

```shell
 "DB_DRIVER=postgres",
 "DB_DSN=host=xxx user=xxx password=xxx dbname=xxx sslmode=disable"
    "GIN_MODE=release",
    "POSTGRES_DB=xxx",
    "POSTGRES_PASSWORD=xxx",
    "POSTGRES_USER=xxx",
    "SMS_EXPIRE_IN_MINUTE=10",
    "SMS_PREFIX=霸姐姐出行",
    "SMS_TWILIO_PHONE=160426",
    "SMS_TWILIO_SID=xxx",
    "SMS_TWILIO_TOKEN=xxx",
```

 

toolbox-admin: 管理后台的前端部分代码，服务端使用的是toolbox。

toolbox-passenger: 乘客端的前端那部分代码，服务端使用的是toolbox。原先作为原生发布，目前是纯wap发布，改回app发布可能地图和定位部分需要做下修改。

toolbox-driver: 服务端使用的是toolbox 以及tracker（位置跟踪部分），原生做发布。App原生构建可能会有找不到包的情况，有些包可能比较老了，建议更改下原生插件。

 

# 2.     部署建议

目前我们使用的是docker-stack+docker-registery方式部署，具体可以toolbox下deploy.prod.sh。并全部放到一台2核4G的服务器。

目前存在的问题是toolbox存在内存泄漏，会导致postgis无法启动。还有目前对于tracker数据的增长也没有很好的规划，建议更改这部分代码。

建议最小配置，两台2核4G服务器，postgis独立放一台，并做好定期备份。

原则上不太建议使用docker-stack,可以考虑使用阿里云的k8s托管

有问题线上交流

 

 