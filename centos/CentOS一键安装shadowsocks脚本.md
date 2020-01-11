Shadowsocks是一个基于python的轻量级socks代理软件,可以在任何系统简单的实现访问被屏蔽的网站。网友也常称为科学上网，简称ss,在此分享与记录CentOS一键安装shadowsocks脚本。
一、使用root用户登录，运行以下命令：

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

二、安装完成后，脚本提示如下：

```
Congratulations, shadowsocks install completed!
Your Server IP:your_server_ip
Your Server Port:8989
Your Password:your_password
Your Local IP:127.0.0.1
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:http://teddysun.com/342.html
Enjoy it!
```
三、卸载方法

```
./shadowsocks.sh uninstall
```
四、配置文件

配置文件路径为：/etc/shadowsocks.json

单用户配置：

```
{
    "server":"your_server_ip",
    "server_port":8989,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"yourpassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
多用户配置：

```
{
    "server":"your_server_ip",
    "local_address": "127.0.0.1",
    "local_port":1080,
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
五、相关使用命令
```

#启动
/etc/init.d/shadowsocks start
#停止
/etc/init.d/shadowsocks stop
#重启
/etc/init.d/shadowsocks restart
#状态
/etc/init.d/shadowsocks status
```
