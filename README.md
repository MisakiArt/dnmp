### **直接下载, 不要git clone, 以防sh文件的换行符变成“\r\n”**
# 目录
- [1.目录结构](#1目录结构)
- [2.快速使用](#2快速使用)
- [3.PHP和扩展](#3PHP和扩展)
    - [3.1 切换Nginx使用的PHP版本](#31-切换Nginx使用的PHP版本)
    - [3.2 安装PHP扩展](#32-安装PHP扩展)
    - [3.3 快速安装php扩展](#33-快速安装php扩展)
    - [3.4 Host中使用php命令行（php-cli）](#34-host中使用php命令行php-cli)
    - [3.5 使用composer](#35-使用composer)
- [4.管理命令](#4管理命令)
    - [4.1 服务器启动和构建命令](#41-服务器启动和构建命令)
    - [4.2 添加快捷命令](#42-添加快捷命令)
- [5.使用Log](#5使用log)
    - [5.1 Nginx日志](#51-nginx日志)
    - [5.2 PHP-FPM日志](#52-php-fpm日志)


## 1.目录结构

```
/
├── data                        数据库数据目录
│   └── mysql5                  MySQL5 数据目录
├── services                    服务构建文件和配置文件目录
│   ├── mysql5                  MySQL5 配置文件目录
│   ├── nginx                   Nginx 配置文件目录
│   └── php                     PHP7.1.33 配置目录
├── logs                        日志目录
├── docker-compose.sample.yml   Docker 服务配置示例文件
├── env.smaple                  环境配置示例文件
└── www                         PHP 代码目录
```

## 2.快速使用
1. 本地安装
    - `git`
    - `Docker`(系统需为Linux，Windows 10 Build 15063+，或MacOS 10.12+，且必须要`64`位）
    - `docker-compose 1.7.0+`
    - `mysql, redis, es 等服务连接dev环境的服务器`
    - `docker按装教程:`[windows](https://www.runoob.com/docker/windows-docker-install.html) [Linux](https://www.runoob.com/docker/centos-docker-install.html) [mac](https://www.runoob.com/docker/macos-docker-install.html)
2. `clone`项目：
    ```
    $ 直接下载, 不要git clone, 以防sh文件的换行符变成“\r\n”
    ```
3. 如果不是`root`用户，还需将当前用户加入`docker`用户组：
    ```
    $ sudo gpasswd -a ${USER} docker
    ```
4. 拷贝并命名配置文件（Windows系统请用`copy`命令），启动：
    ```
    $ cd dnmp                                           # 进入项目目录
    $ cp env.sample .env                                # 复制环境变量文件
    $ cp docker-compose.sample.yml docker-compose.yml   # 复制 docker-compose 配置文件。默认启动2个服务：
                                                        # Nginx、PHP7
    $ docker-compose up                                 # 启动
    ```
5. 在浏览器中访问：`http://localhost`或`https://localhost`(自签名HTTPS演示)就能看到效果，PHP代码在文件`./www/localhost/index.php`。

## 3.PHP和扩展
### 3.1 切换Nginx使用的PHP版本
首先，需要启动其他版本的PHP，比如PHP5.6，那就先在`docker-compose.yml`文件中删除PHP5.6前面的注释，再启动PHP5.6容器。

PHP5.6启动后，打开Nginx 配置，修改`fastcgi_pass`的主机地址，由`php`改为`php56`，如下：
```
    fastcgi_pass   php:9000;
```
为：
```
    fastcgi_pass   php56:9000;
```
其中 `php` 和 `php56` 是`docker-compose.yml`文件中服务器的名称。

最后，**重启 Nginx** 生效。
```bash
$ docker exec -it nginx nginx -s reload
```
这里两个`nginx`，第一个是容器名，第二个是容器中的`nginx`程序。


### 3.2 安装PHP扩展
PHP的很多功能都是通过扩展实现，而安装扩展是一个略费时间的过程，
所以，除PHP内置扩展外，在`env.sample`文件中我们仅默认安装少量扩展，
如果要安装更多扩展，请打开你的`.env`文件修改如下的PHP配置，
增加需要的PHP扩展：
```bash
PHP_EXTENSIONS=pdo_mysql,opcache,redis       # PHP 要安装的扩展列表，英文逗号隔开
PHP56_EXTENSIONS=opcache,redis                 # PHP 5.6要安装的扩展列表，英文逗号隔开
```
然后重新build PHP镜像。
```bash
docker-compose build php
```
可用的扩展请看同文件的`env.sample`注释块说明。

### 3.3 快速安装php扩展
1.进入容器:

```sh
docker exec -it php /bin/sh

install-php-extensions apcu 
```
2.支持快速安装扩展列表
<!-- START OF EXTENSIONS TABLE -->
<!-- ########################################################### -->
<!-- #                                                         # -->
<!-- #  DO NOT EDIT THIS TABLE: IT IS GENERATED AUTOMATICALLY  # -->
<!-- #                                                         # -->
<!-- #  EDIT THE data/supported-extensions FILE INSTEAD        # -->
<!-- #                                                         # -->
<!-- ########################################################### -->
| Extension | PHP 5.5 | PHP 5.6 | PHP 7.0 | PHP 7.1 | PHP 7.2 | PHP 7.3 | PHP 7.4 | PHP 8.0 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| amqp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| apcu | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| apcu_bc |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| bcmath | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| bz2 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| calendar | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| cmark |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| dba | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| decimal |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| enchant[*](#special-requirements-for-enchant) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ev | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| exif | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ffi |  |  |  |  |  |  | ✓ | ✓ |
| gd | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gettext | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gmagick | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| gmp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gnupg | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| grpc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| http | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| igbinary | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| imagick | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| imap | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| interbase | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| intl | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ioncube_loader | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| ldap | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mailparse | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| maxminddb |  |  |  |  | ✓ | ✓ | ✓ | ✓ |
| mcrypt | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| memcache | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| memcached | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mongo | ✓ | ✓ |  |  |  |  |  |  |
| mongodb | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mosquitto | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| msgpack | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mssql | ✓ | ✓ |  |  |  |  |  |  |
| mysql | ✓ | ✓ |  |  |  |  |  |  |
| mysqli | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| oauth | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| oci8 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| odbc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| opcache | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| opencensus |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| parallel[*](#special-requirements-for-parallel) |  |  |  | ✓ | ✓ | ✓ | ✓ |  |
| pcntl | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pcov |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_dblib | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_firebird | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_mysql | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_oci |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_odbc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_pgsql | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_sqlsrv |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| pgsql | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| propro | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| protobuf | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pspell | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pthreads[*](#special-requirements-for-pthreads) | ✓ | ✓ | ✓ |  |  |  |  |  |
| raphf | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| rdkafka | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| recode | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| redis | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| shmop | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| smbclient | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| snmp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| snuffleupagus |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| soap | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sockets | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| solr | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sqlsrv |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| ssh2 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| swoole | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sybase_ct | ✓ | ✓ |  |  |  |  |  |  |
| sysvmsg | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sysvsem | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sysvshm | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| tdlib[*](#special-requirements-for-tdlib) |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| tidy | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| timezonedb | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| uopz | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| uuid | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| wddx | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| xdebug | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xhprof | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xlswriter |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xmlrpc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| xsl | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| yaml | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| yar | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| zip | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| zookeeper | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |

此扩展来自[https://github.com/mlocati/docker-php-extension-installer](https://github.com/mlocati/docker-php-extension-installer)
参考示例文件

### 3.4 Host中使用php命令行（php-cli）

1. 参考[bash.alias.sample](bash.alias.sample)示例文件，将对应 php cli 函数拷贝到主机的 `~/.bashrc`文件。
2. 让文件起效：
    ```bash
    source ~/.bashrc
    ```
3. 然后就可以在主机中执行php命令了：
    ```bash
    ~ php -v
    PHP 7.1.13 (cli) (built: Dec 21 2018 02:22:47) ( NTS )
    Copyright (c) 1997-2018 The PHP Group
    Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
        with Zend OPcache v7.2.13, Copyright (c) 1999-2018, by Zend Technologies
        with Xdebug v2.6.1, Copyright (c) 2002-2018, by Derick Rethans
    ```
### 3.5 使用composer
**方法1：主机中使用composer命令**
1. 确定composer缓存的路径。比如，我的dnmp下载在`~/dnmp`目录，那composer的缓存路径就是`~/dnmp/data/composer`。
2. 参考[bash.alias.sample](bash.alias.sample)示例文件，将对应 php composer 函数拷贝到主机的 `~/.bashrc`文件。
    > 这里需要注意的是，示例文件中的`~/dnmp/data/composer`目录需是第一步确定的目录。
3. 让文件起效：
    ```bash
    source ~/.bashrc
    ```
4. 在主机的任何目录下就能用composer了：
    ```bash
    cd ~/dnmp/www/
    composer create-project yeszao/fastphp project --no-dev
    ```
5. （可选）第一次使用 composer 会在 `~/dnmp/data/composer` 目录下生成一个**config.json**文件，可以在这个文件中指定国内仓库，例如：
    ```json
    {
        "config": {},
        "repositories": {
            "packagist": {
                "type": "composer",
                "url": "https://packagist.laravel-china.org"
            }
        }
    }

    ```
**方法二：容器内使用composer命令**

还有另外一种方式，就是进入容器，再执行`composer`命令，以PHP7容器为例：
```bash
docker exec -it php /bin/sh
cd /www/localhost
composer update
```
    
## 4.管理命令
### 4.1 服务器启动和构建命令
如需管理服务，请在命令后面加上服务器名称，例如：
```bash
$ docker-compose up                         # 创建并且启动所有容器
$ docker-compose up -d                      # 创建并且后台运行方式启动所有容器
$ docker-compose up nginx php mysql         # 创建并且启动nginx、php、mysql的多个容器
$ docker-compose up -d nginx php  mysql     # 创建并且已后台运行的方式启动nginx、php、mysql容器


$ docker-compose start php                  # 启动服务
$ docker-compose stop php                   # 停止服务
$ docker-compose restart php                # 重启服务
$ docker-compose build php                  # 构建或者重新构建服务

$ docker-compose rm php                     # 删除并且停止php容器
$ docker-compose down                       # 停止并删除容器，网络，图像和挂载卷
```

### 4.2 添加快捷命令
在开发的时候，我们可能经常使用`docker exec -it`进入到容器中，把常用的做成命令别名是个省事的方法。

首先，在主机中查看可用的容器：
```bash
$ docker ps           # 查看所有运行中的容器
$ docker ps -a        # 所有容器
```
输出的`NAMES`那一列就是容器的名称，如果使用默认配置，那么名称就是`nginx`、`php`、`php56`、`mysql`等。

然后，打开`~/.bashrc`或者`~/.zshrc`文件，加上：
```bash
alias dnginx='docker exec -it nginx /bin/sh'
alias dphp='docker exec -it php /bin/sh'
alias dphp56='docker exec -it php56 /bin/sh'
alias dmysql='docker exec -it mysql /bin/bash'
```
下次进入容器就非常快捷了，如进入php容器：
```bash
$ dphp
```

### 4.3 查看docker网络
```sh
ifconfig docker0
```
用于填写`extra_hosts`容器访问宿主机的`hosts`地址

## 5.使用Log

Log文件生成的位置依赖于conf下各log配置的值。

### 5.1 Nginx日志
Nginx日志是我们用得最多的日志，所以我们单独放在根目录`log`下。

`log`会目录映射Nginx容器的`/var/log/nginx`目录，所以在Nginx配置文件中，需要输出log的位置，我们需要配置到`/var/log/nginx`目录，如：
```
error_log  /var/log/nginx/nginx.localhost.error.log  warn;
```


### 5.2 PHP-FPM日志
大部分情况下，PHP-FPM的日志都会输出到Nginx的日志中，所以不需要额外配置。

另外，建议直接在PHP中打开错误日志：
```php
error_reporting(E_ALL);
ini_set('error_reporting', 'on');
ini_set('display_errors', 'on');
```

如果确实需要，可按一下步骤开启（在容器中）。

1. 进入容器，创建日志文件并修改权限：
    ```bash
    $ docker exec -it php /bin/sh
    $ mkdir /var/log/php
    $ cd /var/log/php
    $ touch php-fpm.error.log
    $ chmod a+w php-fpm.error.log
    ```
2. 主机上打开并修改PHP-FPM的配置文件`conf/php-fpm.conf`，找到如下一行，删除注释，并改值为：
    ```
    php_admin_value[error_log] = /var/log/php/php-fpm.error.log
    ```
3. 重启PHP-FPM容器。



