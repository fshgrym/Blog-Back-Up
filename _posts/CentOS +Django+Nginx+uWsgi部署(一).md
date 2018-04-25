---
title: CentOS +Django+Nginx+uWsgi部署(一)
date: 2018-4-15 19:08:32
categories:
tags:
     - Python
     - Django
     - Nginx
---
> 部署环境

```
CentOS 7
Python 3.6
```

> 新建用户

如果你是一台全新的服务器的话。通常我们使用root用户登录，但是root下部署代码是不太安全的。建议新建一个普通用户，以下命令将创建一个拥有超级权限的新用户。

```
adduser fsh #添加用户
passwd fsh  #设置密码
```
接下来给新建的用户进行root授权，普通用户在home下才具备完整的权限，访问其他目录是需要授权的。
<!--more-->
`sudo`命令的授权管理在sudoers文件。怎么找到这个文件呢？当然是有办法的，使用whereis来查找。

```
[root@VM_0_12_centos ~]# whereis sudoers
sudoers: /etc/sudoers /etc/sudoers.d /usr/libexec/sudoers.so /usr/share/man/man5/sudoers.5.gz
```

查看sudoers文件权限

```
[root@VM_0_12_centos ~]# ls -l /etc/sudoers
-r--r----- 1 root root 3907 Jun 23  2017 /etc/sudoers
```

`r`表示的是只读，而我们需要更改文件配置。需要添加写入权限

```
[root@localhost ~]# chmod -v u+w /etc/sudoers
mode of "/etc/sudoers" changed from 0440 (r--r-----) to 0640 (rw-r-----)
```

打开sudoers文件,找到root这里添加我们新建的用户后保存。

```
[root@localhost ~]# vim /etc/sudoers
## Allow root to run any commands anywher  
root    ALL=(ALL)       ALL  
fsh  ALL=(ALL)       ALL  #fsh就是刚刚新增的用户
```

vim 简单使用

按`i`写入，退出按一下`esc` 输入`:` 再输入`wq`保存。

不保存退出按`q`退出，`！`强制退出。其他使用命令在网上都能查到。

保存之后需要的刚刚的写入权限收回

```
[root@VM_0_12_centos ~]# chmod -v u-w /etc/sudoers
mode of ‘/etc/sudoers’ changed from 0640 (rw-r-----) to 0440 (r--r-----)
```

切换用户`su 用户名`

> 安装软件

这里我yum源更换为国内源

```
​```mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup```#备份
```

下载新的CentOS-Base.repo 到/etc/yum.repos.d/，对应版本如下

```
CentOS 5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

生成缓存

```
yum makecache
```

> 安装python3.6

安装依赖包

```
yum groupinstall "Development tools"

yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel
```

下载源码

```
wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz
```

编译

```
wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz
tar -xzvf Python-3.6.2.tgz -C  /tmp
cd  /tmp/Python-3.6.2/
./configure --prefix=/usr/local
make
make altinstall
```

加入软连接

```
ln -s /usr/local/bin/python3.6 /usr/bin/python3
ln -s /usr/local//bin/pip3.6 /usr/bin/pip3
```

虚拟环境安装

```
pip3 install virtualenvwrapper
```

打开~/.bashrc文件，加入以下配置。。

```
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```

重载~/.bashrc文件

```
source ~/.bashrc
```

新建环境

```
mkvirtualenv blog
```

进入环境

```
workon blog
```

导出本地虚拟环境的包

```
pip freeze > requirements.txt #导出
pip install -r requirements.txt #安装
```

好了，到这里如果没有什么错误就可以继续下面的步骤。

uwsgi安装

```
#需要在虚拟环境下安装
pip install uwsgi
```
Nginx安装

```
sudo yum install epel-release
sudo yum install nginx
```

启动

```
sudo systemctl start nginx
```

如果系统开启了防火墙，需要开放http和https通信

```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

设置为开机自启

```
sudo systemctl enable nginx
```

访问方式

```
http://服务器ip/
```

maridb(mysql)安装

```
 sudo yum install mariadb-devel
 sudo yum install mariadb-server
```

启动重启

```
sudo systemctl start mariadb
sudo systemctl restart mariadb
```

设置bind-ip

```
vim /etc/my.cnf
    在 [mysqld]:
        下面加一行
        bind-address = 0.0.0.0
```

设置外部ip可以访问，先进入mysql才能运行下面命令

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
FLUSH PRIVILEGES
```

阿里云服务器需要开放端口默认3306，到这里环境搭建就结束了。部署请看下一篇！
