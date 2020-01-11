点击后报错404

前些日子部署了一个Tomcat 9.0，但是打开host-manager时却提示404，刚开始以为XML配置文件限制了ip访问，修改无果，打开webapps后才发现少了一个文件夹：host-manager。查看文档后才知道从Tomcat 5.5 以后的binary 核心安装版不再集成Tomcat Administration Web Application，需要独立下载安装。

解决办法

在以前的Tomcat版本安装目录里面找到WebApps文件夹下的host-manager文件夹将它复制到新安装的Tomcat 9.0同样的目录下即可。

点击后报错403及解决办法

 在tomcat-users.xml里添上几句即可 打开webapps下的host-manager和manager，都有一个共同的文件夹META-INF，里面都有context.xml，这个文件的内容是：
```
 <Context antiResourceLocking="false" privileged="true" > <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127.d+.d+.d+|::1|0:0:0:0:0:0:0:1" /> </Context>
```
 通过查看官方文档，知道，这段代码的作用是限制来访IP的，127.d+.d+.d+|::1|0:0:0:0:0:0:0:1，是正则表达式，表示IPv4和IPv6的本机环回地址，所以这也解释了，为什么我们本机可以访问管理界面，但是其他机器确?403。找到原因了，那么修改一下这里的正则表达式即可，比如我们只允许内网网段192.168.88访问管理页面，那么改成这样就可以：
```   
 <Context antiResourceLocking="false" privileged="true" > <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="192.168.88.*" /> </Context>
```
tomcate8配置管理界面的用户角色
```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```
修改完毕，关闭浏览器，重新打开tomcat，问题解决！
