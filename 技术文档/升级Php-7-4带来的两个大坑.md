由于我机器用的滚动更新的Archlinux，不知不觉Php已经升级到7.4了，没想到这次更新带来了极大的麻烦。首先是Php-fpm的新选项ProtectHome会导致经典的File not found错误，再是Php解释器会对null类型的下标访问直接报错Trying to access array offset on value of type null。

最近在帮一个朋友张罗一个网站，于是把线上代码拉回本地做镜像进行测试。因为web应用有些奇怪的依赖，为了不污染本机的环境，我就把它部署在Docker中进行测试。Docker的基础镜像选择了激进的Archlinux，搭配上个月底才出炉的Php7.4。于是花了整整一个下午栽在Debug大坑中…

首先是一把梭配好了环境后，一跑，报了Php-fpm最经典也是最坑的错误之一：File not found。配过Php-fpm的都知道出现这个错误一般是文件权限不对或者文件路径不对，而这两个错误都是比较难找的。于是我又双叒叕体验了一把大眼瞪小眼的路径检查，没问题。文件权限检查，emmm也没问题呀？又返回去检查路径，还是没问题！搞到最后气的chmod 777一把梭竟然也没能解决问题，有点怀疑人生…

网上搜索Php-fpm的File not found错误，虽然结果很多，可原因都只有这两个。而这两个原因也都被一一排除了，事情突然向神奇的角度发展起来了...

不知过了多久之后我才想到可能是跟Php版本有关（因为我本机也跑了其它Php应用，所以一开始并不觉得Php有问题）。于是我去搜了一下新版Php7.4及Php-fpm7.4的改动，一下就发现了罪魁祸首：

[Php7.4 Commit](https://github.com/php/php-src/commit/40c4d7f1820df1872a71ab07fd26da45a203e37f#diff-c0605c0e7e1db864472acf66a9812d33R22 "php commit ProtectHome")


这个提交中添加了一个选项：ProtectHome。顾名思义，开启了之后php**不会去执行在家目录中的文件**——而这个新选项的默认值恰好是开启的。使用systemctl edit php-fpm.service添加一个选项覆盖，重启服务后，终于一切正常，并迎来第二个大坑错误：

Php中经常使用inlcude，require等来包含其它文件。而调试发现在某个include之后，php直接停止执行并报错Trying to access array offset on value of type null。但是在线上的代码跑起来却一点问题也没有，这就很奇怪了，跟到include的文件中之后发现是有个地方在访问数组元素，而数组本身却是null。在Php这种弱类型语言中这种语法一般是支持的，它会整体返回null，而在新版的Php7.4中这个语法却会报告为错误。看来Php也在一点点规范语言的特性，没办法，这个只能自己改代码了。（虽然我目前选择了使用旧版本的Php）

由于Php7.4在上个月底才刚刚发布，估计还没有大面积更新使用，各个应用的开发者可能也没有针对Php7.4进行过测试和兼容修改。也正是因此，在网上搜索这些信息时，找不到什么有价值的建议，这篇文章除了记录下被这个新特性坑了一下午之外，也算给其它人留一个解决类似问题的思路吧。

原文地址：[https://blog.sbw.so/u/php-fpm-7.4-file-not-found-array-type-null-error.html](https://blog.sbw.so/u/php-fpm-7.4-file-not-found-array-type-null-error.html)
