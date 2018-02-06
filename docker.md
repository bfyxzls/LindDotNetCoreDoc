1. Docker是什么？
Docker是一个软件，它提供了容器服务
除了Docker外，还有CoreOS团队推出的rkt容器软件，也提供容器服务

2. 容器服务是什么？

 
首先，不论什么容器软件都是必须安装在操作系统之上的，linux或windows
上图这个大鲸鱼就是操作系统

容器软件引擎(例如Docker)通过调用linux操作系统内核的LXC,Cgroup,Namespace等技术，实现了将单个操作系统资源有效划分到孤立的组中，这些孤立的组叫做容器(container)
上图大鲸鱼上面装的集装箱就是容器


3.容器相比传统的虚拟机有什么区别
传统虚拟机是虚拟出了整个操作系统，而虚拟出的操作系统除了里面运行的程序占用内存外，操作系统本身还占用内存；相对来说容器技术以共享Kernel的方式实现，几乎没有性能损耗
容器的资源隔离效果要比虚拟机差，因为容器是共享内核的，虚拟机是虚拟化出了一个完整的内核
虚拟机很重，因为需要安装完整的操作系统；相对来说容器就很轻，一个容器的镜像只有几十兆甚至几兆


4. 容器有什么好处
可以快速部署，因为容器轻
可以保证一致性的应用程序交付，因为容器轻，所以能把程序运行环境(jvm,nginx等)和代码一起打包到容器内，直接交付测试生产环境

5. 使用容器需要注意什么?主要是不违反容器有什么好处
容器本身应该是轻量级的,不应该往容器里安装大量的软件,大的镜像难以快速分发；基于容器应该是轻的
容器本身应该是可随时立即摧毁重建的,是临时的(基础Dockerfile或swarm来实现)；基于容器要保证一致性的应用程序交付
容器本身应该是无状态的(可以理解为某个容器挂掉或重建了,不应该影响到其它的容器)；基于容器要保证一致性的应用程序交付
不要将数据存储在容器中,但是程序代码应该放置到容器中,作为最终交付的版本；基于容器要保证一致性的应用程序交付
不要从正在运行的容器中commit镜像,因为这是不可重复的，始终使用Dockerfile；基于容器要保证一致性的应用程序交付
不要只使用"latest"标签,从创建缓存的检索中检索到了一个错误的“latest”版本,使用标签可以方便跟踪和回滚版本问题；基于容器要保证一致性的应用程序交付
不要在一个容器中运行多个进程,Docker的机制如Healthy等只支持单一主进程;基于容器本身的稳定性
容器不要依赖IP地址,例如docker swarm使用services name进行通讯；基于容器本身的稳定性

PS:理论上来说，虚拟机也能实现快速部署和一致性的应用程序交付，就是把程序运行环境(jvm,nginx等)，代码等一起打包到虚拟机内,但是虚拟机相对容器来说，太重了

6.Docker容器的生态环境
单机版的Docker用途有限，无法承载更大的负载，因此需要支持分布式集群解决方案
目前主流的Docker集群方案有：k8s,swarm,mesos
咱们公司用的swarm,这是Docker官方的集群解决方案
k8s,全称Kubernetes，社区解决方案，背后支持者是google
mesos,社区解决方案，背后支持者是 apache 基金会

咱们公司选择的是Docker+swarm的解决方案，原因是swarm入手相对简单，并且目前能满足咱们的使用需求

PS:如上三种方案是为了解决Docker的分布式部署问题；最后逐渐发展了各自的生态圈，包含部署，高可用，迁移，日志搜集，监控等


7.无论使用哪种集群方案，都必须首先安装Docker软件
   这里示例是基于centos7,基于阿里云，安装Docker软件，别的地方安装注意修改下面红色字体
name=A01
hostnamectl set-hostname $name

yum remove -y docker docker-common docker-selinux docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum list docker-ce.x86_64  --showduplicates | sort -r
#列出可以安装的docker版本
yum -y install docker-ce-17.06.1.ce
#安装指定版本的docker

mkdir -p /lib/systemd/system/docker.service.d
cat > /lib/systemd/system/docker.service.d/docker.conf << 'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --default-ulimit nofile=65535
EOF

