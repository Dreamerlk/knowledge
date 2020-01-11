场景

在Windows下用sublime text 3创建了sh文件用于跑定时脚本任务，但是上传到Unix服务器之后发现并没有被执行到。手动执行，发现报错：
报错信息
看到，换行的同时，把shell命令给覆盖了一部分。
vim 对应文件，执行:set ff 命令，发现fileformat=dos 表示这是DOS格式的文件
测试

同时执行原先在Unix环境下创建的sh文件，则一切正常。
找到原因

确定原因，是因为在Windows系统下创建的sh文件的换行格式和Unix系统不同，导致Unix系统不能识别
解决方案

经过Google，百度blabla，归纳出三种解决方案：
一、从源头解决问题：用sublime text 3创建出现的问题，就从st3上去解决。

    Perferences->Settings->User，添加配置”default_line_ending”: “unix”，保存
    将出现问题的sh文件一个个删除，再重新写入shell命令，保存。
    优点：一劳永逸，以后使用st3创建的sh文件直接上传到Unix服务器就可以直接执行，而不会出现问题了。
    缺点：一个个删除文件再重新创建太麻烦了点。- -!!

二、用dos2unix命令
```
dos2unix file.sh

```

优点：一次性可以修复所有文件的换行格式问题
缺点：服务器上修改文件格式，如果有用到git或者SVN版本控制软件，可能导致服务器上的sh文件版本改变，下次从本地上传文件，需要先还原这次修改的内容才可以，还是别这么干吧。。

三、用:set ff=unix命令
```
:set ff=unix

 ```

或
```
:set fileformat=unix

   ```

优点：修改个别文件，不会误操作（唯一能想到的优点）
缺点：（跟dos2unix命令的缺点一样）

四、最佳方案：结合方案一，再使用sed命令

    st3设置Perferences->Settings->User，添加配置”default_line_ending”: “unix”，保存
    本地虚拟机终端使用sed命令sed -i "s/\r//" file.sh

优点：一劳永逸且修改方便
结果

完美解决，再执行sh file.sh，发现正常运行，一切那么美好。。
