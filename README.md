# docker学习


Docker学习笔记


一、初识Docker
Docker是一款容器，通过Docker能够快速搭建好开发和运行环境。 

二、容器

容器的概念：
容器就是打包了应用和服务的环境。每个容器都由一组特定的应用和必要的依赖库组成。

2.1、 容器的管理操作

容器的常见命令包括：查看、创建、启动、终止和删除。

1 创建容器
  创建容器有两种命令：docker create和docker run，区别在于：docker create创建的容器处于停止状态，docker run不仅创建了容器，还启动了容器。
  创建容器后，Docker会立刻返回容器的ID，ID是可以唯一标识一个容器，每个容器的ID都是独一无二的。
  1 命令 docker ps：查看正在运行的容器
  docker ps -a：查看所有容器，包括未启动的容器。
  参数-n=x：表示列出最后创建的x个容器 docker ps 参数含义 CONTAINER ID：唯一标识容器的ID IMAGE：创建容器时使用的镜像 COMMAND：容器最后运行的命  令 CREATE：创建容器的时间 STATUS：容器的状态，Up 2 minutes标识运行2分钟，未运行的状态是Exited(0)表示已退出 PORTS：对外开放的端口 NAMES：容器 名
  
2 运行容器
  可以使用docker run让容器进行运行状态，等同于 docker create + docker start，docker run可以创建两种类型的容器：
    * 1 交互型容器：运行在前台，通常会指定有交互的控制台，用于输入输出，终端关闭即停止。
    * 2 后台型容器：运行在后台，与终端无关，只有调用docker stop或docerk kill命令容器才会停止，创建后台型容器需要加参数-d启动容器 
    -i：表示打开容器的标准输入
    -t：表示建立一个命令行终端
    --name：表示为容器指定一个名字
    ubuntu：表示使用哪个镜像去创建容器，这里使用ubuntu，ubuntu是一个基础镜像，还可以使用的有 fedora、debian、centos等作为创建自己镜像的基础
    /bin/bash：表示要在容器里面执行/bin/bash
    --restart=always/on-failure:x，重启，always表示总是重启，on-failure表示返回值是非0才重启，x是可选参数，表示尝试的重启次数。

3 流程
运行docker run命令后，Docker在本地搜索我们制定的ubuntu镜像，没有找到就会到公有仓库Docker Hub中搜索，找到之后就下载到本地，然后Docker使用这个镜像创建一个新的容器并将其启动，然后为容器分配文件系统和网络，之后执行程序，知道容器停止。

4 终止容器
交互型容器可以在shell中使用exit/ctrl+d来停止，交互型和后台型都可以使用docker stop name/ID来停止，但容器内程序可以捕获该信号并自行处理比如忽略。如果要强行停止容器，可以使用docker kill name/ID

5 删除容器
当容器停止后，容器并没有消失，只是进入了停止状态，必要的话还可以重新运行。如果不再需要这个容器，可以使用docker rm name命令删除它。注意：不能删除运行中的容器！除非使用-f强制删除，一次性删除所有容器的命令：docker rm `docker ps -a -q`，-a列出所有容器，-q只列出ID，相当于将所有ID传给rm进行删除！

2.2、容器内信息获取和命令执行

上一节容器的管理操作，属于外部操作，容器的内部操作包括：获取容器的内部信息以及在容器内部运行命令

1 依附容器
由于docker start启动的交互型容器并没有具体终端可以依附，而交互型容器本身是可以接收用户交互的，这是就需要attach命令来讲终端依附到容器上。注意：后台型容器无法依附终端，因为它本身不接受用户交互输入！ attach

2 查看容器日志

  1 创建一个后台型容器，不断输出自然数：
  docker run -d --name deamon_logs ubuntu /bin/bash -c 'for((i=0;1;i++));do echo $i;sleep 1;done;'
  2 使用命令docker logs -f deamon_logs查看日志，参数：
  -f表示实时查看
  --tail==x表示精确控制logs输出的日志最后的行数，这里表示输出日志最后x行