mkdir -p /etc/docker/
mkdir -p  /srv/docker/
cat > /etc/docker/daemon.json << 'EOF'
{
    "dns": [
        "100.100.2.136",
        "100.100.2.138"
    ],
    "data-root": "/srv/docker/",
    "hosts": [
        "unix:///var/run/docker.sock"
    ],
    "registry-mirrors": [
        "https://0sr73mco.mirror.aliyuncs.com"
    ]
}
EOF
#"registry-mirrors": ["https://registry.docker-cn.com"]，中国docker hub专用地址，docker官方提供

systemctl daemon-reload
systemctl start docker
systemctl enable docker
docker info

7. Docker的版本

2016 年 7 月 29 日  Docker 1.12发布
2017 年 1 月 18 日  Docker 1.13发布

2017 年 3 月 2 日，Docker 官方发布了一篇 blog ，宣布企业版到来。版本也从1.13.x一跃到17.03。
之后，Docker 会每月发布一个 edge 版本(17.03, 17.04, 17.05...)，每三个月发布一个 stable 版本(17.03, 17.06, 17.09...)，
企业版(EE) 和 stable 版本号保持一致，但每个版本提供一年维护。


Docker 的 Linux 发行版的软件仓库也从以前的https://apt.dockerproject.org / https://yum.dockerproject.org 
变更为目前的 https://download.docker.com/。软件包名变更为 docker-ce(社区版) 和 docker-ee(企业版)。
旧的仓库和包名(docker-engine)依旧可以使用，但不确定什么时候会被废弃，docker-engine 的版本号也变成了17.03.0~ce-0这种的版本号。

Docker v17.03.0-ce 版本更新内容和下载地址请查看发行日志。
来源： http://www.oschina.net/news/82496/docker-1-17-03
信息来源：http://www.oschina.net/news/82496/docker-1-17-03


几个概念
docker engine   docker软件进程
docker machine  docker官方部署工具，针对cloud环境，例如阿里，aws等，可以远程部署docker,swarm等，集成了cloud的接口
docker compose 官方编排工具，可以实现同时启动几个具有依赖关系的容器，以写Dockerfile的方式把容器依赖关系写到yaml文件里
docker swarm 官方集群管理工具，接口和docker单机基本一致
docker registry 官方的镜像仓库，可以配合Harbor/Portus使用，具有web管理界面
images 镜像，一般由各个官方网站提供的，例如alpine镜像，centos镜像，nginx镜像，可以理解为咱们平时装系统时的系统盘ISO文件
container 容器，运行一个容器的目的是提供服务，所以容器里面需要安装一些内容，这些内容就是由images提供的

PS： images格式：nginx:1.13,nginx:latest 前面是image名称,后面是tag,tag可以自定义,但一般都是跟版本号;
                                如果没有指定tag,则默认是latest,如nginx,和nginx:latest一个意思


8.单机版Docker的常用命令

ps -ef | grep dockerd
#docker服务的进程，dockerd是Docker软件的server端进程

docker info
#docker命令是Docker软件的client端进程,通过命令行管理dockerd server端的

docker pull alpine
#下载最新的alpine image

docker run -i -t alpine echo "hello world"
docker run -i -t alpine /bin/sh
#docker run创建并运行一个容器
#进入alpine的shell后，测试运行cat /etc/os-release查看alpine版本号
#-i许允我们对容器内的STDIN进行交互,-t在容器内指定一个伪终端或终端

docker images
docker images | grep alpine
docker images --no-trunc
docker images -q
docker images --help
#查看images信息

docker run -d alpine /bin/sh -c "while true; do echo hello world; sleep 1; done"
#docker run创建并运行一个容器，-d在后台运行

docker ps
#查看正在运行的容器
docker ps -a
#查看所有的容器，包含正在运行的，已经退出的，暂停的等等

docker ps -q
#只显示正在运行的容器的短ID,前12位

docker ps --no-trunc
#不截断显示所有的信息

docker run -d  --name testnginx -m 1g -p 8080:80 -v /volume/nginx.conf:/etc/nginx/nginx.conf nginx
docker images | grep nginx
#直接运行一个nginx容器，会自动下载nginx镜像
#nginx 镜像提供nginx服务，默认端口是80端口，但是这个80端口只是在这个运行的容器内部用的
#虽然这个nginx容器运行在一个操作系统上(这里是centos7)，但是这个操作系统也无法直接访问这个nginx容器的80端口
#-p 8080:80是把nginx容器的80端口映射到操作系统的8080端口上
#因此可以直接访问操作系统的8080端口来访问nginx,curl 127.0.0.1:8080
#-m 1g代表这个nginx容器最多可以使用宿主机操作系统1G的物理内存
#--name testnginx 给这个启动的nginx容器起一个名字
#-v /volume/nginx.conf:/etc/nginx/nginx.conf将宿主机操作系统的某个文件挂载到容器内部的某个目录，当然也可以挂载目录

