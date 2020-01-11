我们知道apache + php 是比较经典的搭配，但是两者都会升版，我就经历过两次php 升版。

   一般就是重新下载新版本的php ,然后编译安装。这一切都很繁琐。有没有自动安装工具呢？

   当然是有的，phpevn 就是linux 下的php 多版本管理工具。下面介绍他的安装。
phpenv安装

$ sudo yum install git
$ mkdir -p repos/git
$ cd repos/git
$ git clone https://github.com/CHH/phpenv.git
$ cd phpenv/bin
$ ./phpenv-install.sh
Installing phpenv in /path/to/.phpenv
remote: Counting objects: 1889, done.
remote: Total 1889 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (1889/1889), 297.15 KiB | 155 KiB/s, done.
Resolving deltas: 100% (1182/1182), done.
Success.

export PATH="/path/to/.phpenv/bin:$PATH"
eval "$(phpenv init -)"

Add above line at the end of your ~/.bashrc and restart your shell to use phpenv.

~/.bashrc

 
export PATH="/path/to/.phpenv/bin:$PATH"
eval "$(phpenv init -)"

$ source ~/.bashrc

php-build安装

php-build是phpenv 的一个插件

$ git clone https://github.com/CHH/php-build.git ~/.phpenv/plugins/php-build

    所有可以安装的php 版本确认（2017/6/3）

$ phpenv install --list

5.2.17
5.3.2
5.3.3
5.3.6
5.3.8
5.3.9
略
7.0.0
7.0.1
7.0.2
7.0.3
7.0.4
7.0.5
7.0.6
7.0.7
7.0.8
7.0.9
7.0.10
7.0.11
7.0.12
7.0.13
7.0.14
7.0.15
7.0.16
7.0.17
7.0.18
7.0.19
7.0snapshot
7.1.0
7.1.1
7.1.2
7.1.3
7.1.4
7.1.5
7.1snapshot
master
PHP安装

 必要的安装包安装

   $ sudo yum install gcc bison libxml2 libxml2-devel openssl-devel \ libcurl-devel libjpeg-turbo-devel libpng-devel libmcrypt-devel \ readline-devel libtidy-devel libxslt-devel

  PHP5.3.29安装

$ phpenv install 5.3.29

PHP5.4.32 安装

$ phpenv install 5.4.32

PHP5.5.16 安装

$ phpenv install 5.6.11

安装后的版本確認

$ phpenv versions 　　

5.3.29

5.4.32

5.6.11

PHP版本切换

切换到 PHP5.6.11

$ phpenv local 5.6.11

$ phpenv version 5.6.11 (set by /path/to/.php-version)
