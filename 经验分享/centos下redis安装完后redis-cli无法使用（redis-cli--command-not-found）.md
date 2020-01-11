### 一、下载安装
```
 # 从redis官网下载redis-cli的压缩包
wget http://download.redis.io/redis-stable.tar.gz 
 
# 解压下载下来的压缩包
tar xvzf redis-stable.tar.gz
 
# 进入redis-stable目录
cd redis-stable
 
# 安装
make
 
# 将redis-cli拷贝到/usr/local/bin/下，让redis-cli指令可以在任意目录下直接使用
sudo cp src/redis-cli /usr/local/bin/
```

按照上面的指令执行之后redis-cli就可以正常执行了，如果失败，再把上面的命令再执行一遍即可

### 二、成功后的效果如下图

![centos下redis安装完后redis-cli无法使用（redis-cli: command not found）](http://upload-images.jianshu.io/upload_images/6954572-18b9637cf01ef0d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "centos下redis安装完后redis-cli无法使用（redis-cli: command not found）")

在以上实例中我们连接到本地的 redis 服务并执行 PING 命令，该命令用于检测 redis 服务是否启动

### 三、在远程服务上执行命令

如果需要在远程 redis 服务上执行命令，同样我们使用的也是 redis-cli 命令。

语法：

```
$ redis-cli -h host -p port -a password
```

示例：

```
# 以下示例演示了如何连接到主机IP为 127.0.0.1，端口号为 6379 ，密码为 mypass 的 redis 服务上。
 
$ redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
```
