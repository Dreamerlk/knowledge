后台运行脚本

    执行脚本test.sh:./test.sh
    中断脚本test.sh：ctrl+c
    在1的基础上将运行中的test.sh，切换到后台并暂停：ctrl+z
    执行ctrl+z后，test.sh在后台是暂停状态（stopped）,使用命令：bg number让其在后台开始运行（“number”是使用jobs命令查到的 [ ]中的数字，不是pid）

    直接在后台运行脚本test.sh:./test.sh &
    查看当前shell环境中已启动的任务情况：jobs
    将test.sh切换到前台运行：fg %number（”number”为使用jobs命令查看到的 [ ] 中的数字，不是pid）
    中断后台运行的test.sh脚本：先fg %number切换到前台，再ctrl+c；或是直接kill %number

以上两种在后台运行test.sh的方法，当遇到退出当前shell终端时，后台运行的test.sh也就结束了。这是因为以上两种方法使得test.sh在后台运行时，运行test.sh进程的父进程是当前shell终端进程，关闭当前shell终端时，父进程退出，会发送hangup信号给所有子进程，子进程收到hangup以后也会退出。所以要想退出当前shell终端时test.sh继续运行，则需要使用nohup忽略hangup信号。

    不中断的在后台运行test.sh：nohup ./test.sh &（test.sh的打印信息会输出到当前目录下的nohup.out中）
    使用jobs可看到test.sh处于running状态
    使用ps -ef |grep test.sh可查看到正在运行的test.sh脚本进程
    退出当前shell终端，再重新打开，使用jobs看不到正在运行的test.sh，但使用ps -ef可以看到

在后台不中断的运行test.sh，可以使用nohup忽略hangup信号，或者使用setsid将其父进程改为init进程（进程号为1）

    不中断的在后台运行test.sh另一个命令：setsid ./test.sh &
    使用ps -ef |grep test.sh可看到test.sh进程的父进程id为1

测试脚本

    #！/bin/bash
     
    int=1
    while(( $int<=100 ))
    do
        echo $int
        let "int++"
        sleep 1
    done
