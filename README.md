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
### 安装Ubantu
#### Network connections 选择修改网卡地址
    选择网卡端口 > Edit IPV4 > 改成Manual
     subnet：192.168.1.0/24（根据你实际的网段来设置）
     address:192.168.1.108 (你希望这个服务器的地址)
     Gateway：192.168.1.1 （你的网关路由器地址）
     Name server：192.168.1.1（一般和你路由器一样，如果有特别制定你用英文逗号隔开）
#### Configure ubuntu archive mirror 镜像地址修改成国内的
    Mirror address：把us.archive.ubuntu.com/ubuntu 改成 cn.archive.ubuntu.com/ubuntu
#### Guided Storage configuration 修改空间分配
     移动选项到Custom storage layout 然后空格选中 > 选择Done
     1 设置boot分区：选择free speace > add gpt partition > size: 输入500m > format：xfs > mount /boot > create
     boot引导分区通常用于存放引导加载程序和相关的引导文件
     
     2 设置swap分区：选择free speace > add gpt partition > size:8G(内存的两倍) > create
     Swap 分区用于当物理内存不足时，作为虚拟内存的一部分，用于交换出内存中暂时不活跃的数据
     
     创建root分区：选择free speace > add gpt partition > size:留空把剩余空间都给他> format：xfs > mount / > create
    Done > Continue
#### Profile setup 账号设置
     创建具有sudo权限的用户，用来登录系统
     新装Ubuntu在完成后是无法使用root直接登录系统，需要使用这个用户来登录系统，然后自行设置root密码。
#### SSH setup 
    选择install openshh server，我们之后需要这个来远程管理服务器
#### 开始安装
    这个时候在下载安全更新，这里建议等待，如果不愿意等待你就直接重启即可
    看到这一行就完成了 ： finish：cmd-in-target：SUCCESS
    重启服务器：Reboot now
    删除掉CDROM

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

#### 更改网卡地址（如果安装时没有改网卡地址可以用这个方法）
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

#### 配置数据库
参考网址：https://confluence.atlassian.com/conf719/database-setup-for-mysql-1157467559.html

```bash
#打开配置文件目录
cd /etc/mysql/my.cnf

#编辑配置文件
my.cnf

#添加以下代码到配置文件
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_bin
default-storage-engine=INNODB
max_allowed_packet=256M
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
log_bin_trust_function_creators = 1  

#重启数据库
sudo service mysql restart
```


```bash
#配置数据库
#进入数据库
mysql -u root -p

#创建数据库
CREATE DATABASE confluence CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

#创建用户
CREATE USER 'confluenceuser'@'localhost' IDENTIFIED BY 'Qwer1234';

#给用户权限
GRANT ALL PRIVILEGES ON confluence.* TO 'confluenceuser'@'localhost';

#退出
exit

#重启数据库
sudo systemctl restart mysql

#查看状态
sudo service mysql status
```

#### 安装confluence
把atlassian-confluence-7.19.20-x64.bin 拷贝到 /mnt

```bash
#执行这个命令后，所有用户（包括文件所有者、文件所属组和其他用户）都会对这个文件拥有执行权限。
chmod a+x atlassian-confluence-7.19.20-x64.bin

#开始安装
sudo ./atlassian-confluence-7.19.20-x64.bin

#查看confluence 运行状态
ps aux | grep confluence
```

#### 安装数据库驱动
把数据库驱动复制到mnt
```bash
#安装解压软件
sudo apt install alien

#解压
sudo alien -i -d mysql-connector-java_8.0.22-1ubuntu20.04_all.deb

#执行命令移动解压出来的文件
sudo cp /usr/share/java/mysql-connector-java-8.0.22.jar /opt/atlassian/confluence/confluence/WEB-INF/lib
```

#### 安装代理软件
代理软件地址：https://github.com/haxqer/confluence
把文件放到/opt/atlassian/agent/
打开：/opt/atlassian/confluence/bin/setenv.sh

```bash
#插到这个setenv.sh文件的最后部分，export CATALINA_OPTS 的上面 
CATALINA_OPTS="-javaagent:/opt/atlassian/agent/atlassian-agent.jar ${CATALINA_OPTS}"

#重启
sudo /etc/init.d/confluence restart

#验证是否启动成功
ps aux | grep javaagent

#获取秘钥
java -jar /opt/atlassian/agent/atlassian-agent.jar  -p conf -m  12345678@qq.com -n confluence1 -o confluence2 -s BQK1-NYKM-0CGT-V5KR(你的SN序列号)

#破解插件
java -jar /opt/atlassian/agent/atlassian-agent.jar  -p com.stiltsoft.confluence.smart-attachment-for-confluence -m  12345678@qq.com -n confluence1 -o confluence2 -s B18L-VALX-ETN1-20QQ(你的SN序列号)
```

#### 连接数据库Enjoy
  | 项目   | 参数   |
  |-------|-------|
  |主机名|localhost|
  |端口|3306|
  |数据库名称|confluence|
  |用户名|confluenceuser|
  |密码|Qwer1234|
 
