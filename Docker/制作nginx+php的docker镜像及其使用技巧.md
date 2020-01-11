一、准备工作：

使用工具：

　　1.主流版本的docker，本人使用的是 docker 1.91 版本

　　2.centos的官方docker镜像作为基础镜像

　　3.nginx-1.9.12；php-5.5.34；supervisor

思路：

　　众所周知，docker镜像的制作有2种方法，一种是启动一个容器并在容器里操作，再将容器提交为一个新的镜像；一种是写Dockerfile，然后执行dockerfile由docker给我们一步步自动生成新的镜像；显然第二种方法更高大上，也更适合容器需要不断版本更替的场景。本人在安装nginx和php的时候，更习惯自己下载源码编译安装，所以编译安装这里写Dockerfile实在是繁琐，而且nginx+php并不是需要频发更替版本，通常在制作容器前，跟开发确定好版本号，制作好容器可以一直使用；所以以下我的操作里，前半部分，nginx和php的安装，我会在容器里操作；最后让nginx和php同时启动起来，我则是写了一个Dockerfile。

二、制作容器

1、启动一个centos容器作为基础镜像

　　docker pull centos 

　　docker run -it --name nginx centos bash

2、这样就创建了一个以centos的官方镜像为基础的容器，并进如容器；在容器里用yum安装wget命令和编译安装需要的命令，更新国内yum源，下载Nginx，php源码

　　yum install -y wget gcc gcc-c++ make openssl-devel

　　wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

　　wget http://nginx.org/download/nginx-1.9.12.tar.gz

　　wget https://sourceforge.net/projects/pcre/files/pcre/8.37/pcre-8.37.tar.gz
       ps：所有pcre版本文件（https://sourceforge.net/projects/pcre/files/pcre）

　　wget http://cn2.php.net/distributions/php-5.5.34.tar.gz

3、更新yum源

　　yum update

4、复制源码包到工作目录下

　　mv *.gz /usr/local/src

　　cd /usr/local/src/

5、解压源码包后并删除，建议删除，删除的目的是不要让最后的镜像过于的大；tar自带参数，解压同时删除，忘记了。。

　　tar xf nginx-1.9.12.tar.gz

　　tar xf pcre-8.37.tar.gz

　　tar xf php-5.5.34.tar.gz

　　rm -f nginx-1.9.12.tar.gz pcre-8.37.tar.gz php-5.5.34.tar.gz

6、编译安装nginx：

　　1）创建nginx用户

　　　　groupadd -r nginx

　　　　useradd -r -g nginx nginx

　　2）编译安装nginx

　　　　cd nginx-1.9.12/

./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module --with-pcre=/usr/local/src/pcre-8.37
make && make install
echo "daemon off;" >> /usr/local/nginx/conf/nginx.conf    #在nginx的配置文件里加上这一行很关键，这样nginx可以在docker启动的时候在后台运行！

7、编译安装php

　　1）准备php的依赖包

　　　　yum install -y bison bison-devel zlib-devel libmcrypt-devel mcrypt mhash-devel libxml2-devel libcurl-devel bzip2-devel readline-devel libedit-devel sqlite-devel 
yum -y install libjpeg-devel
yum -y install libpng-devel
yum -y install freetype-devel
yum install libc-client-devel 

　　2）编译安装php，如果过程中报错，提示缺少什么安装包，就用yum安装。ps:可参考文章（https://www.jianshu.com/p/d3d3ede407ce）

　　　　cd php-5.5.34/

　　　　./configure --prefix=/usr/local/php --with-zlib-dir --with-freetype-dir --enable-mbstring --with-libxml-dir=/usr/local/libxml --enable-soap --enable-calendar --with-curl --with-mcrypt --with-zlib --with-gd  --disable-rpath --enable-inline-optimization --with-bz2 --with-zlib --enable-sockets --enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex --enable-exif --enable-bcmath --with-mhash --enable-zip --with-pcre-regex --with-mysql --with-pdo-mysql --with-mysqli --with-jpeg-dir=/usr/local/libjpeg --with-png-dir=/usr/local/libpng --enable-gd-native-ttf --with-openssl --with-fpm-user=www --with-fpm-group=www --with-libdir=lib64 --enable-ftp --with-imap --with-imap-ssl --with-kerberos --with-gettext --with-xmlrpc --with-xsl --enable-opcache --enable-fpm --enable-xml --enable-shmop --enable-session --enable-ctype --with-iconv-dir --with-iconv

　　　　make && make install

　　3）如果编译安装过程中报错，按照报错提示的去用yum解决依赖关系；如果当前的yum源解决不了，那么可以试试：

　　　　wget http://www.atomicorp.com/installers/atomic

　　　　chmod +x atomic

　　　　./atomic 

　　　　yum install -y XXX XXX

　　4）准备php配置文件

　　　　cp php.ini-production /etc/php.ini

　　　　cd /usr/local/php/etc

　　　　cp php-fpm.conf.default php-fpm.conf

　　5）修改php配置文件，跟nginx里加一行的效果一样，为了启动docker时，php可以在后台运行

　　　　;daemonize = yes的注释去掉，并把yes改为no

　　6）安装php扩展，php的扩展很多，安装方法也都大同小异，一下以memcached扩展为例

　　　　wget https://pecl.php.net/get/memcache-2.2.7.tgz

　　　　tar xf memcache-2.2.7.tgz

　　　　cd memcache-2.2.7

　　　　/usr/local/php/bin/phpize

　　　　./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config --with-zlib-dir

