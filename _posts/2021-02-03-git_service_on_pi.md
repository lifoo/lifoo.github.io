---
title: 树莓派搭建Git服务器
data: 2021-02-03 16:46:00 +0800
categories: [Develop]
tags: [git,service]
---

## 安装ssh
```
sudo apt-get install ssh
```

## 启动
树莓派可通过raspi-config开启SSH服务
通过代码启动SSH：
```
systemctl start ssh
```
注意如果apt-get 无法正常请求网络，请检查/etc/network/interfaces 和 resolve.conf 网络配置是否正常。

## 安装git-core
```
sudo apt-get install git-core
```
## 新增git用户
### 添加用户:git
```
adduser --system --shell /bin/bash --gecos 'git version control by pi' --group --home /home/git git
```

### 设置密码
```
passwd git
```

> 注意：很多资料建议修改git shell参数：vim /etc/passwd,找到git用户，将 /bin/bash 改为/bin/bash/git-shell (whereis git-shell)该操作的影响是无法在终端切换到git用户操作

## 设置git用户目录
```
mkdir /home/git
chown -vR git.git /home/git
//git.git 代表git用户组的git用户
```

## 【服务端】设置git仓库
```
cd /home/git

mkdir test.git
chown -vR git.git ./test.git
cd ./test.git

git --bare init
```

## 【客户端】设置git仓库
```
clone git项目
git clone git@xxx.xxx.xxx.xxx:/home/git/test.git
```

> 注意：1. 注意ip地址后的路径：/home/git/test.git,与服务器目录地址保持一致

> 如果没有上传ssh 公钥到服务，需要使用密码登录，此时的密码为git账户密码，如果密码无法登录，请检查sshd_config配置是否允许git用户或git用户组登录

## 本地git项目添加远程仓库
```
git remote add pi git@xxx.xxx.xxx.xxx:/home/git/test.git

git push pi master
```
> 注意：pi为远程仓库别名，默认为origin

## 设置ssh登录
```
cd /home/git
mkdir .ssh
chown -vR git.git ./.ssh

//根据实际测试执行
chmod 755 ./.ssh

cat /tmp/id_rsa.pub >> /home/git/.ssh/authorized_keys

chown -vR git.git ./.ssh/authorized_keys

//根据实际测试执行
chmod 600 ./.ssh/authorized_keys
```
> 说明：1. /tmp/id_rsa.pub为本地上传的公钥文件