# Shark On The Way

来公司才两周就遇到架构调整，[新手任务](./freshman_task.md) 是之前组长给的一个学习大纲，9 大条每周作为一个 task 去挑战一下，也算是接下来三个月的周末安排吧。(当然第一周就病了，也是计划之外的悲哀吧~)

## Week.0: Docker

阅读材料:

* [Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
* [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/)

### Docker 入门教程

* Docker 目标: 解决环境配置的难题。

* 虚拟机

  > 应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。

  缺点: 资源占用多 | 冗余步骤多 | 启动慢

* Linux 容器 (Linux Containers, LXC)

  > Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。

  与虚拟机相比: 容器里的应用是底层系统的进程非虚拟机内部进程(启动快) | 只占用所需资源，虚拟机独享资源(资源占用少) | 只包含要用到的组件基础，虚拟机打包整个操作系统(体积小)

* Docker 是什么

  > Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。

* Docker 的用途

  1.  提供一次性的环境(本地测试、持续集成)
  1.  提供弹性的云服务(随开随关，动态扩容)
  1.  组建微服务架构(多容器)

* Docker 的使用

  1.  验证安装 `docker version` | `docker info`，Docker 是服务器-客户端架构，通过 `service` 或 `systemctl` 启动服务。
  1.  启用 mirror (Performance - Deamon) | `docker-machine create --engine-registry-mirror={--link} -d virtualbox default`
  1.  Docker 把应用程序及其依赖，打包在 image 文件里面。通过 image 生成 Docker 容器。image 类似模版文件，可生成多个同时运行的容器实例。

* hello world

  ```bash
  # 从仓库抓取到本地 [文件组]/[文件名][:tag]
  $ docker image pull library/hello-world
  # 列出本机的所有 image 文件
  $ docker image ls
  # 运行 image (若无则自动抓取)
  $ docker container run hello-world
  ```

  某些容器提供服务不会自动终止，使用 `docker container kill [containID]` 终止。

* 容器文件

  > image 文件生成的 **容器实例** ，本身也是一个文件，称为 **容器文件** 。

  容器生成后，会同时存在 image 文件和容器文件，关闭后并不删除容器文件。可以通过 `docker container ls --all` 查看，通过 `docker container rm [containID]` 删除。

* Dockerfile 文件(用来配置 image 的文本文件)

* 实例

  1.  配置 `.dockerignore` 文件
  2.  配置 `Dockerfile` 文件

  ```
  FROM node:8.4
  COPY . /app
  WORKDIR /app
  RUN npm install --registry=https://registry.npm.taobao.org
  EXPOSE 3000
  ```

  * `FROM node:8.4` 表示该 image **继承** 自官方的 node image
  * `COPY . /app` 拷贝文件(忽略 `.dockerignore` )
  * `WORKDIR /app` 指定工作路径
  * `RUN npm install` 在 /app 目录下安装依赖(安装后将打包进入 image)
  * `EXPOSE 3000` 将容器 3000 端口暴露，允许外部连接

  3.  创建 image 文件

  ```bash
  # [:0.0.1] tag 默认 latest
  $ docker image build -t koa-demo[:0.0.1] .
  ```

  4.  生成容器

  ```bash
  $ docker container run -p 8000:3000 -it koa-demo /bin/bash
  ```

  * `-p 8000:3000` 容器的 3000 端口映射到本机的 8000 端口
  * `it` 容器的 Shell 映射到当前的 Shell
  * `koa-demo` image
  * `/bin/bash` 容器启动后执行的命令 (启动 Bash)
  * `-rm` 运行后自动删除

  5.  CMD 命令

  在 Dockerfile 里面添加 `CMD node demos/01.js` 表示容器启动后自动执行 `node demos/01.js`， `RUN` 在构建阶段执行，将结果打包入 image 文件，`CMD` 在容器启动后执行且只能有一个，并且指定 `CMD` 后 `docker container run` 不能附加命令。

  6.  发布 image

  ```bash
  # 登陆
  $ docker login
  # 标注
  $ docker image tag [imageName] [username]/[repository]:[tag]
  # 或重新构建
  $ docker image build -t [username]/[repository]:[tag] .
  # 发布
  $ docker image push [username]/[repository]:[tag]
  ```

* 其他命令

  ```bash
  # run 新建 start 复用已有
  $ docker container start [containerID]
  $ docker container stop [containerID]
  # 没有 -it 时 使用 logs 参看输出
  $ docker container logs [containerID]
  # 进入正在运行的 docker
  $ docker container exec -it [containerID] /bin/bash
  # 从 docker 内部拷贝文件
  $ docker container cp [containID]:[/path/to/file] .
  ```
