# Centos7下Jdk1.8安装与配置



## 一、获取`Jdk`安装包

> 链接: https://pan.baidu.com/s/1LP1te21pe50DYo2-kKwtuA 
>
> 提取码: ki9k 

## 二、安装与配置

**2.1安装**

对下载的文件解压到指定目录

```shell
tar -zxvf jdk-8u161-linux-x64.tar.gz -C /usr/local
```

对解压后的文件做下软链接

```shell
ln -s /usr/local/jdk1.8.0_161 /usr/local/java
```

**2.2 配置环境变量**