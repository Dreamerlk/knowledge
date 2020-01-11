在移动wap中，经常会使用window.location.href去跳转页面，这个方法在绝大多数浏览器中都不会 
存在问题，但早上测试的同学会提出了一个bug：在安卓手机的微信自带浏览器中，这个是失效的，并没有跳转；

原来的代码：
```
window.location.reload(location.href);
```
初步判断可能是缓存的问题，首先想到的解决办法就是在要跳转的url后面加个时间戳，告知浏览器这是一个新的请求；
```
window.location.reload(location.href+'?time='+((new Date()).getTime()));
```
然而并没有什么卵用，看了下js文档：


href是location对象的一个属性，reload()则是location对象的方法
所以对于href，可以为该属性设置新的 URL，使浏览器读取并显示新的 URL 的内容。
对于reload()则是重新加载当前文档，如果该方法没有规定参数，或者参数是 false，它就会用 HTTP 头 If-Modified-Since 来检测服务器上的文档是否已改变。如果文档已改变，reload() 会再次下载该文档。如果文档未改变，则该方法将从缓存中装载文档。这与用户单击浏览器的刷新按钮的效果是完全一样的。如果把该方法的参数设置为 true，那么无论文档的最后修改日期是什么，它都会绕过缓存，从服务器上重新下载该文档。这与用户在单击浏览器的刷新按钮时按住 Shift 健的效果是完全一样。

但对于安卓手机微信中的浏览器，reload只是从缓存中装载文档，所以当你使用该方法，是失效的；

 

解决办法就是，使用location.href代替reload(),而且在以后的使用中也强烈建议大家使用location.href来进行刷新或者跳转
```
window.location.href = location.href+'?time='+((new Date()).getTime());
```
