在[Docker使用部分](https://docs.docker.com/userguide/usingdocker)我们接触到了通过网络端口来连接运行在Docker容器内的服务。这是同Docker容器内服务和应用互动的方法之一。在这一节中，我们将带你复习一下通过网络端口连接到Docker容器并给你介绍容器连接的概念。

网络端口映射刷新

在[Docker的使用部分](https://docs.docker.com/userguide/usingdocker)中，我们创建了一个运行Python Flask应用的容器。

``$ sudo docker run -d -P training/webapp python app.py``

> 注意：容器有一个内部网络和IP地址（还记得在[使用Docker](https://docs.docker.com/userguide/usingdocker/)部分中我们曾使用docker的监控命令查看容器的IP地址）。Docker可以有各种不同的网络配置。你可以在[这里](https://docs.docker.com/articles/networking/)看到Docker网络的更多相关信息。

当我们创建容器时，我们使用-P标志来自动映射任意网络端口到我们Docker主机上介于49000到49900之间的随机高位端口。随后，当我们运行docker ps时，我们会看到5000端口绑定到了49155端口上了。

``$ sudo docker ps nostalgic_morse``
``CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse``

我们同样看到了怎样使用-P标志将一个容器端口绑定到一个指定端口。

``$ sudo docker run -d -p 5000:5000 training/webapp python app.py``

另外，我们发现这并不是一个好办法，因为它约束我们一个特定端口只能有一个容器.

还有一些其他的使用 -p 的方法。 默认的情况下 -p 会把container的Port绑到宿主机上的所有ip的该端口号上。 (译者：如果你的宿主机有多个IP， 每个IP的那个端口都会被绑到container的那个port上) 我们可以通过指定接口, 例如下面的例子， 端口就只绑定到了 localhost 上。

``$ sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py``

这会吧container内部的5000端口绑定到宿主机的 127.0.0.1 的5000端口上去.

如果只指定了网络接口而没有指定端口号， 则container的端口会绑到这个网络接口上的一个随机端口上.

``$ sudo docker run -d -p 127.0.0.1::5000 training/webapp python app.py``

我们还可以使用UDP

``$ sudo docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py``

使用 docker port 可以查看当前的container的端口的绑定情况， 和特殊的端口的配置。 例如， 如果我们吧container的端口绑到了宿主机的 localhost， 我们输入 docker port 的时候输出会是:

``$ docker port nostalgic_morse``
``127.0.0.1:49155``

> 注 -p 可以被多次使用用来吧多个container里的port绑定到宿主机上

Docker 容器链接

映射网络端口不是吧container彼此连接起来的唯一方法。Docker的linking系统允许你吧多个 container连接起来， 让他们彼此交互信息。Docker的linking会创建一种父子级别的关系。 父container可以看到他的子container提供的信息。

容器命名

Docker的linking系统依赖于container的名字。我们已经注意到了每个container都会被自动的 分配一个名字， 在本教程里大家可能已经熟悉了 nostalgic_morse 这个名字（译者：docker 自动分配的名字都是确实存在的词， 有些名字比较有意思 例如我的一个container就叫做tender einstein 温柔的爱因斯坦， 对于大多数情况， 这些名字绝对是考验你英语单词量的机会, 比如 backstabbing nobel, stoic carson）. 我们可以自己对container命名. 命名有两个用处:

1.  好记 比如命名一个承载web服务的container为web

2.  container 之间可以互相引用， 例如一个 web container 使用一个 db container

通过 --name 可以给container命名, 例如:

``$ sudo docker run -d -P --name web training/webapp python app.py``

你可以看到我们启动了一个新的container, 启动的时候使用了 --name 命名这个container为 web。 我们可以通过命令 docker ps查看container的名字

``$ sudo docker ps -l``
``CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
aed84ee21bde  training/webapp:latest python app.py  12 hours ago  Up 2 seconds 0.0.0.0:49154->5000/tcp  web``

我们也可以用 docker inspect 查看container的名字.

``$ sudo docker inspect -f "{{ .Name }}" aed84ee21bde
/web``

(译者： 上面那个乖乖的语法源于golang自带的模板语言)

>注 Container的名字必须是唯一的。也就是说你只能命名一个container为 web。 要重新使用一个container的名字的时候必须把之前叫这个名字的container删除， 才能 再使用。删除container可以使用 docker rm. 还有个便利的方法就是在启动container的时候 使用 --rm 标记. 这样container停止的时候就会自动被删除。 （译者： 如果你使用过一段时间docker， 用docker ps -a 你可能会发现一大堆的已经停止了的container, 在很多的docker教程里， 在启动container的时候都会带上 --rm.)

容器链接

Links运行container之间发现彼此并且彼此间安全的通讯。使用 --link可以创建一个link. 让我们创建一个运行数据库的container。

``$ sudo docker run -d --name db training/postgres``

这里我们基于 training/postgres image创建了一个叫做 db 的container, 上面提供PostgreSQL 数据库服务.

现在让我们创建一个叫做 web的container， 并且把他和 db container连接到一起。

``$ sudo docker run -d -P --name web --link db:db training/webapp python app.py``

这个命令会吧 db 和 web 连接到一起 --link 的用法：

``--link name:alias``

这里 name 是我们要连接的container的名字， alias 是一link的别名. 下面我们会看到如何使用这个 别名.

下面让我们用 docker ps 看看被连接到一起的container们。

``$ docker ps``
``CONTAINER ID  IMAGE                     COMMAND               CREATED             STATUS             PORTS                    NAMES
349169744e49  training/postgres:latest  su postgres -c '/usr  About a minute ago  Up About a minute  5432/tcp                 db
aed84ee21bde  training/webapp:latest    python app.py         16 hours ago        Up 2 minutes       0.0.0.0:49154->5000/tcp  db/web,web``

这里我们可以看到我们创建的两个分别叫 db 和 web 的container, 注意 web container 在name列里还显示了另外一个名字db/web。 这个名字告诉我们 web container 被连接到了 db container， 并且建立了一种父子关系。

linking到底有什么用呢？我们已经看到了link在两个container间创建了一个 父子 关系. 父container 这个例子里的 db 可以得到他的子container web上的信息. Docker是通过在 两个container建立了一个安全通道来实现的, 这样container就不用对外暴露端口了. 你可能已经注意到了 我们在启动 db container的时候没有使用 -p 或者 -P。 因为我们已经吧两个container通过 link连接起来了， 所以没必要通过端口暴露数据库的服务了

Docker通过下述两种方法吧子container里的信息暴露给父container:

*   环境变量

*   更新 /etc/host 文件

我们先看下docker设定的环境变量。 在 web containre里， 让我们运行 env 命令 列出所有的环境变量。

``root@aed84ee21bde:/opt/webapp# env``
``HOSTNAME=aed84ee21bde``
``DB_NAME=/web/db``
``DB_PORT=tcp://172.17.0.5:5432``
``DB_PORT_5000_TCP=tcp://172.17.0.5:5432``
``DB_PORT_5000_TCP_PROTO=tcp``
``DB_PORT_5000_TCP_PORT=5432``
``DB_PORT_5000_TCP_ADDR=172.17.0.5``

> 注: 这些环境变量只是为第一个在container里的进程设置的。 类似的守护进程(例如 sshd) 会在新建子shell的时候抹除这些变量

我们可以看到Docker创建了一些列的对于我们使用db很有用的环境变量。 每一个变量都以 DB 开头， 这个 DB 就是上面的那个别名。（译者： 为啥用别名这里就清楚了, image 不用知道其他container的名字， 只用在run的时候把名字映射为自己了解的就可以了) 如果我们起的别名是 db1 那么环境变量的名字的前缀就是 DB1_. 你可以使用这些环境 变量来配置你的应用程序来连接 dbcontainer 里的数据库。 连接是安全的、私有的 仅限于 web 和 db 之间。

除了环境变量之外Docker会把父container的IP添加到子container的/etc/hosts里。我们 看下一下 web container里的hosts文件：

``root@aed84ee21bde:/opt/webapp# cat /etc/hosts``
``172.17.0.7  aed84ee21bde``
``. . .``
``172.17.0.5  db！``

我们可以看到， 有两个相关的host配置项。 第一个是给 web container 的， 名字 就是container的ID。 第二条吧父container的别名和父container的IP绑到了一起。我们 试着通过host的名字ping一下.

``root@aed84ee21bde:/opt/webapp# apt-get install -yqq inetutils-ping``
``root@aed84ee21bde:/opt/webapp# ping db``
``PING db (172.17.0.5): 48 data bytes``
``56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms``
``56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms``
``56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms``

> 注 上面第一个命令是用来安装ping的， 因为我们使用的container里没有安装。

我们可以使用这个host吧web服务和数据库服务连接到一起。

注
一个父container可以连接多个子container。 例如， 我们可以把多个web服务的container连接 到一个数据库container上