3 查看容器进程
使用docker top查看容器中正在运行的进程：
top

4 查看容器信息

1 docker inspect查看容器的配置信息，包含容器名、环境变量、运行命令、主机配置、网络配置和数据卷配置等
使用-f或--format格式化标识，可以查看指定部分的信息：
inspect

5 容器内执行命令
docker exec命令在容器中运行新的任务，可以创建两种任务：后台型和交互型任务。

1 创建后台型任务：
docker exec -d deamon_logs touch /etc/new_config_file：表示在容器deamon_logs中新建一个后台型任务，任务为在deamon_logs容器中创建一个new_config_file文件，-d表示创建后台型任务！
2 创建交互型任务：
docker exec -t -i deamon_logs /bin/bash：这个比较常用！通过-i、-t(和创建交互型容器一样)创建一个交互终端，并捕捉进程的标准输入和输出。

2.3、容器的导入和导出

Docker可以将容器导出到本地文件系统中，也可以将导出的容器重新导入到Docker运行环境中。Docker的导入导出分别由import和export命令完成。
export命令：
把容器的文件系统以tar包的格式导出到标准输出，这样可以将tar包分享给别人，起到了docker hub的作用！ export import命令：
：把my_container.tar包中的内容导入为一个镜像！ export 也可以使用url来导入网上的容器：
docker import url res:tag，res和tag分别是生成的镜像和标记。
想要使用镜像构建容器可以使用命令：docker run -i -t --name=xxx res:tag/ID yyy创建交互型容器或者docker run -d --name=xxx res:tag/ID创建后台型容器。

三、镜像

镜像的概念：
镜像是容器的运行基础，容器时镜像运行后的形态。

3.1、镜像的概念

镜像是一个包含程序运行必要依赖环境和代码的只读文件，它采用分层的文件系统，将每一次改变以读写层的形式增加到原来的只读文件上。

1 镜像与容器
镜像是容器运行的基石，如果把容器理解为一套程序运行时的虚拟环境，那么镜像就是用来构建这个环境的模板。通过同一个镜像，可以构建出很多相互独立但运行环境一样的容器。
不同镜像也许有着不同的服务目标，比如ubuntu镜像用来构建一个精简的Ubuntu操作系统容器环境，wordpress镜像用来构建博客程序容器环境。 image 镜像的本质是磁盘上一系列文件的集合。创建新的镜像就是对已有镜像文件进行增、删、改操作。镜像之间并不是孤立的，而是存在单向的文件依赖关系。正以为Docker的这种文件层叠共享机制，才造就镜像占用磁盘空间小、扩展容易、传播灵活等有点。 image
3.2、本地镜像的管理

本地镜像的基础管理：查看、下载和删除。

1 查看
通过docker images命令，可以列出本机上的所有镜像： image REPOSITORY：仓库名称。仓库一般用来存放同一类型的镜像，其名称由它的创建者指定。没有指定则为。仓库的名称有下面几种形式：
      > [namespace\ubuntu]：由命名空间和实际的仓库名称组成，中间通过\隔开。Docker Hub上的账号名就是你的命名空间。
      > [ubuntu]：只有仓库名。属于顶级命名空间。该空间只用于官方的镜像，由Docker官方进行管理。用户可以在本地创建镜像命名，但无法分发到Docker Hub进行共享。
      > [d1.dockerpool.com:5000\ubuntu:14.04]：指定URL路径的方式。如果该镜像不是放置在Docker Hub上，而是防止在自己搭建的Hub或第三方Hub上，则使用这种方式命名。
TAG：用于区分同一仓库中的不同镜像。默认为latest。
IMAGE ID：全网表示一个镜像的ID。该字段只展示前面一部分，已足以在本机唯一表示一个镜像了！
CREATE：镜像的创建时间。
VIRTUAL SIZE：镜像所占用的虚拟大小，包含了所有共享文件大小。
使用images命令只会列出镜像的基本信息，可以使用inspect命令得到镜像更详细的信息。

