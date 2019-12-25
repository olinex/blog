---
description: '本文章由olinex原创, 转载请在页面开头标明出处'
---

# 配置为Windows服务

* 下载[prunsrv.zip](http://archive.apache.org/dist/commons/daemon/binaries/windows/%20)
* 将`prunsrv.exe`复制到`zookeeper\bin`目录, 若系统为64位, 则需要使用`amd64/prunsrv.exe`, 这是服务的执行程序
* 将`prunmgr.exe`复制到`zookeeper\bin`目录, 这是服务的监控行的程序
* 将服务的名称添加到`ZOOKEEPER_SERVICE`环境变量
* 将服务的根目录添加到`ZOOKEEPER_HOME`环境变量
* 在`zookeeper\bin`目录下添加`zkServerStop.cmd` 文件

```bash
@echo off
setlocal
TASKLIST /svc | findstr /c:"%ZOOKEEPER_SERVICE%" > %ZOOKEEPER_HOME%\zookeeper_svc.pid
FOR /F "tokens=2 delims= " %%G IN (%ZOOKEEPER_HOME%\zookeeper_svc.pid) DO (
    @set zkPID=%%G
)
taskkill /PID %zkPID% /T /F
del %ZOOKEEPER_HOME%/zookeeper_svc.pid
endlocal
```

* 在`zookeeper`目录下创建一个`install.bat`文件并运行

```bash
prunsrv.exe "//IS//%ZOOKEEPER_SERVICE%" ^
        --DisplayName="Zookeeper (%ZOOKEEPER_SERVICE%)" ^
        --Description="Zookeeper (%ZOOKEEPER_SERVICE%)" ^
        --Startup=auto --StartMode=exe ^
        --StartPath=%ZOOKEEPER_HOME% ^
        --StartImage=%ZOOKEEPER_HOME%\bin\zkServer.cmd ^
        --StopPath=%ZOOKEEPER_HOME%\ ^
        --StopImage=%ZOOKEEPER_HOME%\bin\zkServerStop.cmd ^
        --StopMode=exe --StopTimeout=5 ^
        --LogPath=%ZOOKEEPER_HOME% --LogPrefix=zookeeper-wrapper ^
        --PidFile=zookeeper.pid --LogLevel=Info --StdOutput=auto --StdError=auto
```

