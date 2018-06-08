# \[Linux\] LAMP环境编译安装

## 准备工作

下载下列系统及软件：

* L：[`CentOS`](https://www.centos.org/download/)
* A：[`Apache`](http://www.apache.org/dyn/closer.cgi)
* M：[`MariaDB`](https://downloads.mariadb.org/)
* P：[`PHP`](http://php.net/downloads.php)

## CentOS

> 使用最新版的[`CentOS-7-x86_64-DVD-1511`](http://centos.ustc.edu.cn/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1511.iso)。 图形界面，选择中文根据提示安装即可。 选择**最小安装**。

### 网络配置

* 进入`/etc/sysconfig/network-scripts`目录

  使用`ls`命令找到类似`ifcfg-enp0s3`的文件，并编辑

  ```text
  配置如下：
  ```

  ```bash
  #开机自动连接
  ONBOOT=yes
  #静态IP地址
  BOOTPROTO=static
  #IP
  IPADDR0=192.168.1.6
  #网关
  GATEWAY0=192.168.1.1
  #DNS
  DNS1=61.139.2.69
  ```

* 重启网络

  ```bash
  service network restart
  ```

  **关闭SELinux**

* 临时关闭

  ```bash
  setenforce 0
  ```

* 修改`/etc/selinux/config` 文件

  ```bash
  SELINUX=disabled
  ```

* 下次重启后，即为关闭状态。

### 配置防火墙

```bash
#永久开放80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
#重启服务
firewall-cmd --reload
```

### 初始软件软件

```bash
yum -y install wget perl-DBI gcc gcc-c++ unzip libxml2* libpng-devel autoconf
```

## MariaDB

> 使用最新版的`MariaDB-5.5.48-centos7-x86_64`组件： [`MariaDB-common`](http://mirrors.opencas.cn/mariadb/mariadb-5.5.48/yum/centos/7/x86_64/rpms/MariaDB-5.5.48-centos7-x86_64-common.rpm) 、[`MariaDB-client`](http://mirrors.opencas.cn/mariadb/mariadb-5.5.48/yum/centos/7/x86_64/rpms/MariaDB-5.5.48-centos7-x86_64-client.rpm)、[`MariaDB-server`](http://mirrors.opencas.cn/mariadb/mariadb-5.5.48/yum/centos/7/x86_64/rpms/MariaDB-5.5.48-centos7-x86_64-server.rpm)

* 删除旧版`mariadb-libs`

  ```bash
  yum remove mariadb-libs
  ```

* 下载`MariaDB-5.5.48-centos7-x86_64`组件：

  ```bash
  wget http://mirrors.opencas.cn/mariadb/mariadb-5.5.48/yum/centos/7/x86_64/rpms/MariaDB-5.5.48-centos7-x86_64-common.rpm
  wget http://mirrors.opencas.cn/mariadb/mariadb-5.5.48/yum/centos/7/x86_64/rpms/MariaDB-5.5.48-centos7-x86_64-client.rpm
  wget http://mirrors.opencas.cn/mariadb/mariadb-5.5.48/yum/centos/7/x86_64/rpms/MariaDB-5.5.48-centos7-x86_64-server.rpm
  ```

* 安装

  ```bash
  rpm -ivh MariaDB-5.5.48-centos7-x86_64-common.rpm
  rpm -ivh MariaDB-5.5.48-centos7-x86_64-client.rpm
  rpm -ivh MariaDB-5.5.48-centos7-x86_64-server.rpm
  ```

* 启动服务

  ```bash
  service mysql start
  ```

## Apache

### 安装目录

> `/usr/local/apache2/`

### 安装`APR`、`APR-util`、`pcre`

* 安装`APR`

  ```bash
  wget http://archive.apache.org/dist/apr/apr-1.4.5.tar.gz
  tar -zxf apr-1.4.5.tar.gz
  rm -f apr-1.4.5.tar.gz
  cd apr-1.4.5
  ./configure --prefix=/usr/local/apr
  make && make install
  ```

* 安装`APR-util`

  ```bash
  wget http://archive.apache.org/dist/apr/apr-util-1.3.12.tar.gz
  tar -zxf apr-util-1.3.12.tar.gz
  rm -f apr-util-1.3.12.tar.gz
  cd apr-util-1.3.12
  ./configure --prefix=/usr/local/apr-util -with-apr=/usr/local/apr
  make && make install
  ```

* 安装`pcre`

  ```bash
  wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.10/pcre-8.10.zip
  unzip -o pcre-8.10.zip
  rm -f pcre-8.10.zip
  cd pcre-8.10
  ./configure --prefix=/usr/local/pcre
  make && make install
  ```

### 安装`Httpd`

* 下载`httpd-2.4.20`：

  ```bash
  wget http://apache.parentingamerica.com/httpd/httpd-2.4.20.tar.gz
  ```

* 安装：

  ```bash
  tar -zxf httpd-2.4.20.tar.gz
  rm -f httpd-2.4.20.tar.gz
  cd httpd-2.4.20
  ./configure --prefix=/usr/local/apache2 --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre --enable-rewrite
  make && make install
  ```

  **注册为系统服务**

* 复制文件：

  ```bash
  cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
  ```

* 编辑`/etc/init.d/httpd`文件，在第二行加入：

  ```bash
  # chkconfig: 35 70 35
  ```

* 注册服务：

  ```bash
  chkconfig --add httpd
  chkconfig httpd on
  #启动服务
  service httpd start
  ```

## PHP

> 使用最新的[`php-7.0.5`](http://cn2.php.net/distributions/php-7.0.5.tar.gz)

### 安装目录

> `/usr/local/php/`

* 下载

  ```bash
  wget http://cn2.php.net/distributions/php-7.0.5.tar.gz
  ```

* 安装

  ```bash
  tar -zxf php-7.0.5.tar.gz
  rm -f  php-7.0.5.tar.gz
  cd php-7.0.5
  ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-gd --with-pdo-mysql --enable-soap --enable-sockets --enable-zip
  make && make install
  ```

* 加入环境变量PATH,编辑`/etc/profile`，再最后一行写入：

  ```bash
  PATH=$PATH:/usr/local/php/bin
  ```

  再执行`source /etc/profile`。

* 配置http.conf支持php

  ```bash
  LoadModule php7_module modules/libphp7.so
  DirectoryIndex index.php index.html
  AddType application/x-httpd-php .php
  ```

  **编译扩展**

> 以`zip.so`为例，进行编译安装。

进入php安装包：

```bash
cd php-7.0.5/ext/zip/
phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

然后再配置`php.ini`即可。

