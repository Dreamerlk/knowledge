
#####upstream模块相关说明
1、upstream模块应放于nginx.conf配置的http{}标签内
2、upstream模块默认算法是wrr (权重轮询 weighted round-robin)
#####一、分配方式
Nginx的upstream支持5种分配方式，下面将会详细介绍，其中前三种为Nginx原生支持的分配方式，后两种为第三方支持的分配方式。
1、轮询 轮询是upstream的默认分配方式，即每个请求按照时间顺序轮流分配到不同的后端服务器，如果某个后端服务器down掉后，能自动剔除。
```
upstream backend {
    server 192.168.1.101:8888;
    server 192.168.1.102:8888;
    server 192.168.1.103:8888;
}
```
2、weight 轮询的加强版，即可以指定轮询比率，weight和访问几率成正比，主要应用于后端服务器异质的场景下。
```
upstream backend { 
    server 192.168.1.101 weight=1; 
    server 192.168.1.102 weight=2;
    server 192.168.1.103 weight=3;
}
```
3、ip_hash 每个请求按照访问ip（即Nginx的前置服务器或者客户端IP）的hash结果分配，这样每个访客会固定访问一个后端服务器，可以解决session一致问题。
```
upstream backend { 
    ip_hash; 
    server 192.168.1.101:7777; 
    server 192.168.1.102:8888;
    server 192.168.1.103:9999;
}
```
注意：1、当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。2、导致负载不均衡。
4、fair fair顾名思义，公平地按照后端服务器的响应时间（rt）来分配请求，响应时间短即rt小的后端服务器优先分配请求。如果需要使用这种调度算法，必须下载Nginx的upstr_fair模块。
```
upstream backend { 
   server 192.168.1.101; 
   server 192.168.1.102; 
   server 192.168.1.103; fair;
}
```
5、url_hash，目前用consistent_hash替代url_hash与ip_hash类似，但是按照访问url的hash结果来分配请求，使得每个url定向到同一个后端服务器，主要应用于后端服务器为缓存时的场景下。
```
upstream backend {
    server 192.168.1.101; 
    server 192.168.1.102; 
    server 192.168.1.103; 
    hash $request_uri; 
    hash_method crc32;
}
```
其中，hash_method为使用的hash算法，需要注意的是：此时，server语句中不能加weight等参数。
提示：url_hash用途cache服务业务，memcached，squid，varnish。特点：每个rs都是不同的。
![](http://upload-images.jianshu.io/upload_images/6954572-b15ee1b79ec6205e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####二、设备状态
从上面实例不难看出upstream中server指令语法如下：
server address [parameters]
**参数说明：**
server：关键字，必选。
address：主机名、域名、ip或unix socket，也可以指定端口号，必选。
parameters：可选参数，可选参数如下：   
 1.down：表示当前server已停用   
 2.backup：表示当前server是备用服务器，只有其它非backup后端服务器都挂掉了或者很忙才会分配到请求。   
 3.weight：表示当前server负载权重，权重越大被请求几率越大。默认是1.   
 4.max_fails和fail_timeout一般会关联使用，如果某台server在fail_timeout时间内出现了max_fails次连接失败，那么Nginx会认为其已经挂掉了，从而在fail_timeout时间内不再去请求它，fail_timeout默认是10s，max_fails默认是1，即默认情况是只要发生错误就认为服务器挂掉了，如果将max_fails设置为0，则表示取消这项检查。

举例说明如下：
```
upstream backend { 
    server backend1.example.com weight=5; 
    server 127.0.0.1:8080 max_fails=3 fail_timeout=30s; 
    server unix:/tmp/backend3;
 }
```
 ![](http://upload-images.jianshu.io/upload_images/6954572-035f64f01a2346c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
