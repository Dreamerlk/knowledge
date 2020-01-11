###内存泄漏
&emsp;&emsp;内存泄漏指的是在程序运行过程中申请了内存，但是在使用完成后没有及时释放的现象， 对于普通运行时间较短的程序来说可能问题不会那么明显，但是对于长时间运行的程序， 比如Web服务器，后台进程等就比较明显了，随着系统运行占用的内存会持续上升， 可能会因为占用内存过高而崩溃，或被系统杀掉
###PHP的内存泄漏
&emsp;&emsp;[PHP](http://lib.csdn.net/base/php)属于高级语言，语言级别并没有内存的概念，在使用过程中完全不需要主动申请或释放内存， 所以在PHP用户代码级别也就不存在内存泄漏的概念了。
但毕竟PHP是使用C编写的解释器，而[C语言](http://lib.csdn.net/base/c)的程序是可能出现内存泄漏问题，所以本质上还是一样的，那么可以这么说：如果你的PHP程序内存泄漏了，会有三种可能：
首先肯能是自己的代码有问题，比如没有及时释放大内存的变量等。
很多公司都会有自己的PHP扩展，而扩展通常也使用C/C++来编写，这样扩展本身也可能会因为内存不正确释放而导致内存泄漏。
有些扩展是对第三方库的一种包裹， 比如PHP的sqlite数据库操作接口主要是在libsqlite之上进行了封装，所以如果 libsqlite本身有内存泄漏的话，那也可能会带来问题。

###PHP-CGI
&emsp;&emsp;根据官方的介绍，php-cgi不存在内存泄漏，每个请求完成后php-cgi会回收内存，但是不会释放给[操作系统](http://lib.csdn.net/base/operatingsystem)，这样就会导致大量内存被php-cgi占用。(这个对于内存不大的服务器，比如云主机就太致命了，利用命令：ps aux|grep php-cgi|grep -v grep|awk '{if($4>=1)print $2}'
 可以查询占比1%的php-cgi的pid，然后kill掉(kill -9),这个方法比较好) 官方的解决办法是降低PHP_FCGI_MAX_REQUESTS
的值。
###Nginx&PHP-FPM
&emsp;&emsp;这里先简单说一下nginx+php-fpm模式的工作原理：
nginx服务器fork出n个子进程（worker），php-fpm管理器fork出n个子进程。
当有用户请求，nginx的一个worker接收请求，并将请求抛到socket中。
php-fpm空闲的子进程监听到socket中有请求，接收并处理请求。

&emsp;&emsp;这里要重点说一下第三步骤。第三步涉及到php-fpm进程生命周期的东西。一个php-fpm的生命周期大致是这样的：模块初始化（MINIT）-> 模块激活（RINIT）-> 请求处理 -> 模块停用（RSHUTDOWN） -> 模块激活（RINIT）-> 请求处理 -> 模块停用（RSHUTDOWN）……. 模块激活（RINIT）-> 请求处理 -> 模块停用（RSHUTDOWN）-> 模块关闭（MSHUTDOWN）。在一个php-fpm进程的生命周期里，会有多次的模块激活（RINIT）-> 请求处理 -> 模块停用（RSHUTDOWN）的过程。这个“请求处理”的大致过程是这样的：php读取相应的php文件，对其进行词法分析，生成opcode，zend虚拟机执行opcode。
&emsp;&emsp;PHP配置文件里面的memory_limit 这个东西，其实，它限制的只是这个“请求处理”的内存。所以，这个参数跟php-fpm进程占用的内存并没有什么关系。php是用c写的，所以，难免又会一些内存泄露。也就是说，在“请求处理”这个过程结束后，有些变量没有被销毁，然后就导致一个php-fpm进程占用的内存越来越大。
那么，有什么办法能阻止这个问题呢？ php-fpm.conf中有个参数pm.max_requests
，等同于PHP_FCGI_MAX_REQUESTS。该值的意思是一个fpm进程处理多少个请求后自动杀掉另起新进程。 这个参数默认是关闭的，我们需要开启这个参数，并且适当降低这个值，用以让php-fpm自动的释放内存。另一个跟它有关联的值max_children，这个是每次php-fpm会建立多少个进程，这样实际上的内存消耗是max_children*max_requests*每个请求使用内存，根据这个我们可以预估一下内存的使用情况，就不用再写脚本去kill了。