2 下载
使用docker run命令运行一个镜像时，Docker首先会在本机寻找该镜像。如果本机不存在，会继续去Docker Hub上搜索符合条件的镜像并将其下载下来运行。使用命令docker search imageName可以搜索出符合要求的镜像： search NAME：镜像的名称。
DESCRIPTION：镜像的简要描述。
START：用户对镜像的评分。
OFFICIAL：是否为官方镜像。
AUTOMATED：是否使用了自动构建。
为了在使用run命令时不等待过久，可以使用docker pull imageName命令先把镜像从Docker Hub中拉去到本地。等到运行时，可以节约时间。
删除镜像：docker rmi ID/name：删除指定的镜像。

3.3、创建本地镜像

除了导入tar，创建镜像的方法还有以下两种：

1 使用commit命令创建本地镜像
使用镜像创建并运行一个容器，实际上是在父镜像基础上创建一个可读写的文件层级。我们再容器里所做的修改都发生在这个层级上面。下面演示在ubuntu镜像上创建和运行一个容器，并在该容器上安装SQLite3以及在根目录下创建一个名为hellodocker的文件，并写入test docker commit：
create commit 使用-m参数描述此次创建image的信息，--author参数指定作者信息，tomotozh和sqlite3分别是仓库名称和镜像名称，v1是TAG名。
使用刚刚创建的镜像来构建一个容器并运行，以检视所做的修改：
test 成功！ 这种方式相当于在父镜像基础上进行文件层级操作，操作之后生成的新的环境再创建成镜像！即创建之前已经是成形的！这种方式只能构建一次，再次构建需要进行再次操作才能构建镜像！

2 使用Dockerfile创建镜像
更加推荐使用Dockerfile来构建镜像。将需要对镜像进的操作全部写到一个文件中，然后使用docker builde命令从这个文件中创建镜像。这种方式相当于把对父容器的操作写入到一个文件中，然后从文件中创建镜像。这种方式可以使镜像的创建变得透明盒杜丽华，并可以重复创建，因为是针对文件，只需要对文件再一次build即可！
docker build -t name .：-t指明了镜像名称，.表示从当前目录下寻找Dockerfile用于构建image！Dockerfile的编写请查看官网！
每执行一步，Docker会从前一个临时镜像创建出容器，然后在容器中执行当前步骤的操作，接着提交这个容器为一个新的临时镜像，供下一条Dockerfile指令使用，同时将前一个临时镜像删除。可以通过设置docker build命令参数-rm=false，避免临时缓存被删除！参数--no-cache=true禁用Docker构建器的缓存机制，。

3.4 Docker Hub

1 Docker Hub简介
Docker Hub提供不限数目的公开镜像托管服务，但仅提供一个私有镜像托管服务。额外的需要付费！

2 镜像的分发
步骤：

1 docker login，首先需要登录
2 docker push imageName，上传镜像到Docker Hub。

四、数据卷及容器连接

应用在容器汇总运行，总会用到或者产生一些数据。

容器网络基础：容器通过对外暴露端口向外提供服务；
数据卷的概念和使用：通过数据卷来存储和共享数据；
容器连接：通过互联让一个容器安全地使用另一个容器已有的服务。

4.1、容器网络基础

容器是寄宿在宿主主机上的，必须能够通过外部网络访问到容器，容器才能提供服务。当Docker启动时，它会在宿主主机上创建一个名为docker0的虚拟网络接口。通过ifconfig命令，可以看到本机的网络接口的情况：

