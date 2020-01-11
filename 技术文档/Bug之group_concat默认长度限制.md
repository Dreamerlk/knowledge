问题：mysql数据库使用group_concat将多个图片的地址连接，每个地址长度都标准是55，发现在超过18张后，第十九张图片地址会被截断；

原因：mysql的group_concat默认连接长度为1024字符，也就是说你需要连接后的连接超过1024字符，它只会显示这么长，其余部分都会被截取丢掉。

解决办法：
（1）使用sql语句（亲测可用）
```
SET GLOBAL group_concat_max_len=102400;
SET SESSION group_concat_max_len=102400; 
```

（2）修改配置文件
在mysql配置文件中添加如下这句，修改配置文件后记得需要重启mysql服务
```
group_concat_max_len = 102400
```

解决过程：前两天的项目中写了一个获取资源数据的接口，每条数据对应很多图片，图片在保存的的时候不是直接以逗号分隔而是单独存表通过外键连接的。在查询的时候就想到了使用group_concat将每一个资源数据对应的图片地址查询出然后以逗号连接成一条数据。今天下午同事突然告诉我，好像图片一超过七张就显示不出来了。查了一下后端返回数据，确实后面的地址不全导致加载不到。
通过debug打出执行sql，直接在navicat中运行发现结果一样,sql语句如下
```
SELECT a.*,( SELECT GROUP_CONCAT( image ) FROM resource_image b WHERE b.resource_data_id = a.id ) AS images FROM resource_data a WHERE a.state <> 0
```

查了好久一无所获，最后将整个项目数据库导出到另一台服务器上，运行同样的sql居然没有截断。就断定这绝壁是那个数据库问题，然后又换了一台服务器，七张图片时没有截断正常。把整个项目部署了一套在自己觉得没问题的服务器上，再查看数据，七张图片没问题，但好像超过十八张就又出问题了，这一次是所有服务器统一，当然第一台服务器还是在七张的时候截。此时突然有个想法出现，是不是group_concat的问题，果断百度了group_concat的长度，果真如此，原来mysql默认限制group_concat长度1024，也就是最后那几台服务器在第19张图片地址的时候截取的长度，而第一台明显被人设置过。在使用两条sql语句
```
SET GLOBAL group_concat_max_len=102400;
SET SESSION group_concat_max_len=102400;
```
 之后问题果断解决。
