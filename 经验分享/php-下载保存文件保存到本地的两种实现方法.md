第一种：
```
<?php 
function downfile()
{
 $filename=realpath("resume.html"); //文件名
 $date=date("Ymd-H:i:m");
 Header( "Content-type:  application/octet-stream "); 
 Header( "Accept-Ranges:  bytes "); 
Header( "Accept-Length: " .filesize($filename));
 header( "Content-Disposition:  attachment;  filename= {$date}.doc"); 
 echo file_get_contents($filename);
 readfile($filename); 
}
downfile();
?>
```
或者：
```
<?php 
function downfile($fileurl)
{
 ob_start(); 
 $filename=$fileurl;
 $date=date("Ymd-H:i:m");
 header( "Content-type:  application/octet-stream "); 
 header( "Accept-Ranges:  bytes "); 
 header( "Content-Disposition:  attachment;  filename= {$date}.doc"); 
 $size=readfile($filename); 
  header( "Accept-Length: " .$size);
}
 $url="url地址";
 downfile($url);
?> 
```
第二种：
```
<?php 
function downfile($fileurl)
{
$filename=$fileurl;
$file  =  fopen($filename, "rb"); 
Header( "Content-type:  application/octet-stream "); 
Header( "Accept-Ranges:  bytes "); 
Header( "Content-Disposition:  attachment;  filename= 4.doc"); 
$contents = "";
while (!feof($file)) {
 $contents .= fread($file, 8192);
}
echo $contents;
fclose($file); 
}
$url="url地址";
downfile($url);
?>
```
问题：
1：如果文件没有以下载文件的形式展示而是以直接在浏览器中打开了，可添加如下代码：
```
header("Content-Disposition:attachment);
```
Content-Disposition 消息头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。

Content-Disposition消息头最初是在MIME标准中定义的，HTTP表单及[`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST "HTTP POST 方法 发送数据给服务器. 请求主体的类型由 Content-Type 首部指定. 一个 POST 请求通常是通过 HTML 表单发送, 并返回服务器的修改结果. 在这种情况下, content type 是通过在 <form> 元素中设置正确的 enctype 属性, 或是在 <input> 和 <button> 元素中设置 formenctype 属性来选择的:") 请求只用到了其所有参数的一个子集。只有`form-data`以及可选的`name`和`filename`三个参数可以应用在HTTP场景中。

在HTTP场景中，第一个参数或者是inline（默认值，表示回复中的消息体会以页面的一部分或者整个页面的形式展示），或者是attachment（意味着消息体应该被下载到本地；大多数浏览器会呈现一个“保存为”的对话框，将filename的值预填为下载后的文件名，假如它存在的话）。