1 暴露网络端口
当在Docker中运行网络应用时，需要在外部访问Docker中运行的应用，需要通过-P或-p参数来指定端口映射。通过端口映射来实现端口暴露是容器对外提供服务的基础方法。
-P 参数：在启动时使用-P 参数，Docker会在宿主主机上随机为应用分配一个49000~49900内未被使用的端口，并将其映射到容器开发的网络端口。
-P 接下来就可以使用localhost：32768进行访问容器内应用了！
-p 参数：可以指定宿主主机上的端口映射到容器内指定的开放端口。有如下三种格式：
ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort
  1 ip:hostPort:containerPort：指定主机的指定端口-->容器内指定端口
  2 ip::containerPort：宿主主机的随机端口-->容器内指定端口
  3 hostPorrt:containerPort：宿主主机的指定端口-->容器内指定端口 -p
2 查看网络配置
命令：docker inspect --format '{{.NetworkSettings}}' ID，即使用docker inspect查看容器信息！
4.2、数据卷 volume

Docker镜像被存储在一系列的只读层，当开启一个容器，Docker读取只读镜像并添加一个读写层在顶部。如果允许的容器修改了现有的文件，该文件被拷贝出底层的只读层到最顶层的读写层！只读层和读写层的组合被Docker称为Union File System联合文件系统
存在的问题：
* 宿主机不方便对容器内文件进行访问 * 多个容器之间的数据不能共享 * 当删除容器时，容器产生的数据将丢失
这时，通过数据卷，容器产生的数据存放到数据卷中，当容器删除时，数据卷中的数据不受影响！宿主机和其他容器也可以访问数据卷！这样达到了文件共享和防止数据丢失！
数据卷是容器中特定的文件或文件夹，这个目录能够独立于联合文件系统的形式存在于宿主主机中。

数据卷是一个可供一个或多个容器使用的数据目录，使用它可以达到如下目的：
* 绕过“拷贝写”系统，以达到本地磁盘I/O性能
* 绕过“拷贝写”系统，有些文件不需要在docker commit的时候打包进镜像
* 在多个容器之间共享目录
* 在宿主和容器之间共享目录
* 在宿主和容器之间共享单个文件

1 创建数据卷
两种方式创建数据卷：

  1 在Dockerfile中，使用VOLUME指令，如：VOLUME /var/lib/postgresql

  2 在命令行中使用docker run'时，使用-v参数来创建数据卷并挂在到容器中：docker run -d -v /webapp training/webapp python app.py，表示创建     了/webapp数据卷并挂在到容器中。
  -v 到这里，只是声明了数据卷！
2 挂载主机目录作为数据卷
除了上述的仅仅声明一个数据卷外，还可以指定宿主主机上的某个目录作为数据卷：
docker run -d -p 5000:5000 -vpwd:/dirname:ro training/webapp python app.py：pwd表示绝对路径，ro表示只读，不加默认为rw可读可写。 -v . 注意，挂载的目录必须为绝对路径！且Dockerfile不支持挂载本地目录到数据卷，主要是因为不同操作系统目录格式不同，为了保证Dockerfile的移植性！

3 挂载主机文件作为数据卷
除了可以将主机目录挂载为数据卷外，还可以将单个主机文件挂载为容器的数据卷。：tomoto$ docker run -i -t --name=ubuntu_v -v ~/docker/test.txt:/test.txt ubuntu:latest /bin/bash：表示将当前文件test.txt挂在到容器内的test.txt。 -v

4 数据卷容器
数据卷容器是指一个专门用于挂载数据卷的容器，以供其他容器引用和使用。主要用在多个容器需要从移除获得数据时。需要将数据容器命名，然后其他容器通过--volumes -from来引用它的数据卷。

1 首先，建立一个数据卷容器，名为dbdata，并为该容器新建数据卷/dbdata：
docker run -d -v /dbdata --name dbdata training/postgres，
2 然后创建一个容器db3，引用dbdata的数据卷：
-v 数据卷容器还可以级联引用，比如db1引用dbdata，db2引用db1，和dbdata一样，db1和db2他们都是共用一个数据卷！
删除容器时删除数据卷的方法：
docker rm -v ID：使用-v参数
docker run -rm ...：启动容器的时候使用--rm参数，停止容器时就会自动删除容器和数据卷！
5 数据的备份和恢复

