 安装git

yum install git

添加用户 设置密码

adduser git
passwd git

到用户目录下创建.ssh目录和文件

cd
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys

然后把需要登录的用户的公钥贴进去authorized_keys里

顺便备注一下权限命令
 chmod 600 ××× （只有所有者有读和写的权限） 
chmod 644 ××× （所有者有读和写的权限，组用户只有读的权限） 
chmod 700 ××× （只有所有者有读和写以及执行的权限） 
chmod 666 ××× （每个人都有读和写的权限） 
chmod 777 ××× （每个人都有读和写以及执行的权限）

创建裸仓库（这里放到了home/git/srv目录）

git init --bare test.git

git init 会创建.git目录 git init --bare 创建的目录里没有工作区，只记录历史信息，里面生成的文件就是上面.git目录里的文件

设置权限

chown -R git:git test.git

客户端clone仓库

git clone git@serverxxx:/home/git/srv/test.git
