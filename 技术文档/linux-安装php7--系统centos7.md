```
wget  http://cn2.php.NET/distributions/php-7.0.4.tar.gz
tar zxvf php-7.0.4.tar.gz
cd  php-7.0.4</pre>
```
注：上面的php版本可以替换成最新的版本

 编译：
```
./configure --enable-fpm --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-fpm-user=nginx --with-fpm-group=nginx --with-bz2 --with-curl --with-gd --with-mcrypt --with-openssl --with-mhash --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir=/usr/local/libiconv --with-gettext --with-libxml-dir --with-zlib --with-xmlrpc --with-pcre-regex --with-pear --with-pdo-mysql=mysqlnd --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-libdir=lib64 --enable-dom --enable-xml --enable-fpm --enable-bcmath --enable-ftp --enable-sockets --disable-ipv6 --enable-mbregex --enable-mbstring --enable-calendar --enable-gd-native-ttf --enable-static --enable-fpm --enable-bcmath --enable-libxml --enable-inline-optimization --enable-mbregex --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-zip
```


如果出现下列错误。。那就是没有编译环境 。 请安装gcc

![image](http://upload-images.jianshu.io/upload_images/6954572-ddb61932f5c960c0..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

yum install gcc

之后看最后又遇到这个问题：

![image](http://upload-images.jianshu.io/upload_images/6954572-6776f648d5782d94..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

yum install libxml2-devel

最后：

make && make install

安装完成。

配置  （一下配置都是我个人的路径配置 大家请按照各自路径配置）

cp  php.ini-development /usr/local/lib/php.ini 
cp sapi/fpm/init.d.php-fpm /etc/init.d/php7-fpm chmod +x /etc/init.d/php7-fpm
cd /usr/local/php/etc 
cp php-fpm.conf.default php-fpm.conf 
cp php-fpm.d/www.conf.default  php-fpm.d/www.conf

配置完毕

/etc/init.d/php7-fpm  start 启动php

注意：在安装完之后看下PHPINFO里面，php.ini的路径，然后去服务器检查试一下是否存在该文件，如果不存在就cp过来

**逐步剔除php7不支持的代码**
检测工具：[https://github.com/sstalle/php7cc](https://github.com/sstalle/php7cc)

