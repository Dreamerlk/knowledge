> 作为一个程序员，我们在做一个项目的时候，往往需要用到一台正式的服务器和一台测试的服务器。如果你的主机配置足够好，那么，你可以利用虚拟机在同一台主机上装多台Web应用的服务器。当然，土豪公司的话也可以多用几台机器去搭建服务器。但是，这些都不是最好的解决办法，最好的办法就是使用Docker。使用docker可以在一台机器上搭建多台服务器，它没有传统虚拟机那么笨重，而且消耗的系统资源也相对较少。并且它是一个隔离的环境，任你在容器里面搅得天昏地暗，对host OS也不会有任何影响。因此它的安全性也很高

关于什么是docker，建议大家先上网查查有关的用法。如果您不了解，在这篇文章中，您可以简单的理解为他是一个轻量级的虚拟机。

**一、docker安装mysql**
首先，我们从仓库拉取一个MySql的镜像

```
docker pull mysql:5.6
```

然后我们可以通过命令 docker images 查看我们刚刚拉下来的mysql的镜像

![docker images ](http://upload-images.jianshu.io/upload_images/6954572-f5619b459dbedc53?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来，我们就开始运行并启动一个容器，通过以下命令

```
docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=xy123456 --name xy_mysql mysql:5.6
```

**参数说明**
-d 让容器在后台运行
-p 添加主机到容器的端口映射
-e 设置环境变量，这里是设置mysql的root用户的初始密码，这个必须设置
–name 容器的名字，随便取，但是必须唯一

> ps:其实我们可以仅仅使用docker run命令就行了。docker run会先去pull，然后再create。个人习惯先把镜像pull下来，在run的时候会很快。

接下来我们就可以通过命令docker ps -a 查看我们刚刚创建的容器

![docker ps -a](http://upload-images.jianshu.io/upload_images/6954572-13732b4bff63c17d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们可以看到我的容器状态的Up状态，表示容器正在运行，并且把可以看到主机和容器的端口映射关系。

接下来，我们就可以进入到我们刚刚创建的容器中，输入命令

```
docker exec -ti xy_mysql /bin/bash
```

**参数说明**
-t 在容器里生产一个伪终端
-i 对容器内的标准输入 (STDIN) 进行交互

容器中默认是没有vim的，所以我们首先要安装vim,需要注意的是安装前记得先执行apt update命令，不然安装会出现问题。
进入到mysql容器后，我们通过创建一个远程可以访问的用户，这样我们就能从别的主机访问到我们的数据库了。

**二、docker安装php-fpm**
同样首先我们拉取php-fpm的镜像

```
docker pull php:7.0-fpm
```

再创建一个phpfpm容器

```
docker run -d -v /var/nginx/www/html:/var/www/html -p 9000:9000 --link xy_mysql:mysql --name xy_phpfpm php:7.0-fpm 
```

**参数说明**
-d 让容器在后台运行
-p 添加主机到容器的端口映射
-v 添加目录映射，即主机上的/var/nginx/www/html和容器中/var/www/html目录是同步的
–name 容器的名字
–link 与另外一个容器建立起联系，这样我们就可以在当前容器中去使用另一个容器里的服务。

这里如果不指定–link参数其实也是可以得，因为容易本身也是有ip的且唯一，所以我们也可以直接利用ip去访问容器。

然后进入到我们的容器，然后我们在/var/www/html目录下新建一个index.php文件

```
touch index.php
```

我们可以看到该目录下新建了一个php文件
接下来我们回到我们的主机上面，访问一下我们主机上/var/nginx/www/html
[图片上传中...(image-49cf22-1514939750417-2)]

我们发现我们在容器里的/var/www/html目录中新建的文件也在主机的/var/nginx/www/html目录中，因为在创建容器的时候，我们已经把主机中的目录挂载到了容器中去了。

因为后面我要使用pdo模块进行测试，所以我需要自己安装pdo_mysql模块，在docker容器中可以这样来安装

```
docker-php-ext-install pdo_mysql
```

然后我们可以通过命令php -m查看我们的php的所有扩展模块，我们可以去看到我们刚刚安装的pdo_mysql扩展也在里面
[图片上传中...(image-796fc6-1514939750417-1)]

**三、docker安装nginx**
首先，我们从仓库里去拉取一个nginx镜像

```
docker pull ngixn:1.10.3
```

接下来运行nginx容器

```
docker run -d -p 80:80 --name xy_nginx\ 
-v /var/nginx/www/html:/var/www/html\
--link xy_phpfpm:phpfpm --name xy_nginx nginx:1.10.3
```

**参数说明：**

-d 让容器在后台运行
-p 添加主机到容器的端口映射
-v 添加目录映射,这里最好nginx容器的根目录最好写成和php容器中根目录一样。但是不一点非要一模一样,如果不一样在配置nginx的时候需要注意
–name 容器的名字
–link 与另外一个容器建立起联系

然后进入nginx容器，修改nginx的配置文件让它支持php

```
docker exec -ti xy_nginx /bin/bash
```

**参数说明**
-t 在容器里生产一个伪终端
-i 对容器内的标准输入 (STDIN) 进行交互

在容器里找到nginx的配置文件，默认是在/etc/nginx目录下

```
location ~ \.php$ {
        root           /var/www/html;
        fastcgi_index  index.php;
        fastcgi_pass   phpfpm:9000;//这里改成我们之前--link进来的容器，也可以直接用php容器的ip
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcdi_script_name;//如果你的根目录和php容器的根目录不一样，这里的$document_root需要换成你php下的根目录，不然php就找不到文件了
        include        fastcgi_params;                                                                                                                                               

    }

```

最后，我们来测试一下我们的安装是否成功

```
<?php
try {
    $con = new PDO('mysql:host=mysql;dbname=test', 'xuye', 'xy123456');
    $con->query('SET NAMES UTF8');
    $res =  $con->query('select * from test');
    while ($row = $res->fetch(PDO::FETCH_ASSOC)) {
        echo "id:{$row['id']} name:{$row['name']}";
    }
} catch (PDOException $e) {
     echo '错误原因：'  . $e->getMessage();
}

```

## ![这里写图片描述](http://upload-images.jianshu.io/upload_images/6954572-fd23a959b6adb121?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当当当，看到正确的输出，就证明我们的配置成功了。一个最最最基本的环境就搭建好了。是不是很简单？

不知道大家有没有注意到我测试代码中的

```
$con = new PDO('mysql:host=mysql;dbname=test', 'xuye', 'xy123456');
```

这一行，我新建容器的时候并没有把mysql容器link进来，这里的host我直接用mysql也能成功，为什么呢？因为真正执行这段代码的是php容器，（如果不清楚nginx和php之间的关系，最好先上网查资料弄清楚）而之前我们在php容器里把php容器link进去了，所以这里是可行的，当前换成mysql容器的ip也是一样的，可以通过dokcer inspect contanier_name|id来查看容器的有关信息， 不过只能在内网里面使用容器的ip。如果你想在外网访问容器里的mysql，还是要通过主机的公网ip:port这种形式来访问。

> ps:上面我们都是通过输入一条条命令去创建容器，为了更高效的创建容器，我们可以事先写一个shell脚本，把这些命令打包，通过命令sh ***.sh去执行脚本
