# TrueNAS上使用Ubuntu安装Confluence

## 配置要求

  Server Hardware Requirements Guide：https://confluence.atlassian.com/conf85/server-hardware-requirements-guide-1283361319.html
  CPU: Quad core 2GHz+ CPU
  内存: 6GB
  最小硬盘空间: 10GB

## 软件环境

  Supported Platforms：https://confluence.atlassian.com/conf719/supported-platforms-1157467959.html
  | 项目   | 版本   | 下载地址   |
  |-------|-------|-------|
  | 虚拟机 | TrueNAS-SCALE-23.10.2 | https://www.truenas.com/download-truenas-scale
  | Confluence |7.19.20 | https://www.atlassian.com/software/confluence/download-archives |
  | 系统| ubuntu-20.04.6-live-server-amd64 | https://ubuntu.com/download/server#downloads|
  |SSH远程控制软件|FinalShell| https://www.hostbuf.com/t/988.html|
  |MySQL|mysql-server_8.0.22-1ubuntu20.04_amd64.deb-bundle.tar|https://downloads.mysql.com/archives/community|
  |MySQL 驱动|mysql-connector-java_8.0.22-1ubuntu20.04_all.deb|https://downloads.mysql.com/archives/c-j|
  
## 安装系统

  创建虚拟机
  安装Ubantu

## Ubuntu基础配置
#### 1 - 修改Root可以远程登录

```bash
sudo passwd root
```

#### 2- 配置 SSH 服务器的行为和参数设置

```bash
vi /etc/ssh/sshd_config
```
你需要：配置文件里面的注释 PermitRootLogin和PasswordAuthentication前面的#并把参数改成yes 

vi操作逻辑
输入i 插入模式，只能输入文字，在i模式里按esc退回命令行模式 删除文字，在命令行模式下按：输入 wq 就可以保存

#### 使用FinalShell登录服务器
为什么要使用Finalshell，因为vi实在太难用了。

#### 更改网卡地址
1 - 配置文件地址：/etc/netplan/00-installer-config.yaml
以下的要修改的配置文件
```bash
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens3:
      dhcp4: no
      addresses: [192.168.1.6/24] #这里是你希望设置的IP地址
      optional: true 
      gateway4: 192.168.1.3 #这里是网关地址（通常是路由器地址）
      nameservers:
           addresses: [192.168.1.3] #这个是DNS地址和路由器一样就OK
  version: 2

#生效配置
sudo netplan apply
```

#### 禁止服务器睡眠
```bash
 systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target

#查看是否成功睡眠
systemctl status sleep.target
```

#### -安装JAVA
```bash
#更新软件包列表
sudo apt update

#安装JAVA 11
sudo apt install openjdk-11-jre openjdk-11-jdk

#检查JAVA版本号
java --version
```

#### 安装MYSQL
文件准备：
在mac解压缩 mysql-server_8.0.22-1ubuntu20.04_amd64.deb-bundle.tar 把文件拷贝到mnt/mysql里面
把文件拷贝到/mnt ：  /mnt/mysql-connector-java_8.0.22-1ubuntu20.04_all.deb

```bash
#安装依赖
sudo apt update
sudo apt install libaio1 libmecab2 libnuma1 psmisc

#进入mnt文件夹依次执行一下命令
#1
sudo dpkg -i mysql-community-server-core_8.0.22-1ubuntu20.04_amd64.deb
#2
sudo dpkg -i mysql-community-client-plugins_8.0.22-1ubuntu20.04_amd64.deb
#3
sudo dpkg -i mysql-community-client-core_8.0.22-1ubuntu20.04_amd64.deb
#4
sudo dpkg -i mysql-common_8.0.22-1ubuntu20.04_amd64.deb
#5
sudo dpkg -i mysql-community-client_8.0.22-1ubuntu20.04_amd64.deb
#6
sudo dpkg -i mysql-client_8.0.22-1ubuntu20.04_amd64.deb
#7
sudo dpkg -i mysql-community-server_8.0.22-1ubuntu20.04_amd64.deb
#8
sudo dpkg -i libmysqlclient21_8.0.22-1ubuntu20.04_amd64.deb
#9
sudo dpkg -i mysql-server_8.0.22-1ubuntu20.04_amd64.deb

#核对安装

#查看版本
mysql --version

#查看运行状态
sudo service mysql status
```