　　　　在php.ini里添加一行extension=/usr/local/php/lib/php/extensions/memcache.so

　　7）测试php-fpm启动

　　　　/usr/local/php/sbin/php-fpm -c /etc/php.ini -y /usr/local/php/etc/php-fpm.conf

　　　　ps -ef | grep php-fpm

8、整合nginx跟php

　　修改nginx.conf；这个可以参考各种网上的资料；下面会给一个例子

　　/usr/local/nginx/sbin/nginx -t     #检查没配置文件

　　/usr/local/nginx/sbin/nginx         #启动nginx

9、整理，删除，清理yum缓存，退出容器

　　cd /usr/local/src/

　　rm -fr *

　　make clean

　　yum clean all

　　exit

10、提交容器

　　docker commit -m “nginx-php” nginx Tom/nginx:v1

 

　　到此，容器基本就已经制作完成了，接下来就是最重要的地方了，docker奉行的是一个容器跑一个进程的思想，所以启动容器的时候一般也只能启动一个进程或者一个脚本；而nginx跟php要能同时工作，需要再在此基础上做些工作！

　　一般有2个方法，一种是写脚本，但是我没有成功。。所以我用了supervisor，一个可以管理进程的工具。接下来我会使用Dockefile完成最后的工作

三、让这个镜像可以跑起来！

1、Dockefile如下：在宿主机下创建一个nginx目录，并到目录下vim Dockefile

　　FROM Tom/nginx:v1

　　# Install supervisor
　　RUN yum install -y python-setuptools
　　RUN easy_install supervisor
　　ADD supervisor.conf /etc/supervisord.conf
　　EXPOSE 80 443
　　CMD ["/usr/bin/supervisord"]

#如果出错换成这个：
 FROM Tom/nginx:v1
　　# Install supervisor
　　RUN yum install -y supervisor
　　ADD supervisor.conf /etc/supervisord.conf
　　EXPOSE 80 443
　　CMD ["/usr/bin/supervisord"]

2、其中supervisor.conf内容为：

　　[supervisord]
　　nodaemon=true

　　[program:nginx]
　　command=/usr/local/nginx/sbin/nginx

　　[program:php-fpm]
　　command=/usr/local/php/sbin/php-fpm -c /etc/php.ini -y /usr/local/php/etc/php-fpm.conf

3、运行Dockerfile

　　docker build -t Tom/nginx-php .

　　到这里，这个镜像就完成了，可以简单的测试一下：

　　docker run -d --name nginx-php -p 80:80 Tom/nginx-php

　　然后用命令docker ps -a 查看下这个容器是否正常启动，如果有问题，可以docker logs -f nginx-php 查看下这个容器启动在哪里出了问题。

三、nginx-php容器的使用技巧

1、创建几个新的目录

　　mkdir /data/nginx/{log,php.conf,data,conf} -p

　　其中log目录我打算把nginx的日志映射到这个目录下，php.cof目录我打算把php的配置文件映射到这个目录下，data目录我打算把网页文件映射到这个目录下，conf我打算把nginx的配置文件映射到这个目录下

2、nginx.conf示例

　　worker_processes 1;

　　events {
　　　　worker_connections 1024;
　　}

　　http {

　　　　server {
　　　　　　listen 80 default_server ;
　　　　　　server_name test.lala.com ;

　　　　　　location / {
　　　　　　　　root /usr/share/nginx/web;
　　　　　　　　index index.html index.htm index.php api/login.php;
　　　　　　}

　　　　　　error_page 500 502 503 504 /50x.html;
　　　　　　location = /50x.html {
　　　　　　　　root /usr/share/nginx/web;
　　　　　　}

　　　　　　location ~ \.php$ {
　　　　　　　　root html;
　　　　　　　　fastcgi_pass 127.0.0.1:9000;
　　　　　　　　fastcgi_index index.php;
　　　　　　　　fastcgi_param SCRIPT_FILENAME /usr/share/nginx/web/$fastcgi_script_name;
　　　　　　　　include fastcgi_params;
　　　　　　}

　　　　}

　　}

　　daemon off;

3、在/data/nginx/conf下准备好nginx.conf 在/data/nginx/php.conf 目录下准备好php.ini和php-fpm.conf ；之后启动容器的时候可以用命令：

　　docker run -d --name nginx-php -v /etc/localtime:/etc/localtime:ro --restart=always -p 80:80 -v /data/nginx/log:/usr/local/nginx/logs/ -v /data/nginx/php.conf/php.ini:/etc/php.ini -v /data/nginx/php.conf/php-fpm.conf:/usr/local/php/etc/php-fpm.conf -v /data/nginx/data:/usr/share/nginx/web -v /data/nginx/conf:/usr/local/nginx/conf/  Tom/nginx-php

4、更新nginx下的web文件，直接更新宿主机上/data/nginx/data/目录下的文件

5、如果要修改nginx的配置文件，直接在宿主机上的/data/nginx/conf目录下修改nginx.conf ；修改完成后，你可以使用下面命令：

　　docker exec nginx-php /usr/local/nginx/sbin/nginx -t                             　#检查配置文件是否正确

　　docker exec nginx-php /usr/local/nginx/sbin/nginx -s reload　　　　　　　　#让容器里的nginx重新读取nginx配置文件

6、如果要修改php的配置文件，直接在宿主机上的/data/nginx/php.conf目录下修改php.ini或者修改php-fpm.conf ；修改完成后要重启容器才能生效

　　docker restart nginx-php

7、容器里的nginx日志输出映射到了宿主机上的/data/nginx/log目录下
