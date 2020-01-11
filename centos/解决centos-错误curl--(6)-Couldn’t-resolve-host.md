今天在一新买的VPS上练手easypanel，才开始安装就提示 curl: (6) Couldn’t resolve host ... ，网上有说是因为开启了VPS IPV6的缘故，但凭直觉还是DNS解析的嫌疑更大，直接PING 国内域名，果然不通，更有把握了。

  centos 6修改DNS非常简单，直接

  vi /etc/resolv.conf

  在里面按下面格式添加后保存生效即可

  nameserver 119.29.29.29

  nameserver 8.8.8.8 
