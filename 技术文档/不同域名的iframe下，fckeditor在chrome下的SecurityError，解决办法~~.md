我这个fckeditor版本，不知道是几，反正在chrome下报错了

http://10.0.103.118:8080"的页面，用iframe嵌套了http://www.a.com:8080的页面，

后者的页面里使用了fckeditor，

然后就报错了。

Uncaught SecurityError: Blocked a frame with origin "http://www.a.com:8080" from accessing a frame with origin "http://10.0.103.118:8080". Protocols, domains, and ports must match. fckeditorcode_gecko.js:36

解决办法：
打开fckeditor\editor\js\fckeditorcode_gecko.js
---------------------------------
找到：
FCKTools.FixDocumentParentWindow = function(A){ if (A.document) A.document.parentWindow=A; for (var i=0;i<A.frames.length;i++) FCKTools.FixDocumentParentWindow(A.frames[i]);};
修改为：
FCKTools.FixDocumentParentWindow = function(A){try{ if (A.document) A.document.parentWindow=A;} catch(e){};for (var i=0;i<A.frames.length;i++) FCKTools.FixDocumentParentWindow(A.frames[i]);};

也就是try,catch了一下，就好使了~~汗~~
