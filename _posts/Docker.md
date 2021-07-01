# Docker

> Docker学习

- Docker概述
- Docker安装
- Docker命令
  - 镜像命令
  - 容器命令
  - 操作命令
  - ...
- Docker镜像
- 容器数据卷
- DockerFile
- Docker网络原理



> 使用教程

#### Docker安装

主要参考资料

```
#docker文档
https://docs.docker.com/docker-hub/
#大佬博客
https://yeasy.gitbook.io/docker_practice/install/
#踩坑
https://www.cnblogs.com/caidingyu/p/10576194.html
```

#### Docker使用

```shell
#hello world
docker run hello-world

#hello world 运行流程
开始->docker判断本地是否有->有则运行，无则在docker hub下载


#查看下载的hello-world镜像
docker images
(base) mozi@mozi:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        8 months ago        13.3kB

#卸载
1. 卸载Docker Engine，CLI和Containerd软件包：
$ sudo apt-get purge docker-ce docker-ce-cli containerd.io
2.主机上的映像，容器，卷或自定义配置文件不会自动删除。要删除所有图像，容器和卷：
$ sudo rm -rf /var/lib/docker
#/var/lib/docker的默认路径
```

镜像加速器

```shell
sudo mkdir -p /etc/docker
#加镜像地址
sudo tee /etc/docker/daemon.json <<-'EOF'
{
 "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
EOF
重启
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### Docker底层原理

Docker是一个Client - Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问!
DockerServer接收到Docker-Client的指令，就会执行这个命令!

#### Docker的常用命令

##### 镜像命令

```shell
#帮助命令

docker version  	 #显示docker的版本信息
docker info     	 #显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help	#万能命令

#镜像命令
docker images --help
#可选项
-a 所有
-q ID

#镜像搜索
docker search
(base) mozi@mozi:~$ docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   9925                [OK]
mariadb                           MariaDB is a community-developed fork of MyS…   3631                [OK]


#镜像拉取
docker pull 镜像名[:tag]
docker pull --help

Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is multi-platform capable
  -q, --quiet                   Suppress verbose output

docker pull  mysql 
Using default tag: latest
latest: Pulling from library/mysql   #不写tag默认最新版本
bf5952930446: Pull complete #分层下载，docker iamge的核心联合文件系统
8254623a9871: Pull complete
938e3e06dac4: Downloading
ea28ebf28884: Downloading
f3cef38785c2: Downloading
894f9792565a: Downloading
1d8a57523420: Downloading
6c676912929f: Download complete
3cdd8ff735c9: Download complete
4c70cbe51682: Download complete
e21cf0cb4dc3: Download complete
28c36cd3abcc: Download complete
latest: Pulling from library/mysql
bf5952930446: Downloading complete
8254623a9871: Download complete
938e3e06dac4: Download complete
ea28ebf28884: Download complete
f3cef38785c2: Download complete
894f9792565a: Download complete
1d8a57523420: Download complete
6c676912929f: Download complete
3cdd8ff735c9: Download complete
4c70cbe51682: Download complete
e21cf0cb4dc3: Download complete
28c36cd3abcc: Download complete
latest: Pulling from library/mysql
bf5952930446: Pull complete
8254623a9871: Pull complete
938e3e06dac4: Pull complete
ea28ebf28884: Pull complete
f3cef38785c2: Pull complete
894f9792565a: Pull complete
1d8a57523420: Pull complete
6c676912929f: Pull complete
3cdd8ff735c9: Pull complete
4c70cbe51682: Pull complete
e21cf0cb4dc3: Pull complete
28c36cd3abcc: Pull complete
Digest: sha256:6ded54eb1e5d048d8310321ba7b92587e9eadc83b519165b70bbe47e4046e76a  #签名
Status: Downloaded newer image for mysql:latest	#真实地址
docker.io/library/mysql:latest

docker pull mysql
docker pull docker.io/library/mysql:last	#两者等价

#镜像删除
docker rmi -f  #rmi remove image
docker rmi -f  容器ID 容器ID
docker rmi -f $(docker images -aq)

```

##### 容器命令

说明∶我们有了镜像才可以创建容器，linux，下载一个centos镜像来测试学习

```shell
docker pull centos

#新建容器并启动
docker run [可选参数] image

