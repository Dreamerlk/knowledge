1：下载扩展包源码地址  https://github.com/TarsPHP/tars-extension   
~~~
git clone https://github.com/TarsPHP/tars-extension.git
~~~
2：进入源码包  
~~~
cd tars-extension
~~~
3：运行phpize命令，写全phpize的路径    
~~~
/usr/local/php/bin/phpize
~~~
4：运行configure命令，配置时 要将php-config的路径附上
~~~
./configure --with-php-config=/usr/local/php/bin/php-config
~~~
5：编译
~~~
make
make test
make install
~~~
6:修改php.ini 
 ~~~
extension=phptars.so
~~~
9:重启对应的php-fpm