docker logs e23dd06d1c50
docker logs testnginx
#打印出一个容器的标准输出日志,后面跟容器名称或id

docker logs -f testnginx
#从头开始，持续输出日志，类似shell的tail -f 1.txt命令

docker logs --tail 100 -f testnginx
#只显示最近的100条日志，并持续输出新生成的日志

docker logs --tail 100 -f -t testnginx
#只显示最近的100条日志，并持续输出新生成的日志,-t给每条日志打上一个时间戳

docker exec -it testnginx /bin/sh
#进入一个正在运行的容器的shell

docker diff  testnginx
#可以查看一个容器的内部文件变化情况，非常有用
#A-added,D-deleted,c-changed


docker rm affectionate_kepler
docker rm -f testnginx
#-f 强制删除一个正在运行的容器

docker system df
#查看docker中各项占用多少空间，

docker system df -v
#可以查看更加详细的信息

docker system events
docker system events --since 2016
#查看docker系统日志，包含创建销毁容器的信息

docker system prune -af
#清理docker无用的镜像、容器、卷、网络

docker system prune -af --volumes
#17.06.1默认不会删除volumes，需要额外定义

docker image prune
#删除无用的镜像
docker container prune
#删除无用的容器
docker volume prune
#删除无用的卷
docker network prune
#删除无用的网络


docker stats $(docker ps --format={{.Names}})
docker stats --format 'table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}\t{{.PIDs}}'
docker stats --all --format "table {{.ID}}\t{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker stats --all --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
#查看docker stats状态
#--format显示那些列
#-all显示所有容器信息，Running and Stopped

docker stats $(docker ps --format={{.Names}}) --no-stream  
#查看docker stats状态，不刷屏

docker stats testnginx
#显示指定的容器信息



9. docker swarm集群

swarm是什么？
swarm集群就是一个集群软件名称叫swarm，安装好之后再把那些单机版本的dockerd加入到集群中来，由swarm进行统一管理
由于dockerd本身集成了swarm，在安装docker软件本身时就已经安装了swarm，谁让swarm是docker自家推出的集群呢

1. 创建swarm集群

docker swarm init --advertise-addr 192.168.2.212
#初始化docker swarm集群，首先创建第一个节点，ip地址是节点IP，第一个节点是swarm的manager节点，保存swarm集群信息

docker swarm join-token manager
#生成加入swarm manager节点命令，在需要加入swarm的dockerd节点上运行

docker swarm join-token worker
#生成加入swarm work节点命令,work节点不保存集群信息，在需要加入swarm的dockerd节点上运行

PS:
既然由swarm进行统一管理了,管理那么多个dockerd,就必须保证swarm的高可用性,所以swarm的集群信息至少要保存在三个节点上来保证高可用
由于swarm的集群信息只保存在manager节点上，因此至少创建三个manager节点;
manager节点的状态有Leader和Reachable两个状态，状态是Leader的节点是主节点，只有一个，其余的manager节点都是Reachable状态
work节点不保存集群信息，只会接受并运行manager节点发送来的任务，例如创建容器，销毁容器
manager节点和work节点默认都可以创建容器，销毁容器

docker swarm leave
#退出swarm集群，在需要退出swarm的dockerd节点上运行

下面的命令只能在manager节点上执行
-----------------------------------------------------------------------------------------------------------
docker node ls
#列出所有swarm的节点，manager，work节点

docker node inspect z48ts3fx22c8snzazjac99lnh --pretty
#查看某个节点的信息

docker node update --availability drain z48ts3fx22c8snzazjac99lnh
docker node update --availability active z48ts3fx22c8snzazjac99lnh
docker node update --availability pause z48ts3fx22c8snzazjac99lnh
#更新某个节点的可用性，drain表示这个节点不再接受来自manager节点的任务，并且将当前节点上运行的容器全部销毁，节点维护时使用
#更新某个节点的可用性，pause表示这个节点不再接受来自manager节点的任务,但已有的容器不会被销毁
#更新某个节点的可用性，active表示这个节点接受来自manager节点的任务,可以正常创建容器，销毁容器

