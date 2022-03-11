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

**2.2 配置系统环境变量**

```shell
vim /etc/profile
```

在文件尾部添加如下内容

>export JAVA_HOME=/usr/local/java
>export JRE_HOME=$JAVA_HOME/jre
>export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
>export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

添加完后保存

执行以下命令，使环境变量生效
```shell
source /etc/profile
```

**2.3 验证**

输入一下命令来验证

```shell
```

