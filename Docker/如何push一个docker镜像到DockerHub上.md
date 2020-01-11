有时候想要保存自己的docker镜像，又不想自己搭建docker registry，那么就可以了借用DockerHub来用，一般不会有多少人在意你的镜像，不过万一被人看上了呢，这谁说的准呢，废话不多说，下面来看看操刀记录

1.  在DockerHub上创建账号：[https://hub.docker.com/](https://hub.docker.com/)
    这里我的账号是firewarm
2.  本地下载镜像（这里拿alpine做示例）,并为镜像打tag

```
[root@host-30 ~]# docker pull alpine:3.4
[root@host-30 ~]# docker tag alpine:3.4 firewarm/alpine:3.4
```

3.  登录到DockerHub上

```
[root@host-30 ~]# docker login
# 输入用户名和密码
```

4.  push镜像到DockerHub上

```
[root@host-30 ~]# docker push firewarm/alpine:3.4
The push refers to a repository [docker.io/firewarm/alpine]
4fe15f8d0ae6: Pushed 
3.4: digest: sha256:dc89ce8401da81f24f7ba3f0ab2914ed9013608bdba0b7e7e5d964817067dc06 size: 528
```

5.  打完收工
