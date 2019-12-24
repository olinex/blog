---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 配置Windows服务

* 下载[prunsrv.zip](http://archive.apache.org/dist/commons/daemon/binaries/windows/%20)
* 将`prunsrv.exe`复制到`kafka\bin\windows`目录, 若系统为64位, 则需要使用`amd64/prunsrv.exe`, 这是服务的执行程序
* 将`prunmgr.exe`复制到`kafka\bin\windows`目录, 这是服务的监控行的程序
* 将服务的名称添加到`KAFKA_SERVICE`环境变量
* 将服务的根目录添加到`KAFKA_HOME`环境变量
* 在`kafka\bin\windows`目录下添加`startKafka.bat` 文件

```bash
@echo off

echo start Kafka

call %KAFKA_HOME%\bin\windows\kafka-server-start %KAFKA_HOME%\config\server.properties

echo end Kafka
```

* 在`kafka\bin\windows`目录下添加`stopKafka.bat`文件

```bash
@echo off

echo stop Kafka

call %KAFKA_HOME%\bin\windows\kafka-server-stop

echo end stop Kafka
```

* 在`zookeeper`目录下创建一个`install.bat`文件并运行

```bash
prunsrv.exe "//IS//%KAFKA_SERVICE%" ^
        --DisplayName="kafka" ^
        --Description="Kafka Service" ^
        --Startup=auto --StartMode=exe ^
        --StartPath=%KAFKA_HOME% ^
        --StartImage=%KAFKA_HOME%\bin\windows\startKafka.bat ^
        --StopPath=%KAFKA_HOME%\ ^
        --StopImage=%KAFKA_HOME%\bin\windows\stopKafka.bat ^
        --StopMode=exe --StopTimeout=5 ^
        --LogPath=%KAFKA_HOME% --LogPrefix=kafka-wrapper ^
        --PidFile=kafka.pid --LogLevel=Info --StdOutput=auto --StdError=auto
```

* 运行`install.bat`