#参数说明
--name="Name"  		 	容器名字
-d						后台运行的方式，ja nohup
-it						使用交互方式运行，进入容器查看内容
-p						指定容器的端口
-p ip:8080:8080
-p 8080:8080			主机端口映射到容器端口
-P						大写p随机指定端口

#启动并进入容器
(base) mozi@mozi:~$ docker run -it centos /bin/bash
[root@30cf952075f1 /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@30cf952075f1 /]#
[root@30cf952075f1 /]# cat /etc/redhat-release
CentOS Linux release 8.2.2004 (Core)

#退出
exit	#容器停止且退出
ctrl+P+Q#容器不停止退出

#查看正在运行的image
docker ps
docker ps -a	#查看所有曾运行的image

#删除容器
docker rm -f 
docker rm $(docker ps -aq)
docker ps -a -q|xargs docker rm

#启动和停止容器的操作
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id
```

后台启动容器

```shell
#命令docker run -d镜像名!
[root@kuangshen/]# docker run -d centos
#问题docker ps，发现centos停止了
#常见的坑:docker容器使用后台运行，就必须要有要一个前台进程，docker发现没有应用，就会自动停止
# nginx，容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
```

查看日志

```shell
docker logs -f -t --tail容器，没有日志
#自己编写一段she77脚本
[root@kuangshen/]# docker run -d centos /bin/sh -c "while true;do echo kuangshen;sleep 1; done"
# [root@kuangshen/]# docker ps
CONTAINER ID	IMAGE
dce7b86171bf	centos
#显示日志
-tf 		#显示日志
--tail number	#要显示日志条数
[root@kuangshen/]# docker 1ogs -tf --tail 10 dce7b86171bf
```

查看容器中进程信息ps

```shell
#命令 docker top 容器id
#命令docker top容器id
[root@kuangshen/]# docker top dce7b86171bf
```

**查看容器的元数据**

```shell
(base) mozi@mozi:~$ docker inspect 55a3216317f2
[
    {
        "Id": "55a3216317f2bc0f1485e4ec5ebde89181e7320b5a8d0396fe220f96f2693631",
        "Created": "2020-09-19T09:36:58.925339521Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 10135,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-09-19T09:37:00.628149916Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:0d120b6ccaa8c5e149176798b3501d4dd1885f961922497cd0abef155c869566",
        "ResolvConfPath": "/var/lib/docker/containers/55a3216317f2bc0f1485e4ec5ebde89181e7320b5a8d0396fe220f96f2693631/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/55a3216317f2bc0f1485e4ec5ebde89181e7320b5a8d0396fe220f96f2693631/hostname",
        "HostsPath": "/var/lib/docker/containers/55a3216317f2bc0f1485e4ec5ebde89181e7320b5a8d0396fe220f96f2693631/hosts",
        "LogPath": "/var/lib/docker/containers/55a3216317f2bc0f1485e4ec5ebde89181e7320b5a8d0396fe220f96f2693631/55a3216317f2bc0f1485e4ec5ebde89181e7320b5a8d0396fe220f96f2693631-json.log",
        "Name": "/vigorous_driscoll",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
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
                "LowerDir": "/var/lib/docker/overlay2/c76e3145a7d2b11c8125e8a094c1f01bc2264c5543e3efa5327777a14306b8cb-init/diff:/var/lib/docker/overlay2/ab544364cf519c23d59a2fa7d9537cc40e9a2510289ac60c70424ca47ffdeba9/diff",
                "MergedDir": "/var/lib/docker/overlay2/c76e3145a7d2b11c8125e8a094c1f01bc2264c5543e3efa5327777a14306b8cb/merged",
                "UpperDir": "/var/lib/docker/overlay2/c76e3145a7d2b11c8125e8a094c1f01bc2264c5543e3efa5327777a14306b8cb/diff",
                "WorkDir": "/var/lib/docker/overlay2/c76e3145a7d2b11c8125e8a094c1f01bc2264c5543e3efa5327777a14306b8cb/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "55a3216317f2",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20200809",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "5b0a2898845503194c9d79958c10b914356eeed4d1c1caff7a50b4576c381651",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/5b0a28988455",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "df9de8b49d5bf2d1266a0f1c5c6174d91b2e1b2409639128081bbfa76abeca78",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "a4975541ae355e8c370c736a6a46d6f2e580cc1347bbb8dfa9d7c3cabaab3ddd",
                    "EndpointID": "df9de8b49d5bf2d1266a0f1c5c6174d91b2e1b2409639128081bbfa76abeca78",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

