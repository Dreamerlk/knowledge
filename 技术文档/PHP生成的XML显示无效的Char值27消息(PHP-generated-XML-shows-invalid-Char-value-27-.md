问 题

我使用PHP库生成XML如下：

$ dom = new DOMDocument（“1.0”，“utf- 8“）; 

执行以上操作会在输出的顶部显示一条消息。

此页面包含以下错误：
错误在第16行在列274505：PCDATA无效的字符值27
下面是一个页面的渲染，第一个错误。**

我已经尝试使用Tidy库纠正。使用iconv获取UTF-8中的中文字符。

解决方案

> 在本网站建议一个有用的功能来摆脱这个错误。
> [http://www.phpwact.org/php/i18n/charsets#common_problem_areas_with_utf8](http://www.it1352.com/"http://www.phpwact.org/php/i18n/charsets#common_problem_areas_with_utf-8\")

当你将utf-8编码的字符串放在一个XML文档中时，你应该记住，并非所有utf-8有效字符都是已在XML文档中接受 [http://www.w3.org/TR/REC-xml/#charsets](http://www.it1352.com/"http://www.w3.org/TR/REC-xml/#charsets\")

所以你应该去掉不需要的字符，否则你会有一个XML致命的解析错误，如下面

```
 function utf8_for_xml（$ string）
 {
 return preg_replace（'/[^\x{0009}\x{000a}\x{000d}\x{0020}-\x{D7FF}\x{E000}-\x{FFFD}]+/u'，''，$ string）; 
} 
  
```

希望节省别人一些时间..
