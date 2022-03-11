# Harbor问题排查-unknown blob

## docker push 报错 unknown blob

### 1.背景描述

> 目前公司在做harbor私服迁移，需要重新搭建一套harbor，环境如下
>
> 系统: centos7.8
>
> Docker: 20.10.8
>
> Harbor: v1.10.10
>
> Harbor部署方式：通过官方提供的安装包进行安装
>
> 环境配置好以后，会提供一个外部域名，实际是通过nginx作为负载均衡来访问Harbor私服仓库

### 2.Harbor配置

> Harbor.yml 文件只修改了两个参数,参数如下

```yaml
hostname: registry.bitmart.run
external_url: https://registry.bitmart.run
```

> 其余配置均未动

### 3.问题描述

> 本地往私服镜像推送镜像，出现如下错误

![avatar](../../images/devops/harbor/docker_push_unknown_blob.png)

> 查看habor的nginx配置，发下了如下报错

![avatar](../../images/devops/harbor/harbor_nginx_err.png)

> docker push 会出现404问题

### 4.问题解决

**总结为三步**

- 1、修改`harbor.yml`

- 2、修改`nginx.conf`

- 3、重启`nginx`

  

**第一步:** 修改harbor.yml

结合两项报错，到Google上查阅，需要修改`harbor.yml`的 `relativeurls`值设为`true`，如下所示

```yaml
relativeurls: true # 默认为false
```

**注:**如果没有该值，就手动加上



**第二步:**修改`nginx.conf`

接下来需要对`harbor`的`nginx`配置做修改(路径在`harbor`同级目录下的`commom/config/nginx/`下)

**注:**配置文件里说的比较清楚，

``` When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.```

```text
proxy_set_header X-Forwarded-Proto $scheme;
```

大意就是，如果我们设置了例如nginx的反向代理，如果有`proxy_set_header X-Forwarded-Proto $scheme;`类似配置时，请删掉。

因此，修改`nginx.conf`如下

```nginx
worker_processes auto;
pid /tmp/nginx.pid;

events {
  worker_connections 1024;
  use epoll;
  multi_accept on;
}

http {
  client_body_temp_path /tmp/client_body_temp;
  proxy_temp_path /tmp/proxy_temp;
  fastcgi_temp_path /tmp/fastcgi_temp;
  uwsgi_temp_path /tmp/uwsgi_temp;
  scgi_temp_path /tmp/scgi_temp;
  tcp_nodelay on;

  # this is necessary for us to be able to disable request buffering in all cases
  proxy_http_version 1.1;

  upstream core {
    server core:8080;
  }

  upstream portal {
    server portal:8080;
  }

  log_format timed_combined '$remote_addr - '
    '"$request" $status $body_bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '$request_time $upstream_response_time $pipe';

  access_log /dev/stdout timed_combined;

  server {
    listen 8080;
    server_tokens off;
    # disable any limits to avoid HTTP 413 for large image uploads
    client_max_body_size 0;

    # Add extra headers
    add_header X-Frame-Options DENY;
    add_header Content-Security-Policy "frame-ancestors 'none'";

    # costumized location config file can place to /etc/nginx/etc with prefix harbor.http. and suffix .conf
    include /etc/nginx/conf.d/harbor.http.*.conf;

    location / {
      proxy_pass http://portal/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
      #proxy_set_header X-Forwarded-Proto $scheme;
      
      proxy_buffering off;
      proxy_request_buffering off;
    }

    location /c/ {
      proxy_pass http://core/c/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
      #proxy_set_header X-Forwarded-Proto $scheme;
      
      proxy_buffering off;
      proxy_request_buffering off;
    }

    location /api/ {
      proxy_pass http://core/api/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
      #proxy_set_header X-Forwarded-Proto $scheme;
      
      proxy_buffering off;
      proxy_request_buffering off;
    }

    location /chartrepo/ {
      proxy_pass http://core/chartrepo/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
      #proxy_set_header X-Forwarded-Proto $scheme;
      
      proxy_buffering off;
      proxy_request_buffering off;
    }

    location /v1/ {
      return 404;
    }

    location /v2/ {
      proxy_pass http://core/v2/;
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
      #proxy_set_header X-Forwarded-Proto $scheme;
      proxy_buffering off;
      proxy_request_buffering off;

      proxy_send_timeout 900;
      proxy_read_timeout 900;
    }

    location /service/ {
      proxy_pass http://core/service/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
      #proxy_set_header X-Forwarded-Proto $scheme;

      proxy_buffering off;
      proxy_request_buffering off;
    }

    location /service/notifications {
      return 404;
    }
  }
}
```

接下来就重启`nginx`

**第三步:**重启`nginx`

需要进入`nginx`容器重启即可，命令如下

```shell
docker exec -it {HARBOR_NGINX_NAME} nginx -s reload
```

本环境容器名为`nginx`，所以执行一下命令重启

```shell
docker exec -it nginx nginx -s reload 
```

### 5.问题验证

在终端，再推送镜像，提示成功，如下图所示

![avatar](../../images/devops/harbor/docker_push_success.png)

验证成功，至此，问题处理完毕！

[参考链接](https://lcc3108.github.io/articles/2020-11/harbor-in-proxy-required-setting.html)