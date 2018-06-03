# Docker 基础

Docker公司公有仓库：Docker Hub,https://hub.docker.com

Docker中文文档：http://www.dockerinfo.net/

## Docker基本操作

- 启动容器

  - 运行一次后退出

    ```
    docker run IMAGE [COMMAND] [ARG]
    ```

  - 启动交互式容器

    ```
    docker run -i -t IMAGE /bin/bash
    ```

    -i,--interactive=true|false，默认是false，告诉docker守护进程，为容器始终打开标准输入

    -t,--tty=true|false，默认是false，为创建的容器分配一个伪tty终端，这样新创建的容器才能提供交互式shell

    --neme=[CONTAINER_NAME]，给创建的容器指定名字

  - 启动守护式容器

    ```
    docker run -i -t -d IMAGE [COMMAND] [ARG]
    ```

    -d，在启动容器时，以后台的方式执行命令

    -P，--publish-all=true|false，默认false，为容器暴露的所有端口进行映射

    -p，--publish=[]，指定映射的端口

- 查看建立的容器

  ```
  docker ps -al
  ```

  不指定参数，查看正在运行的docker

  -a,列出所有的容器

  -l,列出最新创建的容器

- 查看已经建立的容器

  ```
  docker inspect [CONTAINER_ID/CONTAINER_NAME]
  ```

  返回容器的配置信息

- 重新启动容器

  ```
  docker start -i [CONTAINER_NAME]
  ```

- 停止容器

  ```
  docker stop [CONTAINER_NAME]
  ```

  ```
  docker kill [CONTAINER_NAME]
  ```

- 删除容器

  ```
  docker rm [CONTAINER_NAME]
  ```

  删除已经停止的容器

- 查看容器日志

  ```
  docker logs -f -t --tail [CONTAINER_NAME]
  ```

  -f,--follows=true|false，默认false，一直跟踪日志的变化，来返回结果

  -t,--timestamps=true|false，默认false，在返回结果上加上时间戳

  --tail="all"，选择从结尾开始算，日志的数量，如果不指定，返回所有日志

- 查看容器内的进程

  ```
  docker top [CONTAINER_NAME]
  ```

- 在运行的容器内启动新的进程

  ```
  docker exec [-d] [-i] [-t] [CONTAINER_NAME] [COMMAND] [ARG]
  ```

  ​