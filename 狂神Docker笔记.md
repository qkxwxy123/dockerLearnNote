# 狂神Docker笔记

## Docker的常用命令

### 容器命令

**说明：我们有了镜像才可以创建容器，linux，下载一个centos镜像来测试学习**

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image

# 参数说明
--name="Name"    容器名字  tomcat01 tomcat02，用来区分容器
-d               后台方式运行
-it              使用交互方式运行，进入容器查看内容
-p               指定容器的端口 -p  8080:8080
	-p ip:主机端口:容器端口
    -p 主机端口:容器端口（常用）
    -p 容器端口 不走外面端口
    容器端口
-P               随机指定端口

# 测试，启动并进入容器
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker run -it centos /bin/bash
[root@a4dc7f490639 /]#ls     #查看容器内的centos，基础版本，很多命令都是不完善的
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@a4dc7f490639 /]# exit
exit
[root@iz2ze4t93bwwn1hyg068wpz ~]# ls
[root@iz2ze4t93bwwn1hyg068wpz ~]# cd ..
[root@iz2ze4t93bwwn1hyg068wpz /]# ls
bin   dev  home  lib64       media  opt    proc  run   srv  tmp  var
boot  etc  lib   lost+found  mnt    patch  root  sbin  sys  usr  www
```

**列出所有的运行的容器**

```shell
#docker ps 命令
     # 列出当前正在运行的容器
-a   # 列出当前正在运行的容器+带出历史运行过的容器
-n=？   #显示最近创建的容器	
-q   #只显示容器的编号，静默模式

[root@iz2ze4t93bwwn1hyg068wpz /]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@iz2ze4t93bwwn1hyg068wpz /]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
a4dc7f490639        centos              "/bin/bash"         3 minutes ago       Exited (0) About a minute ago                       strange_mcclintock
7fd19a2cb21e        hello-world         "/hello"            6 hours ago         Exited (0) 6 hours ago                              eager_varahamihira
```

**退出容器**

```
exit	# 直接容器停止并退出
Ctrl + P + Q # 容器不停止退出
```

**删除容器**

```shell
docker rm 容器id                 # 删除指定的容器，不能删除正在运行的容器，强制需要rm -f
docker rm -f $(docker ps -aq)   # 删除所有的容器
docker ps -a -q|xargs docker rm # 删除所有的容器，管道技术
```

**启动和停止容器的操作**

```shell
docker start 容器id    # 启动容器
docker restart 容器id  # 重启容器
docker stop 容器id     # 停止当前正在运行的容器
docker kill 容器id     # 强制停止当前容器
```





### 常用其他命令

**后台启动容器**

```shell
# 通过命令docker run -d 镜像名
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker run -d centos

# 问题docker ps，发现 centos 停止了

# 常见的坑，docker 容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止。
# 比如nginx，容器启动后发现自己没有提供服务，就会立刻停止，就是没有程序了（垃圾回收机制）
```

**查看日志**

```shell
docker logs
```

