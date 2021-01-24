---
title: Linux下使用Clash代理
data: 2021-01-03 20:33:00 +0800
categories: [Tool Box]
tags: [clash,proxy]
---

## 下载clash

[Clash Linux版本下载地址](https://github.com/Dreamacro/clash/releases)

根据自己的CPU及系统架构，选择合适的安装包，可通过以下命令查看：
```shell
pi@raspberrypi:~ $ uname -a
Linux raspberrypi 5.4.79-v7l+ #1373 SMP Mon Nov 23 13:27:40 GMT 2020 armv7l GNU/Linux
pi@raspberrypi:~ $ getconf LONG_BIT
32
pi@raspberrypi:~ $ getconf WORD_BIT
32
```

下载Clash已编译的包：

```
pi@raspberrypi:~ $ wget https://github.com/Dreamacro/clash/releases/download/v1.3.5/clash-linux-armv7-v1.3.5.gz
```

## 解压运行

```
pi@raspberrypi:~ $ gunzip ***.gz
pi@raspberrypi:~ $ chmod +x clash***
```


`./clash***`

此时会提示：

clash会自动在~/.config/下创建clash文件夹，里面有config.yaml 与Country.mmdb 2个文件，config.yaml 为空文件，Country.mmdb为正在下载的文件，等待Country.mmdb下载完成之后关闭clash程序，将config.yaml 替换为自己的托管文件。这个文件一般的机场会提供，如果没有，就按以下命令创建

`pi@raspberrypi:~ $ curl  Clash托管链接   >>  config.yaml`
 
将生成的config.yaml替换~/.config/config.yaml

## 系统配置

打开config.yaml文件，开头如下

```yaml
port: 7890
socks-port: 7891
redir-port: 7892
allow-lan: false
mode: Rule
log-level: info
external-controller: '0.0.0.0:9090'
```

## 运行测试
 
运行clash:

./clash***
打开网页http://clash.razord.top/ ，输入external-controller指定的ip及端口号即可管理clash

### Termial设置代理
可以在~/.bash_aliases加入以下别名，要开启代理时直接setproxy即可
```
alias setproxy='export http_proxy=127.0.0.1:7890 export https_proxy=127.0.0.1:7890'
alias unsetproxy='unset http_proxy unset https_proxy'
```

### 浏览器设置代理
```
chromium-browser --proxy-server="socks5://127.0.0.1:7891"
```

### sbt 代理设置
```
sudo vi /usr/share/sbt/conf/sbtconfig.txt
加入以下配置
 -Dhttp.proxyHost=127.0.0.1
 -Dhttp.proxyPort=7890
 -Dsocks.ProxyHost=127.0.0.1
 -Dsocks.ProxyPort=7891 
```
