最近centos发布了最近版本7.3。centos 7.0，7.1，7.2支持升级到7.3最新版本。

本文将教你如何升级centos到最新版本。

centos中“update”命令可以一次性更新所有软件到最新版本。

注意：不推荐使用update的y选项，-y选项会让你在安装每项更新前都进行确认（译者注：这样会非常费时间，更新进度忙）；

对于centos 5.X和6.X的系统我们在更新后需要重新安装应用程序恢复数据，庆幸的是centos7不需要这么麻烦，可以直接升级。为了安全起见，如果你有重要数据的话还是建议升级系统前做好备份。

以下是centos 7.X升级的步骤
一、检查系统版本

$ cat /etc/redhat-release

CentOS Linux release 7.1.1503 (Core)
二、备份重要数据（例如/etc, /var,/opt）。如果centos是安装在虚拟机上，那么可以使用快照进行备份。像VMware虚拟机可以快照备份，当然更奢侈一点是备份整个虚拟机。也可以针对重要程序数据进行备份，例如MySQL, Appache, Nginx, DNS等等。

三、运行yum命令升级

$ sudo yum clean all

$ sudo yum update
四、重启系统

$ sudo reboot
五、查看现在系统版本

$ cat /etc/redhat-release

CentOS Linux release 7.3.1611 (Core)
