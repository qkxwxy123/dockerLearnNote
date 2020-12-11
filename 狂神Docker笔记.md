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

