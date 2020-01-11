php.ini 中缺省的最长执行时间是 30 秒，这是由 php.ini 中的 max_execution_time 变量指定，如果脚本需要跑很长时间，例如要大量发送电子邮件，或者分析统计大量数据，服务器会在 30 秒后强行中止正在执行的程序，这种情况就要更改php脚本最大执行时间。

PHP设置脚本最大执行时间的三种方法

1、在php.ini里面设置

max_execution_time = 120;

2、通过PHP的ini_set函数设置

ini_set("max_execution_time", "120");

3、通过set_time_limit 函数设置

set_time_limit(120);

以上几个数字设置为0则无限制，脚本会一直执行下去，直到执行结束。

所以，需要长时间执行的脚本，一般在php代码开头处添加如下代码就可以了
set_time_limit(0);

done!