docker node promote z48ts3fx22c8snzazjac99lnh
#将一个节点升级成manager节点
docker node demote z48ts3fx22c8snzazjac99lnh
#将一个节点降级成work节点
docker node rm z48ts3fx22c8snzazjac99lnh
#从集群中主动删除一个节点，最好同时在要退出swarm的dockerd节点上运行docker swarm leave

2. 创建swarm网络

docker network ls
docker network create --driver overlay my-network
docker network ls
docker network inspect my-network
#注意看，此时会有两个Driver是overlay的网络
#overlay网络可以让swarm集群中位于不同宿主机上的容器直接互相通讯
#只有处于相同的overlay网络的容器才能互相通讯，不同的overlay网络的容器不能互相通讯


3. 创建一个swarm service

单机版dockerd创建容器的方法是docker run直接创建一个容器运行
swarm创建容器的方法是先创建一个service,然后指定这个服务里运行的容器的副本的个数，它的本质也是在在dockerd节点上创建容器，不过是有swarm集中管理的

docker service create --replicas 3 --name helloworld nginx
docker service ls
docker service ps helloworld
docker service logs -f 5bwrqvtr8fty
docker ps
docker service inspect helloworld --pretty
docker network inspect ingress
#创建service时，没有指定--network networkname时，默认使用ingress的默认overlay网络

docker service scale helloworld=5
#动态调整一个service的replicas副本容器数量

docker service ls
docker service ps helloworld
docker service update --image nginx:1.12 helloworld
#更新一个service，这里是更换helloworld这个service的image镜像
docker service ls
docker service ps helloworld

export SERVICE_NAME='yougroup-test-nginx'
#export IMAGE_NAME='registry.yourhost.cn/yougroup/test/nginx'
export IMAGE_NAME="nginx:1.13"

docker service create \
    --name "$SERVICE_NAME" \
    --hostname "{{.Node.ID}}-{{.Service.Name}}" \
    --network my-network \
    --publish "mode=host,published=80,target=80" \
    --publish "mode=host,published=443,target=443" \
    --label com.example.foo="bar" \
    --container-label aliyun.logs."$SERVICE_NAME"=/var/log/nginx/access2.log \
    --container-label aliyun.logs."$SERVICE_NAME".format=json \
    --container-label aliyun.logs."$SERVICE_NAME".tags="host="$(hostname)"" \
    --env MYVAR=foo \
    --replicas=2 \
    --update-delay 5s \
    --rollback-delay 1s \
    --update-parallelism 1 \
    --rollback-parallelism 3 \
    --update-failure-action=rollback \
"$IMAGE_NAME"

docker service inspect --format='{{json .Spec.Mode.Replicated.Replicas}}' yougroup-test-nginx
#json格式
docker service ps yougroup-test-nginx --no-trunc 
#显示ps的完整信息

docker service update --image nginx:1.12 yougroup-test-nginx
#将yougroup-test-nginx的image修改成nginx:1.12；更新代码版本时也是直接更新打包好的新的image

docker service update --rollback yougroup-test-nginx
#--rollback，手动回滚，会把服务回滚到最近的docker service update之前的配置，这里会把更新后的image nginx:1.12回滚成nginx:1.13
#经实验证明，即使更新前后使用的image名称tag一致，依然可以回滚到之前的配置，包括更新前的service使用的image,但依然建议每次build时使用不同的tag


10. Dockerfile书写规范
什么是Dockerfile？
Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像，它们简化了从头到尾的流程并极大的简化了部署工作
我们可以在Dockerfile里定义一些我们自己的内容,例如环境变量,配置文件,代码等等最终打包成一个新的image来满足我们的使用
我们书写Dockerfile的目的就是把它打包成image来供我们使用

Dockerfile从FROM命令开始，紧接着跟随者各种方法，命令和参数。其产出为一个新的可以用于创建容器的镜像。


FROM nginx:1.12.1-alpine
LABEL "maintainer"="14339989@qq.com" \
           "aliyun.logs.nginx=/var/log/nginx/access2.log"
#LABEL是通过docker inspect containerid来查看的
EXPOSE 80 443
ENV myName="John Doe" \
        myDog=Rex \
        TZ=Asia/Shanghai \
        myCat=fluffy
#通过image创建的容器，可以看到ENV的环境变量；环境变量也可以通过docker service --env来创建
WORKDIR /tmp/    
#ADD,COPY,CMD,ENTRYPOINT,RUN命令都会在这个目录下执行
COPY check.sh /scripts/
COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d/ /etc/nginx/conf.d/

