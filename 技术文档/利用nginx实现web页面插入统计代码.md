今天，突然研发的同事说需要在公司网站页面底部添加一个流量统计的scripts，但是不打算改代码，所以希望能在代理的层面解决。这里只讨论如果是在nginx来实现，技术如何操作。但是具体是研发改代码，还是写在nginx，这个视情况而定。
这里可以使用 nginx 的 ngx_http_sub_module 模块完成。我这里是使用tengine，所以这个模块已经有了。如果没有的话，自己编译添加以下：方法如下：
# 查看后发现，默认安装没有开启这个模块
nginx -V
![nginx页面替换01](http://upload-images.jianshu.io/upload_images/6954572-576cace964602ecc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "nginx页面替换")

# 保存好修改的文件，使用brew卸载掉nginx再重装
然后修改nginx配置文件，在对应的 server > location 上下文中，添加 sub_filter 指令。
这里使用了变量 $injected 保存了脚本地址，对response中有 body 标签的地方修改插入。

# nginx.conf
```
location  /  {

    ...

    set  $injected  '<script src="统计代码引用的js地址如：http://localhost:8088/other/status.js"></script>';

    sub_filter  '<body>'  '<body>${injected}';

    sub_filter_types *;

    sub_filter_once on;
    #sub_filter 一行代码前面是需要替换的内容，后面单引号内是替换成的内容。
    #sub_filter_once 意思是只查找并替换一次。on是开启此功能，off是关闭——默认值是on。
    #sub_filter_types 一行意思是选定查找替换文件类型为文本型。也可以不加此行，因为默认只查找text/html文件。
    #sub_filter模块可以用在http, server, location模块中。主要作用就是查找替换文件字符。
}

 ```

这里还可以在 sub_filter 多添加一条规则，将客户端ip地址替换注入到页面中。

# 替换响应中的 UUIP_IP 字段为 ip 地址
sub_filter 'UUIP_IP' '${remote_addr}';

转载自：http://www.ppzedu.com/archives/1049.html
