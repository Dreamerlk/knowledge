如果ngxin不能使用service nginx start 开启，报Starting nginx (via systemctl):  Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details，则写入以下代码到nginx.service并放到/usr/lib/systemd/system下：

[unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/nginx
ExecReload=/usr/local/nginx/nginx -s reload
ExecStop=/usr/local/nginx/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target

以上环境在centos7，nginx版本为1.12.1

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

通常来说如果不是必须可以不用上面的方法，当在当前环境下开启或关闭遇到问题时应先用以下命令看下nginx的状态，然后根据状态来解决问题：

命令 ：systemctl status nginx

当不能使用service nginx start 开启或者使用service nginx stop 关闭时，查看systemctl status nginx 会有一定的错误提示，拿我遇到的来说有如下提示：

Jul 13 06:07:11 localhost.localdomain systemd[1]: PID file /var/run/nginx.pid not readable (yet?) after start.
这个提示是说在/etc/init.d/nginx这个文件中pidfile所指定的路径没有可读权限，当打开这个文件发现这行是被#所注释的，但想要解决这个问题还是需要把这个路径换为一个可读的，如下：

/usr/local/nginx/nginx.pid。


然后使用systemctl daemon-reload 更新下systemctl。

通过以上操作后发现service nginx start 不再有问题，但是stop还是不能停止，查看/etc/init.d/nginx这个文件后发现stop函数在运行后会删除掉$lockfile所指定的文件即：lockfile=/var/lock/subsys/nginx，而当打开这个目录发现在nginx停止后subsys目录下的nginx并没有被删除，所以导致使用service nginx stop 命令停止nginx不成功，解决这个问题只需使用root账户把此文件打开然后使用:wq保存一下即可。过后使用ll /var/local/subsys/nginx 发现这个文件的权限为644即可。

当然最好是在配置/etc/init.d/nginx时将其路径修改为有权限的目录下如：

lockfile=/usr/local/nginx/lock/nginx

这样即可。
--------------------- 
