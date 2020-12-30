---
title: 在vscode上连接远程服务器，使用带有GUI的工具
data: 2020-12-30 14:55:00 +0800
categories: [Develop]
tags: [vscode,remote,ssh,gui,xming]
---
在使用vscode的remote-ssh插件连接远程服务器写代码时，当调用含有GUI的工具时，比如gvim，会发生Cannot connect to display的错误，需要在本地搭建图形化服务，使得远程输出的图像化界面能在本地显示。

# 本地配置
## 安装Xming
图形化服务我个人选择Xming，因为网上的教程计较多，配置起来也很方便。

1.下载地址：<https://sourceforge.net/projects/xming/?source=typ_redirect>

2.安装：一路点next即可。

3.配置：打开Xming安装目录找到X0.host文件，在localhost下插入远程服务器ip即可。如下192.168.2.100就是远程服务器ip地址，不清楚地址可以用ifconfig在远程服务器上查看。
```
localhost
192.168.2.100
```
# 远程服务器配置
在工程配置目录.vscode中添加launch.json,添加如下配置,其中192.168.2.88是开启Xming服务的本地机器ip地址。
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "env":{
                "DISPLAY":"192.168.2.88:0.0"
            },
        }
    ]
}
```

如果要在远程服务器的terminal中使用图形化界面，同理需要将环境变量DISPLAY指向本地的图形化界面服务器：
```shell
export DISPLAY="192.168.2.88:0.0"
```