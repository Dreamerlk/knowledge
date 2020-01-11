1、升级内核到3.10.x

方式一、yum安装

cd /etc/yum.repos.d

wget http://www.hop5.in/yum/el6/hop5.repo

yum install kernel-ml-aufs kernel-ml-aufs-devel

方式二、rpm安装（推荐）

rpm -ivh kernel-ml-aufs-3.10.5-3.el6.x86_64.rpm kernel-ml-aufs-devel-3.10.5-3.el6.x86_64.rpm

rpm可以到[http://down.51cto.com/data/1903250](http://down.51cto.com/data/1903250)下载

2、修改/etc/grub.conf中default=0

default=0

timeout=5

splashimage=(hd0,0)/grub/splash.xpm.gz

hiddenmenu

title CentOS (3.10.5-3.el6.x86_64)

root (hd0,0)

 kernel /vmlinuz-3.10.5-3.el6.x86_64 ro root=UUID=2d12dbef-aa63-4ad4-98bf-5beb0f9bfc48 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet

 initrd /initramfs-3.10.5-3.el6.x86_64.img

title CentOS (2.6.32-358.el6.x86_64)

root (hd0,0)

 kernel /vmlinuz-2.6.32-358.el6.x86_64 ro root=UUID=2d12dbef-aa63-4ad4-98bf-5beb0f9bfc48 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet

initrd /initramfs-2.6.32-358.el6.x86_64.img

3、重启系统

reboot

4、验证内核版本

[root@vmname1 ~]# uname -r

3.10.5-3.el6.x86_64

5、查看内核是否支持aufs

[root@vmname1 ~]# grep aufs /proc/filesystems

nodev aufs