ENV TZ=Asia/Shanghai
RUN echo "http://mirrors.aliyun.com/alpine/v3.5/main" >/etc/apk/repositories \
    && echo "http://mirrors.aliyun.com/alpine/v3.5/community" >>/etc/apk/repositories \
    && apk add --update tzdata \
    && apk add curl \
    && rm -rf /var/cache/apk/* \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && rm -rf /etc/nginx/conf.d/default.conf

#ADD,COPY都不可以拷贝Dockerfile所在目录外的文件；需要的话可以通过其他方法先拷贝到Dockerfile所在目录
#如果目标位置不以"/"结尾，则目标则会被认为成一个常规文件，不会当作目录处理
#如果目标位置不存在，则会自动创建
#建议只用COPY

HEALTHCHECK --interval=5s --timeout=3s CMD curl -f http://localhost/ || exit 1
#间隔5s检测一次，每次检测3s内没有正常返回则判断检测失败，默认检测重试3次后，判断容器为unhealthy状态,然后会销毁当前容器并重建新的容器，如此往复


HEALTHCHECK --start-period=30s --interval=5s --timeout=3s --retries=16 CMD /bin/sh /scripts/check.sh $URL
#--start-period=30s检测周期30s后开始，间隔5s检测一次，每次检测3s内没有正常返回则判断检测失败，默认检测重试3次后，判断容器为unhealthy状态
#$URL这个环境变量可以在Dockerfile里直接定义，也可以在创建service时定义

: '
cat /scripts/check.sh
#!/bin/sh -xe
URL=$1
if [ "$URL" == "" ];then
    exit 1
fi

Output=`curl -s $URL | jq -r '.status'`
if [ "$Output" == "UP" ];then
    exit 0
else
    exit 1
fi
'


docker build --pull -t mynginx ./
#./目录里的必须包含名称是Dockerfile的文件，文件内容就是上面写的那些内容
#--pull构建时总是拉去最新的镜像
# -t构建的新的image的名称

docker tag mynginx ciregistry.yourhost.cn:8443/yougroup/test/mynginx
#给新构建的image mynginx起一个别名

docker push ciregistry.yourhost.cn:8443/yougroup/test/mynginx
#如果这个别名正好是一个私有仓库的地址，就可以把这个image推送到私有仓库了
#docker私有仓库必须支持https，才能被docker正常使用


docker run -d -p 5000:5000 --name registry -v /data/registry:/tmp/registry registry
#启动registry server docker容器，创建一个docker私有仓库
#docker私有仓库必须支持https，才能被docker正常使用，所以可以通过nginx upstream反向代理一下docker registry



11.项目实战，搭建ELK日志搜集系统

需要先安装fluentd
--------------------------------------------------------
mkdir /scripts/fluentd -p
cd /scripts/fluentd

cat  > Dockerfile << 'EOF' 
FROM fluent/fluentd:v0.12-onbuild

ENV TZ=Asia/Shanghai
RUN echo "http://mirrors.aliyun.com/alpine/v3.5/main" >/etc/apk/repositories \ 
        && echo "http://mirrors.aliyun.com/alpine/v3.5/community" >>/etc/apk/repositories \
        && apk add --update tzdata \
        && apk add curl \
        && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
        && echo $TZ > /etc/timezone

RUN apk add  --virtual .build-deps \
        sudo build-base ruby-dev \
&& sudo gem install \
        fluent-plugin-elasticsearch \
&& sudo gem sources --clear-all \
&& apk del .build-deps \
&& rm -rf /var/cache/apk/* \
          /home/fluent/.gem/ruby/2.3.0/cache/*.gem
EOF

mkdir plugins -p
mkdir -p /srv/volume/fluentd/

cat > /scripts/fluentd/fluent.conf << 'EOF'
<source>
  @type forward
  port  24224
  bind 0.0.0.0
</source>

<filter docker.**>
  type parser
  format json
  time_format %Y-%m-%dT%H:%M:%S.%L%Z
  key_name log
  reserve_data true
</filter>

<filter docker.yougroup-test-nginx**>
  type record_transformer
  enable_ruby true
  <record>
    "@timestamp"  ${Time.now.strftime("%Y-%m-%dT%H:%M:%S.%L%z")}
    "@target_index" "pipipa-bbb"
  </record>
</filter>

<match docker.**>
  @type elasticsearch
  hosts yougroup-test-elasticsearch:9200

  target_index_key @target_index
  index_name _default_index
  type_name fluentd

  include_tag_key true
  tag_key DockerName

  flush_interval 1s
  buffer_type memory
</match>
EOF


docker build --no-cache --pull -t yougroup/tools/fluentd ./

mkdir -p /srv/volume/fluentd/log/
mkdir -p /srv/volume/fluentd/scripts/
docker service rm yougroup-test-fluentd
docker service create \
    --name yougroup-test-fluentd \
    --network my-network \
    --mount type=bind,src=/srv/volume/fluentd/log/,dst=/fluentd/log/ \
    --mount type=bind,src=/scripts/fluentd/fluent.conf,dst=/fluentd/etc/fluent.conf \
    --mount type=bind,src=/srv/volume/fluentd/scripts/,dst=/fluentd/scripts/ \
    --env FLUENTD_CONF=fluent.conf \
    --publish "mode=host,published=24224,target=24224" \
yougroup/tools/fluentd



安装nginx
--------------------------------------------------------
mkdir /scripts/nginx -p
cd /scripts/nginx

cat > nginx.conf << 'EOF'
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include      /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format json 
                '{"times":"$time_local",'
                '"remoteuser":"$remote_user",'
                '"remoteip":"$remote_addr",'
                '"bodysize":$body_bytes_sent,'
                '"requesttime":$request_time,'
                '"upstreamtime":$upstream_response_time,'
                '"upstreamhost":"$upstream_addr",'
                '"http_host":"$host",'
                '"request":"$request",'
                '"url":"$uri",'
                '"xff":"$http_x_forwarded_for",'
                '"referer":"$http_referer",'
                '"@target_index":"yougroup-magiceye-backend",'
                '"@timestamp":"$time_iso8601",'
                '"status":$status}';
    access_log  /var/log/nginx/access.log  json;
    sendfile        on;
    #tcp_nopush    on;
    keepalive_timeout  65;
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
}
EOF

docker service rm yougroup-test-nginx
cat > service.sh << 'EOF'
#!/bin/bash
set -xe
export IMAGE_NAME=yougroup/test/nginx
docker service create \
    --name yougroup-test-nginx \
    --network my-network \
    --replicas 1 \
    --publish "mode=host,published=8080,target=80" \
    --log-driver=fluentd \
    --log-opt=fluentd-address=192.168.2.212:24224 \
    --log-opt=tag=docker.{{.Name}} \
    --mount type=bind,src=/scripts/nginx/nginx.conf,dst=/etc/nginx/nginx.conf \
nginx
EOF


安装elasticsearch
--------------------------------------------------------
mkdir /scripts/elasticsearch -p
cd /scripts/elasticsearch
cat > Dockerfile << EOF
FROM elasticsearch:5.5.2
EOF

cat > service.sh << 'EOF'
#!/bin/bash
set -xe
export IMAGE_NAME=yougroup/test/elasticsearch
docker build --no-cache --pull -t $IMAGE_NAME ./
#docker service rm yougroup-test-elasticsearch
mkdir /srv/volume/es/data/ -p
docker service create \
    --name yougroup-test-elasticsearch \
    --network my-network \
    --replicas 1 \
    --mount type=bind,src=/srv/volume/es/data,dst=/usr/share/elasticsearch/data \
    --publish "mode=host,published=9200,target=9200" \
$IMAGE_NAME
EOF


安装kibana
-----------------------------------------------------
mkdir /scripts/kibana -p
cd /scripts/kibana
cat > Dockerfile << EOF
FROM kibana:5.5.2
EOF

cat > service.sh << 'EOF'
#!/bin/bash
set -xe
export IMAGE_NAME=yougroup/test/kibana
docker build --no-cache --pull -t $IMAGE_NAME ./
#docker service rm yougroup-test-kibana
mkdir /srv/volume/kibana/ -p
docker service create \
    --name yougroup-test-kibana \
    --network my-network \
    --replicas 1 \
    --publish "mode=host,published=80,target=5601" \
    --mount type=bind,src=/srv/volume/kibana/kibana.yml,dst=/etc/kibana/kibana.yml \
$IMAGE_NAME
EOF

mkdir -p /srv/volume/kibana
cat > /srv/volume/kibana/kibana.yml << EOF
server.host: '0.0.0.0'
elasticsearch.url: 'http://yougroup-test-elasticsearch:9200/'
EOF