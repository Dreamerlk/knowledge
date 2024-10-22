Docker出现后，容器技术在互联网领域得到了空前的普及，无论是大公司还是屌丝创业公司的码农基本上都会在各种技术社区或者各种演讲会议上了解到过相关技术，我们作为一家屌丝创业公司也不例外，去年对Docker做了一番了解，并在年前测试了一些方案，今天在这里总结一下遇到的各种坑以及踩坑过程中的一些思考，希望能对了解过Docker并且跃跃欲试的同学有点启发，当然更欢迎在这方面有丰富经验的同学给一些建议或者指点。

> 这篇文章不会提及Docker的理论知识或者基本概念相关的内容，如果想了解Docker的基本使用，Docker的[官方文档](https://docs.docker.com/)是个不错的选择，如果想了解基本的理论，可以参考下coolshell博客上的几篇文章：[Docker基础技术：Linux Namespace（上）](http://coolshell.cn/articles/17010.html)、[Docker基础技术：Linux Namespace（下）](http://coolshell.cn/articles/17029.html)、[Docker基础技术：Linux CGroup](http://coolshell.cn/articles/17049.html)、[Docker基础技术：AUFS](http://coolshell.cn/articles/17061.html)、[Docker基础技术：DeviceMapper](http://coolshell.cn/articles/17200.html)。

## 一、项目环境

我们项目用的标准的PHP技术栈：PHP-FPM + Nginx，运行在阿里云的ECS上，数据库也是阿里云的服务。

## 二、缘起

对于一个普通的屌丝创业公司的屌丝项目来说，理论上来说是没必要用太复杂的技术的，对新技术的克制也是码农的一个职业操守。然而我之所以在项目上做这个尝试也并非是想尝试新技术，而是源自于项目上线以来遇到的各种问题，比如前面文章里有提到过的[laravel的性能问题](https://segmentfault.com/a/1190000002511729)，而HHVM或者PHP7这些新技术都大大提到了性能，一下子全部切换风险太大，所以可以用容器来运行某几个服务测试一下；再比如之前有出现过一个BUG，某个接口出现问题占用内存太高导致整个系统响应超时；再比如，看了各种技术大会上别人分享的经验确实想自己试一下，哈哈。

## 三、牛刀小试

根据Docker的理念，每个容器都是一个独立的服务，服务之间通过接口相互协作。我们采用的方案是PHP-FPM和Nginx分别跑在不同的容器中，PHP-FPM容器暴露端口给Nginx容器。

容器之间的网络通信最初用的是Docker自带的link，link虽然很简单，但是不太好用，link是通过在容器启动时修改/etc/hosts文件来实现的，当时用的时候遇到的一个问题是被link的容器重启后，这个容器IP变了，但是link的容器里并没有更新（1.10版本对link机制做了修改，不知道是否解决了这个问题，还没具体看），还有一个问题就是当同一个服务需要开多个容器时就搞不定了，当然如果是用集群的方式来跑就更搞不定了，所以link只能在本地开发时玩玩，在生产环境没啥用。最后这部分用的方案是用consul来做服务发现，关于consul就不在这里展开介绍了，有兴趣的同学可以看下这篇文章：[Service Discovery for Docker Containers using Consul and Registrator](https://www.livewyer.com/blog/2015/02/05/service-discovery-docker-containers-using-consul-and-registrator)。

然后代码是放在host上，在容器启动时挂载一下代码的目录。根据接口使用的场景将接口分到了几个不同的容器里面去跑，对资源隔离做了下压力测试，符合预期，很满意！然后选了两个容器跑HHVM测试了下，确实性能提高了很多。整个结构如下图所示：

![c456cf9a1ce061932a362ae59b78fa3a.jpeg](http://upload-images.jianshu.io/upload_images/6954572-4fde0739ff26e592.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这样可以正常使用了，然后就是日志的处理，最开始挂载了一个host上的目录用来存放日志，但是这样也不太符合Docker的理念，于是就又弄了个logstash容器，将各种容器的日志写到logstash容器里面去。

然后就是对容器的监控，我们用了oneapm提供的服务，还不错，也有其他一些开源工具的选择，但是用oneapm的服务更方便一些。

一个完整的技术栈就这样搞定了，跑了几天也没出现什么问题。作为一个有追求的码农，当然是要有更高的要求的，后来想到这样一个问题：Docker的理念是容器即是一个完整的服务，但是我们的用法是把在容器内把PHP-FPM跑起来了，代码并不在里面，而是通过目录挂载的方式来实现的，这样就需要在host上面部署好代码，没有完美的实现容器即服务的理念。好，我们追求下完美！

## 四、完美之路

其实把代码放进容器并不只是为了去追求理念上的完美，还有另外一个用意，那就是基于Docker的CI。基本思路是每次发布都会build一个新版本的镜像，build过程中需要把代码更新到最新版本，然后安装依赖的第三方库以及其他一些业务相关的预处理，build完成后再push到registry，然后在各个节点pull下来，重启容器。整个流程如下图所示：

![dac9492d4576f1b1355bd0f6f29ebba2.jpeg](http://upload-images.jianshu.io/upload_images/6954572-10fd981d77aa61da.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


怎么样？这下完美了吧！如果现实也是那么完美就好了，但是偏偏现实总是会有各种各样的坑等着你！

首先，在没有用这个机制之前，发布一次代码也就几秒钟的事情，merge下分支，然后各个节点pull下代码就可以了，但是用了Docker后，merge分支后就需要build一下镜像，build后再push到registry，然后再到各个节点把镜像pull下来，少说也是要花分钟级别的。这时候有的同学可能会说Docker镜像的原理不是layer机制么，对，确实是layer，每次build只需要更新需要更新的layer就可以，但是还是没办法做到像git那样只更新那几个文件，至少要更新整个项目的代码目录。

其次，之前只需要更新一下代码就可以了，现在却需要把容器重启一下！本来重启容器是很快的，但是在容器里PHP-FPM跑起来后却很慢，我猜测是在执行docker stop时会发一个SIGTERM的信号到容器里的进程，PHP-FPM在处理SIGTERM信号时会做一些清理工作。一个节点至少有几十个容器，每个容器都重启一下非常耗时！

这样本来很简单的一个发布，用了这个「完美」的机制后却需要好几分钟。虽然整个流程完全自动化，但是确实没必要为了完美的理念做出这样一个妥协。

此外还遇到了另外一个非常奇葩的问题：升级到1.10后，容器启动后经常会遇到端口不通的情况，这时候如果进入到容器里发起一次网络请求，比如ping以下某个IP，端口就会恢复正常。用各种工具调试了很久依然没找出来问题在哪，屌丝创业公司也没精力研究这么高深的问题，只好暂时搁置了。

挣扎了很久，最后又回到了老路子。

## 五、面对现实

虽然没能理念上完美的方案，但是也要让整套架构更方便的扩展，比如在流量高峰期快速上架节点。得益于云计算近几年的发展，这个问题也比较容易解决。前面提到代码是放在host的一个目录，那么基本上每个host只需要三个条件就能够上线：

1.  项目代码目录
2.  安装Docker
3.  配置好salt-minion（用来更新配置文件）

只需要在购买一台ECS后，做完上面的操作，生成一个系统镜像，需要上线新节点时用这个系统镜像来直接配置服务器即可。节点上线后，启动各个容器，通过前面提到的用consul实现的服务发现机制，自动加入集群。

## 六、继续探索

后面打算用Docker Swarm来简单做下集群管理，未来对调度需求比较大时可能会尝试下Mesos或者Kubernetes。不过关于容器的生态链都发展的非常快，可能等需要的时候又有更好的方案了吧。

简单总结下整个过程的感悟吧，从架构上来说，永远都没有完美的架构，只有最适合自己的架构。从码农角度来说，在做技术选型时最重要的是权衡当前项目需求、团队对各种技术了解的程度以及该技术选型带来的复杂度。切勿看了几个大牛的技术架构分享就在自己项目上跃跃欲试，最后弄的连自己都搞不定还给别人留下一个烂摊子收拾。最后，在项目迭代过程中保持项目的简单的能力永远是衡量一个码农技术的水平的重要标准。
