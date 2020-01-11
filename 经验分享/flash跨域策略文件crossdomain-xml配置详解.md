一、简介


flash在跨域时唯一的限制策略就是crossdomain.xml文件，该文件限制了flash是否可以跨域读写数据以及允许从什么地方跨域读写数据。

位于www.a.com域中的SWF文件要访问www.b.com的文件时，SWF首先会检查www.b.com服务器目录下是否有crossdomain.xml文件，如果没有，则访问不成功；若crossdomain.xml文件存在，且里边设置了允许www.a.com域访问，那么通信正常。所以要使Flash可以跨域传输数据，其关键就是crossdomain.xml。

本文将着重介绍crossdomain.xml文件的配置方法及不同配置对flash跨域的影响。


二、crossdomain.xml的配置


2.1 crossdomain.xml的放置位置


自flash 10以后，如有跨域访问需求，必须在目标域的根目录下放置crossdomain.xml文件，且该根目录下的配置文件称为“主策略文件”。若不存在主策略文件，则该域将禁止任何第三方域的flash跨域请求。

主策略文件对全站的跨域访问起控制作用。

也可以单独在某路径下放置仅对该路径及其子路径生效的crossdomain.xml配置文件，这需要在flash的AS脚本中使用如下语句来加载该配置文件：[具体的加载权限限制，将受后文中site-control策略的影响]

Security.loadPolicyFile("http://www.xxx.com/subdir/crossdomain.xml")


2.2 crossdomain.xml的配置方法及影响


crossdomain.xml需严格遵守XML语法，有且仅有一个根节点cross-domain-policy，且不包含任何属性。在此根节点下只能包含如下的子节点：site-control、allow-access-from、allow-access-from-identity、allow-http-request-headers-from。下面将分别介绍这四个子节点：


2.2.1 site-control：通过检查该节点的属性值，确认是否可以允许加载其他策略文件。[如果该策略文件并非主策略文件，则此节点被自动忽略]

每个site-control标签有且仅有属性permitted-cross-domain-policies，该属性指定相对于非主策略文件的其他策略文件的加载策略。permitted-cross-domain-policies属性值有如下情况：

none: 不允许使用loadPolicyFile方法加载任何策略文件，包括此主策略文件。

master-only: 只允许使用主策略文件[默认值]。

by-content-type:只允许使用loadPolicyFile方法加载HTTP/HTTPS协议下Content-Type 为text/x-cross-domain-policy的文件作为跨域策略文件。

by-ftp-filename:只允许使用loadPolicyFile方法加载FTP协议下文件名为       crossdomain.xml的文件作为跨域策略文件。

all: 可使用loadPolicyFile方法加载目标域上的任何文件作为跨域策略文件，甚至是一 个JPG也可被加载为策略文件！[使用此选项那就等着被xx吧！]

如需要对网站某子目录做单独的flash跨域限制策略，在主策略文件中必须进行相应的site-control设置。

下面的例子将site-control策略配置为可加载本服务器中其它的text/x-cross-domain-policy文件作为跨域策略文件。

<cross-domain-policy>

     <site-control permitted-cross-domain-policies="by-content-type" />

</cross-domain-policy>

 
2.2.2 allow-access-from：通过检查该节点的属性值，确认能够读取本域内容的flash文件来源域。


allow-access-from标签有三个属性：

·domain：该属性指定一个确切的 IP 地址、一个确切的域或一个通配符域（任何域）。只有domain中指定的域，才有权限通过flash读取本域中的内容。

可采用下列两种方式之一来表示通配符域：

1）单个星号（*），如：<allow-access-fromdomain="*" />，表示匹配所有域和所有 IP 地址，此时任何域均可跨域访问本域上的内容。[这是极不安全的！]

2）后接后缀的星号，表示只匹配那些以指定后缀结尾的域，如*.qq.com可匹配   game.qq.com、qq.com。形如www.q*.com或www.qq.*的为无效配置。

Tips：当domain被指定为IP地址时，只接受使用该IP作为网址来访问的来源请求[此时ip地址也就相当于一个域名而已]，如domain被设置为192.168.1.100时，使用http://192.168.1.100/flash.swf 来请求该域内容是允许的，但是使用指向192.168.1.100的域名www.a.com来访问时[http://www.a.com/flash.swf]将会被拒绝，因为flash不懂得dns解析：）

·to-ports：该属性值表明允许访问读取本域内容的socket连接端口范围。可使用to-ports="1100,1120-1125"这样的形式来限定端口范围，也可使用通配符（*）表示允许所有端口。

·secure：该属性值指明信息是否经加密传输。当crossdomain.xml文件使用https加载时，secure默认设为true。此时将不允许flash传输非https加密内容。若手工设置为false则允许flash传输非https加密内容。

下面的例子配置为允许所有qq.com下的所有二级域名[包括qq.com本身]通过https访问本域中的内容。

<cross-domain-policy>

     <allow-access-from domain="*.qq.com" secure="true" />

</cross-domain-policy>


2.2.3 allow-access-from-identity：该节点配置跨域访问策略为允许有特定证书的来源跨域访问本域上的资源。每个allow-access-from-identity节点最多只能包含一个signatory子节点。形如：

<allow-access-from-identity>

   <signatory>

     <certificate

       fingerprint="01:23:45:67:89:ab:cd:ef:01:23:45:67:89:ab:cd:ef:01:23:45:67"

       fingerprint-algorithm="sha-1"/>

   </signatory>

</allow-access-from-identity>


2.2.4 allow-http-request-headers-from：此节点授权第三方域flash向本域发送用户定义的http头。


allow-access-from节点授权第三域提取本域中的数据，而 allow-http-request-headers-from 节点授权第三方域将数据以http头的形式发送到本域中。[简而言之，allow-access-from是控制读取权限，allow-http-request-headers-from是控制以http头形式的写入权限]

allow-http-request-headers-from包含三个属性：

·domain：作用及参数格式与allow-access-from节点中的domain类似。

·headers：以逗号隔开的列表，表明允许发送的http头。可用通配符（*）表示全部    http头。

·secure：作用及用法与allow-access-from节点中的secure相同。

在下面的示例中，任何域都可以向当前域发送 SOAPAction 标头：

<cross-domain-policy>

     <allow-http-request-headers-from domain="*" headers="SOAPAction" />

</cross-domain-policy>

 
三、总结
 

不正确的crossdomain.xml策略将导致严重的安全问题，如信息泄露、CSRF等。从上文中可以看出，在进行安全评估时，我们应重点关注以下几点：

1）allow-access-from标签的domain属性检测：domain属性应根据最小化原则按需设置，仅允许可信任的来源跨域请求本域内容。禁止将该属性值设置为“*”。

2）allow-http-request-headers-from标签的domain属性检测：domain属性应根据最小化原则按需设置，仅允许可信任的来源向本域跨域发送内容。禁止将该属性值设置为“*”。

3） site-control标签的permitted-cross-domain-policies属性检测：根据业务的实际需求及可行性，对该属性做相应设置。禁止将该属性值设置为“all”。 
