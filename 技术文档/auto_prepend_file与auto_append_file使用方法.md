auto_prepend_file与auto_append_file使用方法

如果需要将文件require到所有页面的顶部与底部。


第一种方法：在所有页面的顶部与底部都加入require语句。

例如：

    require('header.php');
    页面内容
    require('footer.php');

但这种方法如果需要修改顶部或底部require的文件路径，则需要修改所有页面文件。而且需要每个页面都加入require语句，比较麻烦。


第二种方法：使用auto_prepend_file与auto_append_file在所有页面的顶部与底部require文件。

php.ini中有两项

auto_prepend_file 在页面顶部加载文件

auto_append_file  在页面底部加载文件

使用这种方法可以不需要改动任何页面，当需要修改顶部或底部require文件时，只需要修改auto_prepend_file与auto_append_file的值即可。


例如：修改php.ini，修改auto_prepend_file与auto_append_file的值。

    auto_prepend_file = "/home/fdipzone/header.php"
    auto_append_file = "/home/fdipzone/footer.php"

修改后重启服务器，这样所有页面的顶部与底部都会require /home/fdipzone/header.php 与 /home/fdipzone/footer.php


注意：auto_prepend_file 与 auto_append_file 只能require一个php文件，但这个php文件内可以require多个其他的php文件。


如果不需要所有页面都在顶部或底部require文件，可以指定某一个文件夹内的页面文件才调用auto_prepend_file与auto_append_file

在需要顶部或底部加载文件的文件夹中加入.htaccess文件，内容如下：

    php_value auto_prepend_file "/home/fdipzone/header.php"
    php_value auto_append_file "/home/fdipzone/footer.php"


这样在指定.htaccess的文件夹内的页面文件才会加载 /home/fdipzone/header.php 与 /home/fdipzone/footer.php，其他页面文件不受影响。

使用.htaccess设置，比较灵活，不需要重启服务器，也不需要管理员权限，唯一缺点是目录中每个被读取和被解释的文件每次都要进行处理，而不是在启动时处理一次，所以性能会有所降低。
