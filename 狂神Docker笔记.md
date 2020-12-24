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
docker logs -f -t --tail 容器，没有日志

# 自己编写一段shell脚本
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker run -d centos /bin/sh -c "while true;do echo kxxy; sleep 1; done"

[root@iz2ze4t93bwwn1hyg068wpz ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
e8b0b46f444d        centos              "/bin/sh -c 'while t…"   4 seconds ago       Up 3 seconds                            affectionate_mcnulty

# 显示日志
 -tf           # f -follow持续显示日志，t时间戳
 --tail number # 要显示的日志行数
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker logs -tf --tail 10 e8b0b46f444d

```

**查看容器中进程信息**

```shell
# 命令 docker top 容器id
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker top e8b0b46f444d
UID                 PID                 PPID                C                   STIME     
root                28821               28804               0                   14:45     
root                29447               28821               0                   14:50     
```

**查看镜像的元数据**

```shell
# 命令
docker inspect 容器id

# 测试
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker inspect e8b0b46f444d
[
    {
        "Id": "e8b0b46f444d352243c3b3d4ab58df326cfc1d857dac00b33cd8f8f4f09b3c1f",
        "Created": "2020-12-10T06:45:41.694915332Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo kxxy; sleep 1; done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 28821,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-12-10T06:45:42.37550912Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/e8b0b46f444d352243c3b3d4ab58df326cfc1d857dac00b33cd8f8f4f09b3c1f/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/e8b0b46f444d352243c3b3d4ab58df326cfc1d857dac00b33cd8f8f4f09b3c1f/hostname",
        "HostsPath": "/var/lib/docker/containers/e8b0b46f444d352243c3b3d4ab58df326cfc1d857dac00b33cd8f8f4f09b3c1f/hosts",
        "LogPath": "/var/lib/docker/containers/e8b0b46f444d352243c3b3d4ab58df326cfc1d857dac00b33cd8f8f4f09b3c1f/e8b0b46f444d352243c3b3d4ab58df326cfc1d857dac00b33cd8f8f4f09b3c1f-json.log",
        "Name": "/affectionate_mcnulty",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/28119bd7b7a6a10b44c846a17d9c57e0dfba046e5cd4af6ea47e8a5a99ed8f3b-init/diff:/var/lib/docker/overlay2/7762de1c34b1663532d9c1f47fa4a316787208eec60ead4f1f32c2e02ed1c915/diff",
                "MergedDir": "/var/lib/docker/overlay2/28119bd7b7a6a10b44c846a17d9c57e0dfba046e5cd4af6ea47e8a5a99ed8f3b/merged",
                "UpperDir": "/var/lib/docker/overlay2/28119bd7b7a6a10b44c846a17d9c57e0dfba046e5cd4af6ea47e8a5a99ed8f3b/diff",
                "WorkDir": "/var/lib/docker/overlay2/28119bd7b7a6a10b44c846a17d9c57e0dfba046e5cd4af6ea47e8a5a99ed8f3b/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "e8b0b46f444d",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo kxxy; sleep 1; done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "a97d0ac9c52e49ce8c1f2834c3df62c0ff25afe6c994366d870b977913cce250",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/a97d0ac9c52e",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "252517f9b82b12b414f17e56c58c606aeacf4524389fbd1c08c7a08433870bee",
            "Gateway": "172.18.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.18.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:12:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "1223cf8fa88c7b74f3bd7b36b788a4c2b2247bca404aa05d0ba53bad78d3b2ca",
                    "EndpointID": "252517f9b82b12b414f17e56c58c606aeacf4524389fbd1c08c7a08433870bee",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

**进入当前正在运行的容器**

```shell
# 我们通常容器都是使用后台方式运行的，需要进入容器，修改一些配置

# 命令
docker exec -it 容器id bashShell

# 测试
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e16e11186028        centos              "/bin/bash"         2 days ago          Up 2 days                               angry_banach
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker exec -it e16e11186028 /bin/bash
[root@e16e11186028 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@e16e11186028 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Dec08 pts/0    00:00:00 /bin/bash
root        14     0  0 07:16 pts/1    00:00:00 /bin/bash
root        28    14  0 07:16 pts/1    00:00:00 ps -ef

# 方式二
docker attach 容器id
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker attach e16e11186028
[root@e16e11186028 /]# 
进入后进到正在执行的代码...

# docker exec      # 进入容器后开启一个新的终端，可以操作（常用）
# docker attach    # 进入容器正在执行的终端，不会启动新的进程
```

**从容器内拷贝文件到主机**

```shell
docker cp 容器id:容器内路径  目的的主机路径

# 查看当前主机
[root@iz2ze4t93bwwn1hyg068wpz ~]# cd /home
[root@iz2ze4t93bwwn1hyg068wpz home]# ls
www

# 进入docker容器
[root@iz2ze4t93bwwn1hyg068wpz home]# docker attach e16e11186028
[root@e16e11186028 /]# cd /home
[root@e16e11186028 home]# ls
[root@e16e11186028 home]# touch test.java
[root@e16e11186028 home]# ls
test.java
[root@e16e11186028 home]# exit
exit
[root@iz2ze4t93bwwn1hyg068wpz home]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS               NAMES
e8b0b46f444d        centos              "/bin/sh -c 'while t…"   25 hours ago        Exited (137) 9 minutes ago                       affectionate_mcnulty
38fa182feb55        centos              "/bin/sh -C 'while t…"   25 hours ago        Exited (127) 25 hours ago                        crazy_turing
95c90efd4fd1        centos              "/bin/bash"              39 hours ago        Exited (0) 39 hours ago                          brave_brahmagupta
e16e11186028        centos              "/bin/bash"              2 days ago          Exited (0) 8 seconds ago                         angry_banach
7fd19a2cb21e        hello-world         "/hello"                 2 days ago          Exited (0) 2 days ago                            eager_varahamihira

# 将文件拷贝出来
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker cp e16e11186028:/home/test.java /home
[root@iz2ze4t93bwwn1hyg068wpz ~]# cd /home
[root@iz2ze4t93bwwn1hyg068wpz ~]# cd /home
[root@iz2ze4t93bwwn1hyg068wpz home]# ls
kxxy.java  test.java  www

# 拷贝是一个手动过程，未来我们使用-v卷计数，可以实现打通
```

### 作业

> 作业1：Docker部署nginx

```shell
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker search nginx
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                              Official build of Nginx.                        14133               [OK]                
jwilder/nginx-proxy                Automated Nginx reverse proxy for docker con…   1921                                    [OK]
richarvey/nginx-php-fpm            Container running Nginx + PHP-FPM capable of…   796                                     [OK]
linuxserver/nginx                  An Nginx container, brought to you by LinuxS…   131                                     
jc21/nginx-proxy-manager           Docker container for managing Nginx proxy ho…   118                                     
tiangolo/nginx-rtmp                Docker image with Nginx using the nginx-rtmp…   106                                     [OK]
bitnami/nginx                      Bitnami nginx Docker Image                      91                                      [OK]
alfg/nginx-rtmp                    NGINX, nginx-rtmp-module and FFmpeg from sou…   81                                      [OK]
jlesage/nginx-proxy-manager        Docker container for Nginx Proxy Manager        75                                      [OK]
nginxdemos/hello                   NGINX webserver that serves a simple page co…   64                                      [OK]
nginx/nginx-ingress                NGINX Ingress Controller for Kubernetes         46                                      
privatebin/nginx-fpm-alpine        PrivateBin running on an Nginx, php-fpm & Al…   43                                      [OK]
nginxinc/nginx-unprivileged        Unprivileged NGINX Dockerfiles                  27                                      
schmunk42/nginx-redirect           A very simple container to redirect HTTP tra…   19                                      [OK]
staticfloat/nginx-certbot          Opinionated setup for automatic TLS certs lo…   16                                      [OK]
centos/nginx-112-centos7           Platform for running nginx 1.12 or building …   15                                      
nginx/nginx-prometheus-exporter    NGINX Prometheus Exporter                       15                                      
raulr/nginx-wordpress              Nginx front-end for the official wordpress:f…   13                                      [OK]
centos/nginx-18-centos7            Platform for running nginx 1.8 or building n…   13                                      
flashspys/nginx-static             Super Lightweight Nginx Image                   8                                       [OK]
mailu/nginx                        Mailu nginx frontend                            8                                       [OK]
bitwarden/nginx                    The Bitwarden nginx web server acting as a r…   7                                       
bitnami/nginx-ingress-controller   Bitnami Docker Image for NGINX Ingress Contr…   7                                       [OK]
ansibleplaybookbundle/nginx-apb    An APB to deploy NGINX                          1                                       [OK]
wodby/nginx                        Generic nginx                                   1                                       [OK]
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
6ec7b7d162b2: Pull complete 
bbce32568f49: Pull complete 
5928664fb2b3: Pull complete 
a85e904c7548: Pull complete 
ac39958ca6b1: Pull complete 
Digest: sha256:31de7d2fd0e751685e57339d2b4a4aa175aea922e592d36a7078d72db0a45639
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              7baf28ea91eb        4 days ago          133MB
centos              latest              300e315adb2f        7 days ago          209MB
mysql               5.7                 ae0658fdbad5        3 weeks ago         449MB
mysql               latest              dd7265748b5d        3 weeks ago         545MB
hello-world         latest              bf756fb1ae65        11 months ago       13.3kB

# 运行测试，-d后台运行，--name起名字，-p暴露端口，3344为外部端口，可以从3344访问容器端口80
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker run -d --name nginx01 -p:3344:80 nginx
3bff58e5c530bf9f833e4927b605afcccfa1a6cf1c7cb2c30a4016bd8c439829
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
3bff58e5c530        nginx               "/docker-entrypoint.…"   6 seconds ago       Up 6 seconds        0.0.0.0:3344->80/tcp   nginx01
# 内网访问
[root@iz2ze4t93bwwn1hyg068wpz ~]# curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 进入容器
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker exec -it nginx01 /bin/bash
root@3bff58e5c530:/# whereis eginx
eginx:
root@3bff58e5c530:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@3bff58e5c530:/# cd /etc/ngin
bash: cd: /etc/ngin: No such file or directory
root@3bff58e5c530:/# cd /etc/nginx
root@3bff58e5c530:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf
```

> 作业2：docker安装一个tomcat

```shell
# 官方的使用
docker run -it --rm tomcat:9.0

# 我们之前的启动都是后台，停止了容器之后，容器还是可以查到。 docker run -it --rm 一般用来测试，用完即删除容器，但是镜像还在。

# 下载再启动
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
Digest: sha256:f728ca177fee0851aea29499fbb2013737231a00264f517cc3d185f6f8bf09a8
Status: Downloaded newer image for tomcat:latest
docker.io/library/tomcat:latest
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              9.0                 6d15a1d68603        3 days ago          649MB
tomcat              latest              6d15a1d68603        3 days ago          649MB
nginx               latest              7baf28ea91eb        5 days ago          133MB
centos              latest              300e315adb2f        8 days ago          209MB
mysql               5.7                 ae0658fdbad5        3 weeks ago         449MB
mysql               latest              dd7265748b5d        3 weeks ago         545MB
hello-world         latest              bf756fb1ae65        11 months ago       13.3kB
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker run -d -p 3355:8080 --name tomcat01 tomcat
150a71744fa8c44fe6399cf1d93dbf7f34a497559550c2ab2da7070cf254f7db

# 测试访问没有问题，报404错误

# 进入容器
[root@iz2ze4t93bwwn1hyg068wpz ~]# docker exec -it tomcat01 /bin/bash
root@150a71744fa8:/usr/local/tomcat# ls
BUILDING.txt  CONTRIBUTING.md  LICENSE	NOTICE	README.md  RELEASE-NOTES  RUNNING.txt  bin  conf  lib  logs  native-jni-lib  temp  webapps  webapps.dist  work
root@150a71744fa8:/usr/local/tomcat# ls -al
total 176
drwxr-xr-x 1 root root  4096 Dec 12 17:40 .
drwxr-xr-x 1 root root  4096 Dec 16 08:10 ..
-rw-r--r-- 1 root root 18982 Dec  3 11:48 BUILDING.txt
-rw-r--r-- 1 root root  5409 Dec  3 11:48 CONTRIBUTING.md
-rw-r--r-- 1 root root 57092 Dec  3 11:48 LICENSE
-rw-r--r-- 1 root root  2333 Dec  3 11:48 NOTICE
-rw-r--r-- 1 root root  3257 Dec  3 11:48 README.md
-rw-r--r-- 1 root root  6898 Dec  3 11:48 RELEASE-NOTES
-rw-r--r-- 1 root root 16507 Dec  3 11:48 RUNNING.txt
drwxr-xr-x 2 root root  4096 Dec 12 17:41 bin
drwxr-xr-x 1 root root  4096 Dec 16 08:10 conf
drwxr-xr-x 2 root root  4096 Dec 12 17:40 lib
drwxrwxrwx 1 root root  4096 Dec 16 08:10 logs
drwxr-xr-x 2 root root  4096 Dec 12 17:40 native-jni-lib
drwxrwxrwx 2 root root  4096 Dec 12 17:40 temp
drwxr-xr-x 2 root root  4096 Dec 12 17:40 webapps
drwxr-xr-x 7 root root  4096 Dec  3 11:45 webapps.dist
drwxrwxrwx 2 root root  4096 Dec  3 11:43 work
root@150a71744fa8:/usr/local/tomcat# cd webapps
root@150a71744fa8:/usr/local/tomcat/webapps# ls
root@150a71744fa8:/usr/local/tomcat/webapps# 

# 发现问题：1.linux命令少了  2.没有webapps.
# 原因：阿里云镜像问题，默认是最小的镜像，所有不必要的都删除掉，保证运行即可
root@150a71744fa8:/usr/local/tomcat/webapps.dist# ls
ROOT  docs  examples  host-manager  manager
root@150a71744fa8:/usr/local/tomcat/webapps.dist# cd ..
root@150a71744fa8:/usr/local/tomcat# cp webapps.dist/* webapps
cp: -r not specified; omitting directory 'webapps.dist/ROOT'
cp: -r not specified; omitting directory 'webapps.dist/docs'
cp: -r not specified; omitting directory 'webapps.dist/examples'
cp: -r not specified; omitting directory 'webapps.dist/host-manager'
cp: -r not specified; omitting directory 'webapps.dist/manager'
root@150a71744fa8:/usr/local/tomcat# cp -r webapps.dist/* webapps
root@150a71744fa8:/usr/local/tomcat# cd webapps
root@150a71744fa8:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager
root@150a71744fa8:/usr/local/tomcat/webapps# 
# 发现webapps.dist里有，复制到webapps内，刷新页面即可
```

> 作业3：部署es+kibana

```shell
# es 暴露的端口很多！
# es 十分的耗内存
# es 的数据一般需要放置到安全目录，挂载

# 下载启动
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

# 启动后服务器就卡住了 docker stats 查看cpu的状态

# 停止整个docker

# 加上内存限制，修改配置  -e 环境配置修改
docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
[root@iz2ze4t93bwwn1hyg068wpz ~]# curl localhost:9200
{
  "name" : "1d4ca35f2866",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "q0DNf6u9TAm3GGXM9j2pTQ",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
# 访问成功
```

> 作业4：用kibana连接es

### 可视化

* portainer(先用这个)

  ```shell
  docker run -d -p 8089:9000 \
  --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
  ```

* Rancher(CI/CD再用)

**什么是portainer？**

Dokcer图形化管理工具！提供一个后台面板供我们操作

```shell
docker run -d -p 8089:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问测试：外网ip:8089

可以通过它来访问了

## Docker镜像讲解

### 镜像是什么

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

所有的应用，直接打包docker镜像，就可以直接跑起来！

如何得到镜像：

* 远程仓库下载
* 朋友copy给你
* 自己制作一个镜像DockerFile

### Docker镜像加载原理

> UnionFS(联合文件系统)

联合文件系统（[UnionFS](https://en.wikipedia.org/wiki/UnionFS)）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。
联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

> Docker镜像加载原理

docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system) 主要包含bootloader和kernel，bootloader 主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就存在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。



roorfs （root file system），在bootfs之上。包含的就是典型Linux系统中的 /dev ，/proc，/bin ，/etx 等标准的目录和文件。rootfs就是各种不同的操作系统发行版。比如Ubuntu，Centos等等。

平时我们安装虚拟机的CentOS都是好几个G，为什么docker这里才200M？



![在这里插入图片描述](https://img-blog.csdnimg.cn/20190522121705397.png)



对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供rootfs就行了。由此可见对于不同的linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以共用bootfs。

虚拟机是分钟级别，容器是秒级！！！

### 分层理解

> 分层的镜像

我们可以去下载一个镜像，注意观察下载的日志输出，可以看到是一层一层的在下载！

**为什么docker镜像要采用这种分层结构呢？**

最大的一个好处就是**共享资源**
比如：有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像,同时内存中也只需加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

> 特点

Docker的镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部！

这一层就是我们通常说的容器层，容器之下的都叫镜像层！



**所有用户的操作基于容器层**

如何提交一个自己的镜像？

### Commit镜像

```shell
docker commit 提交容器成为一个新的副本
# 命令和git原理类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

实战：

```shell
# 启动一个默认的tomcat

# 默认的tomcat没有webapps应用，镜像的原因

# 自己拷贝进去基本的文件

# 将操作过的容器通过commit提交为一个镜像，我们以后就使用修改过的镜像即可。
```

![image-20201217132738564](C:\Users\kxxy\AppData\Roaming\Typora\typora-user-images\image-20201217132738564.png)

```shell
如果你想要保存当前容器的状态，就可以通过commit来提交，获得一个镜像，就好比学习VM的时候的快照。
```

## 容器数据卷

### 什么是容器数据卷

程序要保存数据？如果数据都在容器中，容器删除，数据就会丢失。**需求：数据可以持久化**

MySQL，容器删了=删库跑路，**需求：MySQL数据可以存储在本地**

容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地！

这就是卷技术！实质是目录的挂在，将我们容器内的目录，挂载到Linux上面！

**总结一句话：容器的持久化和同步化操作！容器间也是可以数据共享的！**

### 使用数据卷

> 方式一：直接使用命令来挂载 -v

```shell
docker run -it -v 主机目录:容器内目录

[root@iz2ze4t93bwwn1hyg068wpz home]# docker run -it -v /home/ceshi:/home centos /bin/bash

# 启动起来之后我们可以通过docker inspect 容器id后可以看到有这个Mounts说明挂载成功。
```

![image-20201217134613268](C:\Users\kxxy\AppData\Roaming\Typora\typora-user-images\image-20201217134613268.png)

测试文件的同步

![image-20201217135003414](C:\Users\kxxy\AppData\Roaming\Typora\typora-user-images\image-20201217135003414.png)

类似Linux的主从复制，双向绑定的过程，哪怕停止容器在本地修改的文件容器内的文件依旧被修改。

好处：我们以后修改只需要在本地修改即可，容器内会自动同步。

### 实战：安装MySQL

思考：MySQL的数据持久化的问题！（会占用两倍存储）

```shell
# 获取镜像
docker pull mysql:5.7

# 运行容器，需要做数据挂载！
# 安装启动MYSQL，需要配置密码的，这是要注意的！
# 官方测试：docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动我们的mysql
-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境配置
--name 容器名字
[root@iz2ze4t93bwwn1hyg068wpz ceshi]# docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Wxy5211314 --name mysql01 mysql:5.7
1d712015ff5125638eeeead35d27d9b1a6b9ec9c3afc3c8ce981a8a515bc9ce8

# 启动成功之后，我们在本地使用工具连接测试一下

# 在本地创建一个数据库，在本地文件夹出现数据库，测试成功
```

![image-20201217140633706](C:\Users\kxxy\AppData\Roaming\Typora\typora-user-images\image-20201217140633706.png)

假设我们将容器删除，我们发现挂载到本地的数据卷依旧没有丢失，实现了持久化功能。



### 具名和匿名挂载

```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx
# -P随机端口，-v不指定主机路径

# volume 查看容器内的情况
docker volume ls

# 这里发现，这种就是匿名挂载，没有名字，只写了容器内路径，没写容器外的路径。

# 通过 -v 卷名：容器内路径实现具名
# 查看以下这个卷
```

所有的docker容器内的卷，没有指定目录的情况下都是在 **/var/lib/docker/volumes/xxx/_data**  里。

我们通过具名挂载可以方便的找到我们的一个卷，大多数情况在使用 具名挂载

```shell
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载！
-v 容器内路径            #匿名挂载
-v 卷名：容器内路径       #具名挂载
-v /宿主机路径：容器内路径 #指定路径挂载
```

拓展：

```shell
# 通过ro rw改变读写权限
ro # 只读，只能从外部改变
rw # 可读可写，内部外部都可以改变
# 一旦设置了这个容器权限，容器对我们挂载出来的内容就有限定了！
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx
```

## DockerFile



## Docker 网络