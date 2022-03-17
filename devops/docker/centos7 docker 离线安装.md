# centos7-docker离线安装

## 一.下载离线安装包

> 问地址: [docker离线下载地址](https://download.docker.com/linux/static/stable/x86_64/)
>
> 选择对应版本,此处选择docker-20-10.8.tgz

![avatar](../../images/devops/docker/docker_version_selected.png)

可以下载到本地后上传至服务器，或者在服务器执行以下命令直接下载

```shell
yum -y install wget && wget https://download.docker.com/linux/static/stable/x86_64/docker-20.10.8.tgz
```

## 二、解压安装

执行以下命令，对第一步下载的`docker-20.10.8.tgz`进行解压

```shell
tar -zxvf docker-20.10.8.tgz
```

解压后会得到一个`docker`文件夹

![avatar](../../images/devops/docker/docker_extract.png)

执行以下命令，将docker命令添加到环境变量

```shell
# 拷贝docker文件到制定目录
$ cp docker/* /usr/bin
```

验证

输入以下命令，验证是否生效

```shell
$ docker version
```

![avatar](../../images/devops/docker/docker_test.png)

出现上述则说明安装成功，最下面的报错可以忽略，因为目前docker还没启动

## 三.启动docker

需要配置docker.service

docker.service内容如下

```shell
$ cat > docker.service << EOF
> [Unit]
> Description=Docker Application Container Engine
> Documentation=https://docs.docker.com
> After=network-online.target firewalld.service containerd.service
> Wants=network-online.target
> 
> [Service]
> Type=notify
> ExecStart=/usr/bin/dockerd
> ExecReload=/bin/kill -s HUP $MAINPID
> TimeoutSec=0
> RestartSec=2
> Restart=always
> StartLimitBurst=3
> StartLimitInterval=60s
> LimitNOFILE=infinity
> LimitNPROC=infinity
> LimitCORE=infinity
> TasksMax=infinity
> Delegate=yes
> KillMode=process
> 
> [Install]
> WantedBy=multi-user.target
> EOF
```

移动docker.service到如下目录

```shell
$ mv docker.service /usr/lib/systemd/system/
```

重新加载配置

```shell
$ systemctl daemon-reload
```

启动docker

```shell
$ systemctl start docker 
```

验证

输入以下命令，即可查看docker是否启动

```shell
$ systemctl status docker 
```

![avatar](../../images/devops/docker/docker_status.png)

启动成功，到此docker安装结束!