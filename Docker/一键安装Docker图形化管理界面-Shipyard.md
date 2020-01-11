Shipyard是一款开源的图形化的Docker管理工具，记得以前安装很麻烦的，现在官方有了自动安装脚本，使用非常方便。复制、粘贴、使用，就这么简单。先不研究他是如何实现的，安装使用起来再说。

      $ curl -s https://shipyard-project.com/deploy | bash -s
```
 Deploying Shipyard
 -> Starting Database
 -> Starting Discovery
 -> Starting Cert Volume
 -> Starting Proxy
 -> Starting Swarm Manager
 -> Starting Swarm Agent
 -> Starting Controller
Waiting for Shipyard on 192.168.2.xxx:8080
..
Shipyard available at http://192.168.2.xxx:8080

Username: admin Password: shipyard
```
Shipyard 启用了7个容器，默认访问端口是8080，默认用户名和密码是admin 和 shipyard

![20161101130338795.jpg](http://upload-images.jianshu.io/upload_images/6954572-380b6695dd928e57.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注意事项：

如果安装出现了问题怎么办？是否是因为端口冲突？网络出现问题怎么办？这个项目用到了哪些Docker镜像？一键安装的脚本是如何实现的？

1、Shipyard的默认访问端口为8080，这个端口许多程序都会用，使用时尽量要避免冲突。如果你在测试机器上安装过多款软件，然后再安装Shipyard时，却发现无法访问Shipyard，可以考虑一下，是不是因为端口被其他程序占用的问题。

2、由于网络的原因，因此第一次执行时可能不会很顺利，镜像可能未下载全，又或者端口冲突，导致无法通过8080端口访问shipyard页面。查看主机发现其中有几个Shipyard容器已经运行了，怎么办？不妨先使用 docker ps -a 命令，查看一下正在进行的容器情况，然后用docker stop xxx 把7个shipyard开头的容器都停止掉、最后再用docker rm xxx 把上一次安装出现问题的容器都删除掉，最后再次执行curl这一行命令。

3、比较稳妥的方法是先下载这七个Docker镜像，然后再运行这一行。其中rethinkdb 181MB，shipyard/shipyard 58MB，七个一共300MB。
```
docker pull alpine
docker pull swarm 
docker pull shipyard/shipyard
docker pull rethinkdb
docker pull microbox/etcd
docker pull ehazlett/curl 
docker pull shipyard/docker-proxy
```
4、如果访问不了，请检查你使用的浏览器，记得使用谷歌的chrome浏览器。

5、安装Shipyard 的脚本地址： https://shipyard-project.com/deploy，有兴趣的可以看一看如何部署一个小型的容器应用。
