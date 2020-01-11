在平常的工作中，经常会遇到一些例行任务，需要每天定时运行。解决这类问题就可以使用crontab命令，下面一起来看一下～
 首先需要启动crontab服务
 
service crond start #启动服务
service crond stop #关闭服务
service crond restart #重启服务
service crond reload #重新载入配置
 
 
然后使用crontab -e进行编辑，然后进行例行任务的编辑，之后保存退出即可。
具体的格式说明如下：
 每一行的格式为：分　时　日　月　周　命令
第1列表示分钟1～59 每分钟用*或者 */1表示
第2列表示小时1～23（0表示0点）
第3列表示日期1～31
第4列表示月份1～12
第5列标识号星期0～6（0表示星期天）
第6列要运行的命令
 
值得注意的是，在crontab中，无论是命令还是文件的路径都要写全，否则不认。
比如要每天6点执行一个/home/run/test.sh的脚本，可以配置如下：
0 6 * * * /sbin/sh /home/run/test.sh 
或者
0 6 * * * cd /home/run && /bin/sh test.sh
这里需要注意的是，test.sh脚本中的命令也需要使用全路径，否则crontab找不到的。
 如果还想有一些其他的需求，比如保留输出可以将输出重定向，与正常的脚本运行没有区别
 
0 6 * * * cd /home/run && /bin/sh test.sh > log
