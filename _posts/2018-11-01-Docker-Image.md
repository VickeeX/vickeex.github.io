---
layout:     post
title:      Docker Image
subtitle:   
date:       2018-11-01
author:     VickeeX
header-img: img/post-bg-docker-logo-1.png
catalog: true
tags:
    - Docker
    - Image
    - Docker Image
---

对docker的操作主要是基于：镜像，容器，数据卷，网络，service/stack/swarm，registry等。因为我也是初学，所以应有的对以上几个概念的详细介绍（关联/结构等）会在之后我弄清楚了的时候再来补充上。接下来的系列基于docker提供的CLI命令进行相关介绍和用法解析，这次我们先说image!!

docker中，image即是container的静态实体，可类比为 {hello_world.py文件} 和 {对应的hello_world进程} 的关系。image包括了应用程序所依赖的环境，比如：软件或者包/库依赖等，例：python3.5。

###  Dockerfile
可以通过自编写Dockerfile而构建自己的镜像从而运行一个个性化的container，一个最基本的Dockerfile（from: https://docs.docker.com/get-started/part2/#apppy）如下：
```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
一个镜像会由多个层进行构建，从而具有层级结构，每一层代表着Dockerfile中的每一条指令。比如，上述代码中的“FROM python:2.7-slim”就会根据python:2.7-slim镜像构建一层，COPY命令则又根据当前目录增添一些文件，RUN则会使用 pip install安装所需的库。整个构建过程是“增量式”的，即每一层都是基于上一层的更改的集合。

所需安装的库有（requirements.txt）：
```
Flask
Redis
```
而app.py就是我们期望运行的应用，如下为一个简单的Web应用：
```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
### build
有了Dockerfile和“context“(Dockerfile所在的路径或URL)，运行以下命令即可构建镜像啦：
```
$ docker build -t YOUR_IMAGE_NAME ./Dockerfile
```
在docker build命令中，常用的参数选项如下表，另外还可以在构建时为镜像指定网络连接方式(--network [ bridge | none | container | host | <user-defined-network-name>]  )、指定id(--iidfile string)、不使用cache(--no-cache)、添加label元数据(--label)、压缩等等。

| 简写 | 完整参数选项 | 说明 |
| ------ | ------ | ------ |
| -c | --cpu-shares int | 共享的CPU (relative weight) |
| -f | --file string | Dockerfile目录名 (默认为'PATH/Dockerfile') |
| -m | --memory-swap string | 内存上限 |
| -q | --quiet | 静默模式，不打印中间输出，只在完成时输出构建好的镜像ID |
| -t | --tag value  | 镜像版本名，用以对同名的镜像进行区分：“name:tag” |

注：可单独使用docker image tag为指定的源镜像创建tag。


### push/pull
因为有镜像仓库，所以与git一样，我们也可以push/pull镜像。若不指定TAG则默认为“:latest”。
```
$docker push NAME[:TAG]
$docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

* **push**
  * 默认push到Docker Hub，也可指定其他仓库。
  * Docker daemon默认同时（并发）push五个层，也可通过更改daemon选项“--max-concurrent-uploads”来更改并发数。

* **pull**：
  * 默认从Docker Hub获取镜像，也可指定其他的registry，如 $ docker pull myregistry.local:5000/test/t-image
  * DIGEST，用于在不希望镜像被更新到更新的版本，而想使用镜像的某指定版本时。
  * 可以批量拉取，如$ docker pull --all-tags fedora

注：可使用docker commit以根据现有的container创建新镜像。registry相关操作会在后续文章介绍。
push/pull以外，Docker image还有save和load操作。save将镜像输出重定向以生成tar包，load则从输入流加载并重新存储镜像(基于tags)。
```
$ docker image save fedora > fedora-all.tar
$ docker load --input fedora.tar
```


### get information
查看当前本地已有的镜像可以用以下命令。其中，ls命令可以指定路径（不指定则默认为本地仓库）、指定镜像名或tag、查看某镜像之前/之后的所有镜像（-since / - before），还支持--tree或--dot参数以供不同的显示方式。
```
$ docker images
$ docker images IMAGE_NAME
$ docker image ls 
```
可以通过history命令查看一个镜像的创建过程：
```
$ docker history IMAGE_NAME 
      IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    2ac9d1098bf1        3 months ago        /bin/bash                                       241.4 MB            Added Apache
    88b42ffd1f7c        5 months ago        /bin/sh -c #(nop) ADD file:1fd8d7f9f6557cafc7   373.7 MB            
    c69cab00d6ef        5 months ago        /bin/sh -c #(nop) MAINTAINER Lokesh Mandvekar   0 B                 
    511136ea3c5a        19 months ago                                                       0 B                 Imported from -

```

### rm && prune
docker image rm 删除指定的镜像
docker image prune 删除未使用的镜像
