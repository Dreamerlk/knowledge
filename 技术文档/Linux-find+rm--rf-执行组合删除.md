 Linux find+rm -rf 执行组合删除

 

[ 语法 ]

 

# find  对应目录  -mtime + 天数  -name " 文件名 " -exec rm -rf {} \;

 

 

[ 举例说明 ]

 

# find /usr/local/backups -mtime +10 -name "*.*" -exec rm -rf {} \;

 

-- 将 /usr/local/backups 目录下所有 10 天前带 "." 的文件删除

 

 

[ 参数说明 ]

 

      find                         --linux 的查找命令，用户查找指定条件的文件

　　 /usr/local/backups      -- 想要进行清理的任意目录

　　 -mtime                     -- 标准语句写法

　　＋ 10                         -- 查找 10 天前的文件，这里用数字代表天数

　　 "*.*"                        -- 希望查找的数据类型， "*" 表示查找所有文件

　　 -exec                       -- 固定写法

　　 rm -rf                       -- 强制删除文件，包括目录

　　 {} \;                        -- 固定写法，一对大括号 + 空格 +\ 
