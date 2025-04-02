# Docker Study

## 参考资料

https://iphysresearch.github.io/blog/post/programing/docker-tutorial/

## 1.难题

### 1.1环境配置的难题

软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？

用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。

如果某些老旧的模块与当前环境不兼容，那就麻烦了。开发者常常会说：“它在我的机器可以跑了”（It works on my machine），言下之意就是，其他机器很可能跑不了。

环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。

### 1.2虚拟机

虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，
因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。

但是，这个方案有几个缺点。

1. 资源占用过多

虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。

2. 冗余步骤多

虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。

3. 启动慢

启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

## 2.为什么使用docker

1. 更高效的利用系统资源
由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker 对系统资源的利用率更高。无论是应用执行速度、内存损耗或者文件存储速度，都要比传统虚拟机技术更高效。

2. 更快速的启动时间
 Docker 容器应用，由于直接运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启动时间。

3. 一致的运行环境
由于开发环境、测试环境、生产环境不一致，导致有些 bug 并未在开发过程中被发现。而 Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性.

4. 持续交付和部署
使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过 Dockerfile 来进行镜像构建，并结合 持续集成(Continuous Integration) 系统进行集成测试，
而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合 持续部署(Continuous Delivery/Deployment) 系统进行自动部署。

5. 更轻松的迁移
由于 Docker 确保了执行环境的一致性，使得应用的迁移更加容易。

#### **p1 比较了虚拟机和docker的区别**


## 3.Docker 是什么？

1. Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。 它是目前最流行的 Linux 容器解决方案。

2. Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。


## 4.Docker 的用途
Docker 的主要用途，目前有三大类。

1. 提供一次性的环境。 比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
2. 提供弹性的云服务。 因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
3. 组建微服务架构。 通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## 5.实例：Hello-world

```shell 
# 抓取官方的 hello-world 镜像
$ docker image pull hello-world

# 查看
$ docker image ls

# 运行这个 image 文件
$ docker container run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

... ...
# (运行成功！)
```
_**注意，docker container run命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。因此，前面的docker image pull命令并不是必需的步骤。**_

## 6.容器文件

image 文件生成的容器实例，本身也是一个文件，称为容器文件。 也就是说，一旦容器生成，就会同时存在两个文件： image 文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。

```shell
# 列出本机正在运行的容器
$ docker container ls

# 列出本机所有容器，包括终止运行的容器
$ docker container ls --all

```

## 7.Dockerfile 文件
Dockerfile 文件，它是一个文本文件，用来配置 image。Docker 根据 该文件生成二进制的 image 文件。

### 示例

```shell
# 该 image 文件继承我自己的 gwave image，冒号表示标签，这里标签是2.0.0，即2.0.0版本的 gwave。
FROM iphysreserch/gwave:2.0.0

# 将当前目录下的所有文件(除了.dockerignore排除的路径),都拷贝进入 image 文件里微系统的/waveform目录
COPY . /waveform

# 指定接下来的工作路径为/waveform (也就是微系统的 pwd)
WORKDIR /waveform

# 定义一个微系统里的环境变量
ENV VERSION=2.0.0	# optional

# 将容器 3000 端口暴露出来， 允许外部连接这个端口
EXPOSE 3000			# optional

# 在/waveform目录下，运行以下命令更新系统程序包。注意，安装后所有的依赖都将打包进入 image 文件
RUN apt-get update && apt-get upgrade	# optional

# 将我这个 image 做成一个 app 可执行程序，容器启动后自动执行下面指令
ENTRYPOINT ["bash", "setup.sh"]

```
### 提醒
可以在项目的根目录下创建一个 .dockerignore 文件夹，表示可排除的文件，类似 .gitignore。

也可将 ENTRYPOINT 换做 CMD ，都是容器启动后自动执行指令，简单区别就是 ENTRYPOINT 可以在本地启动容器时加额外的shell参数。
另外，一个 Dockerfile 可以包含多个RUN命令，但是只能有一个CMD 或者 ENTRYPOINT 命令。

```shell
CMD bash setup.sh

```

其中用ENTRYPOINT 来构造镜像，但是启动它的容器时候想要用带有参数（如 <command> <arg1> <arg2>）的指令，那怎么办呢？

参考此博文 https://blog.csdn.net/chenxing109/article/details/85319489 就可以知道，可以利用 --entrypoint 参数来覆盖并运行容器：

```shell
docker run --entrypoint <command> <image>:<tag> <arg1> <arg2>
```

另外，DEBIAN_FRONTEND 这个环境变量很有用，可以告诉操作系统应该从哪里获得用户输入。常用的设置是 noninteractive，这样你就可以直接运行命令，
而无需向用户请求输入（即所有操作都是非交互式的）。这在运行 apt-get 命令的时候格外有用，因为它会不停的提示用户进行到了哪步并且需要不断确认。
非交互模式会选择默认的选项并以最快的速度完成构建。请确保只在 Dockerfile 中调用的 RUN 命令中设置 了该选项，而不是使用 ENV 命令进行全局的设置。
因为 ENV 命令在整个容器运行过程中都会生效, 所 以当你通过 BASH 和容器进行交互时，如果进行了全局设置那就会出问题。


要注意的是，正确的做法：只为这个命令设置 ENV 变量：

```shell
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y python3
```
再比如：

```shell
FROM ubuntu:trusty
RUN \
DEBIAN_FRONTEND=noninteractive apt update && \
DEBIAN_FRONTEND=noninteractive apt clean
```

## 8. 创建image