1 备份
利用数据卷容器，可以备份一个数据卷容器的数据。做法就是：新建一个容器A，引用数据卷容器B，并创建一个数据卷映射，将本机目录d映射到容器中的目录f，再把数据卷容器引用的目录打包到f中，这样本机目录d就拥有了目录f中的数据，也就是数据卷容器中的数据了！这样就讲数据卷容器中的数据进行了本地备份！
2 恢复数据
恢复好备份类似！新建容器dbdata，再新建容器B引用dbdata，并关联本地目录，将本地目录中的备份数据放到容器B中的dbdata，这样数据卷容器dbdata就有了备份数据，这样就达到了备份的目的！
4.3、容器连接

容器连接是容器对外提供服务的一种方法。容器连接诶包含源容器和目标容器。
源容器是提供服务的一方，对外提供指定服务；目标容器连接到源容器之后，就可以使用其所提供的服务了。
容器连接依赖于容器名，所以当需要使用容器连接时，首先需要命名容器，然后使用link参数进行连接。

1 容器命名
--name，使用run命令启动容器时，通过--name参数指定名字。
2 容器连接
连接的格式为--link name:alias，其中name是源容器的名称，alias是这个连接的别名。比如A容器需要连接B容器，就可以使用docker run --link B:alias A进行连接。
-link -link 此时，webapp容器成功与dbdata容器建立连接。通过这种方式，dbdata容器为web容器提供了服务，但并没有像-P或者-p参数那样，让容器对外暴露端口，这使得源容器dbdata更安全！

web容器如何使用dbdata容器的服务呢？
Docker给目标容器提供了两种方式来暴露连接提供的服务：

环境变量
/etc/hosts文件
  1， 环境变量
  当两个容器通过连接互联会后，Docker会在目标容器中设置相关的环境变量，以便在目标容器中使用源容器提供的服务。连接环境变量的命名格式为alias_NAME，其中alias为--link参数中的别名。一般情况下，可以使用env命令来查看一个容器的环境变量：
-env 
  2， /etc/hosts文件
  查看目标容器的/etc/hosts配置文件，具体操作如下：
-env 可以看到，容器连接webdb对应的地址为172.17.0.2，该地址实为dbdata容器的地址！容器对webdb连接的操作将会映射到该地址上！！！

3 代理连接
上面说的容器连接都是在一个宿主主机上的连接。对于跨主机的容器连接，可以利用ambassador模式实现跨主机连接，这种模式的连接称为代理连接！
通过代理连接，可以解耦两个原本直接相连的容器的耦合性。

原始模式： 
  1，不能跨主机连接；
  2，耦合性太高，当需要连接到新的redis-server时，redis-client必须重启才行！
ambassador代理模式： 
  客户机上的client容器连接到本机的ambassador1代理容器，ambassador1代理容器通过网络连接到服务器主机上的ambassador2代理容器，ambassador2代理容器连接到server容器，最终实现client使用server的服务。client根本不需要关心连接到的是哪一个server！
  
步骤：
  1，在服务器主机上启动一个redis-server服务的容器： 
  2，在服务器主机上建立一个代理容器ambassador2，将它连接到redis-server： 
  3，客户机上建立一个ambassador1代理容器，将它连接到服务器主机的代理容器ambassador2： 
  4，客户机上只需连接到本机的ambassador1dialing容器即可：


五、创建SSH服务镜像

之前的命令比如attach、exec等在容器内部管理容器的命令，无法解决远程管理容器的需求。需要使用SSH服务连接到远端系统进行管理系统。Docker很多镜像都没有安装SSH服务，我们需要自己为其安装SSH服务！

5.1、基于commit命令的方式

Docker的commit命令提供了将用户修改过的容器提交成为新镜像的功能。
