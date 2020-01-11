php 获取今日、昨日、上周、本月的起始时间戳和结束时间戳的方法，主要使用到了 php 的时间函数 mktime。下面首先还是直奔主题以示例说明如何使用 mktime 获取今日、昨日、上周、本月的起始时间戳和结束时间戳，然后在介绍一下 mktime 函数作用和用法。
~~~ 
//php获取今日开始时间戳和结束时间戳
$beginToday=mktime(0,0,0,date('m'),date('d'),date('Y'));
$endToday=mktime(0,0,0,date('m'),date('d')+1,date('Y'))-1;
//php获取昨日起始时间戳和结束时间戳
$beginYesterday=mktime(0,0,0,date('m'),date('d')-1,date('Y'));
$endYesterday=mktime(0,0,0,date('m'),date('d'),date('Y'))-1;
//php获取上周起始时间戳和结束时间戳
$beginLastweek=mktime(0,0,0,date('m'),date('d')-date('w')+1-7,date('Y'));
$endLastweek=mktime(23,59,59,date('m'),date('d')-date('w')+7-7,date('Y'));
//php获取本月起始时间戳和结束时间戳
$beginThismonth=mktime(0,0,0,date('m'),1,date('Y'));
$endThismonth=mktime(23,59,59,date('m'),date('t'),date('Y'));
~~~
语法
mktime(hour,minute,second,month,day,year,is_dst)
hour 	可选。规定小时。
minute 	可选。规定分钟。
second 	可选。规定秒。
month 	可选。规定用数字表示的月。
day 	可选。规定天。
year 	可选。规定年。在某些系统上，合法值介于 1901 - 2038 之间。不过在 PHP 5 中已经不存在这个限制了。
is_dst 	可选。如果时间在日光节约时间(DST)期间，则设置为1，否则设置为0，若未知，则设置为-1。

自 5.1.0 起，is_dst 参数被废弃。因此应该使用新的时区处理特性。
用法

参数总是表示 GMT 日期，因此 is_dst 对结果没有影响。

参数可以从右到左依次空着，空着的参数会被设为相应的当前 GMT 值。

注意在 PHP 5.1 之前，如果该函数的参数非法，则会返回 false。

另外需要注意的是该函数对于日期运算和验证非常有用。它可以自动校正越界的输入，如：
1 	echo(date("M-d-Y",mktime(0,0,0,12,36,2001)));

将输出结果如：

Jan-05-2002
