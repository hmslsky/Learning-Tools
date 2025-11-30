# 文件共享服务器搭建

## 一、NFS/Samba

> Network File System(NFS) is a distributed file system protocol originally developed by Sun Microsystems in 1984.

```
# 安装
yum install samba samba-common
# 修改配置
vim /etc/samba/smb.conf
# 挂载smb磁盘
smbclient -L //127.0.0.1 -U vbrug
# 测试参数是否有问题
testparam

```

### 1.1 软件安装

yum install nfs-utils

## 二、Seafile

### 2.1 docker-compose安装

```yaml
version: '2.0'
services:
  db:
    image: mariadb:10.5
    container_name: seafile-mysql
    environment:
      - MYSQL_ROOT_PASSWORD={password}  # Requested, set the root's password of MySQL service.
      - MYSQL_LOG_CONSOLE=true
    volumes:
      - /data/seafile-mysql/db:/var/lib/mysql  # Requested, specifies the path to MySQL data persistent store.
      - /etc/localtime:/etc/localtime
    networks:
      - seafile-net

  memcached:
    image: memcached:1.6
    container_name: seafile-memcached
    entrypoint: memcached -m 256
    networks:
      - seafile-net

  seafile:
    image: seafileltd/seafile-mc:latest
    container_name: seafile
    ports:
      - "8118:80"
      - "8080:8080"
#      - "443:443"  # If https is enabled, cancel the comment.
    volumes:
      - /data/seafile-data:/shared   # Requested, specifies the path to Seafile data persistent store.
      - /etc/localtime:/etc/localtime
    environment:
      - DB_HOST=db
      - DB_ROOT_PASSWD=[mysql_password]  # Requested, the value shuold be root's password of MySQL service.
      - TIME_ZONE=Asia/Shanghai # Optional, default is UTC. Should be uncomment and set to your local time zone.
      - SEAFILE_ADMIN_EMAIL=[admin_email] # Specifies Seafile admin user, default is 'me@example.com'.
      - SEAFILE_ADMIN_PASSWORD=[admin_password]     # Specifies Seafile admin password, default is 'asecret'.
      - SEAFILE_SERVER_LETSENCRYPT=false   # Whether use letsencrypt to generate cert.
      - SEAFILE_SERVER_HOSTNAME=vbrugsky.com # Specifies your host name.  【hostname必须为域名格式，否则会出问题】
    depends_on:
      - db
      - memcached
    networks:
      - seafile-net

networks:
  seafile-net:

```

### 2.2 onlyoffice搭建配置

#### onlyoffice安装自动保存配置

```bash
# onlyoffice安装
docker pull onlyoffice/documentserver:latest
docker run -dit -p 8117:80 --name onlyoffice --restart=always onlyoffice/documentserver:latest
# onlyoffice自动保存修改
vi /etc/onlyoffice/documentserver/local.json
# 增加以下配置
{

    "services": {
        "CoAuthoring": {
            "autoAssembly": {
                "enable": true,
                "interval": "5m"
            }
        }
    }
}

```

#### seafile配置调用

```
# 添加以下配置信息
vi ./data/conf/seahub_settings.py
# Enable Only Office
ENABLE_ONLYOFFICE = True
VERIFY_ONLYOFFICE_CERTIFICATE = False
ONLYOFFICE_APIJS_URL = 'http{s}://{your OnlyOffice server's domain or IP}/web-apps/apps/api/documents/api.js'
ONLYOFFICE_FILE_EXTENSION = ('doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx', 'odt', 'fodt', 'odp', 'fodp', 'ods', 'fods')
ONLYOFFICE_EDIT_FILE_EXTENSION = ('docx', 'pptx', 'xlsx')

```

### 2.3 webdav开启使用

##### SeaDAV配置文件：./data/conf/seafdav.conf，如果没有可以自行创建

```
[WEBDAV]
# Default is false. Change it to true to enable SeafDAV server.
enabled = true
port = 8080
# If you deploy seafdav behind nginx/apache, you need to modify "share_name".
# 挂载时可增加子级目录
share_name = /

```

#### webdav使用挂载，挂载时可以增加子级目录

```
# 执行以下命令，然后帐号密码
sudo mount -t davfs -o uid=vbrug http://IP:8080/wiki/ /netdisk

```

## 三、NextCloud安装

### 3.1 CentOS8基础环境安装

```bash
# 基础lib
dnf install -y epel-release yum-utils unzip curl wget \
bash-completion policycoreutils-python-utils mlocate bzip2
# 更新系统
dnf update -y

```

### 3.2 Apache安装

```bash
# 包管理器直接安装，centos8默认2.4版本
dnf install httpd
# 编辑nextcloud配置
vi /etc/httpd/conf.d/nextcloud.conf
<<comment    增加以下内容
<VirtualHost *:80>
  DocumentRoot /var/www/html/nextcloud/
  ServerName  your.server.com

  <Directory /var/www/html/nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>

  </Directory>
</VirtualHost>
comment>>
systemctl enable httpd.service
systemctl start httpd.service

```

### 3.3 PHP安装

```bash
##-----------------------------------------
## CentOS 8 系统安装php 8.0
##-----------------------------------------
dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
dnf -y install yum-utils
dnf module reset php
dnf module install php:remi-8.0 -y
dnf install php -y
dnf -y install php-{cli,fpm,mysqlnd,zip,devel,gd,mbstring,curl,xml,pear,bcmath,json}

##-----------------------------------------
## CentOS 7 系统安装php 8.0
##-----------------------------------------
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum -y install yum-utils
yum-config-manager --disable 'remi-php*'
yum-config-manager --enable remi-php80
yum -y install php php-{cli,fpm,mysqlnd,zip,devel,gd,mbstring,curl,xml,pear,bcmath,json}

##-----------------------------------------
## 安装phh模块
##-----------------------------------------
dnf install -y php php-gd php-mbstring php-intl php-pecl-apcu\
     php-mysqlnd php-opcache php-json php-zip
dnf install -y php-redis php-imagick

```

### 3.4 Database安装

可使用sqllite、mysql、mariaDB，sqlite适合测试环境使用

```bash


```

### 3.5 Redis安装

### 3.6 nextcloud安装


