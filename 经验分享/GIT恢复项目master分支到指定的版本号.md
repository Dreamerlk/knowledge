先备份当前的master分支的代码，可以以当前master分支拉取一个备份分支

然后开始恢复
1：在该项目的master分支中执行  git checkout  指定版本号
2：git push -f origin master
3：git push -f origin HEAD:master
然后就ok了。
