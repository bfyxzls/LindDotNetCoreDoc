[回到目录](http://www.cnblogs.com/lori/p/7154409.html)
## 环境部署
### 需要部署的工具
1. [x] Docker
2. [x] Sqlserver
1. [x] Mysql
1. [x] Nginx
4. [x] Redis
5. [x] Mongodb
6. [x] Rabbitmq
7. [x] Solr
#### Docker安装
```bash
yum -y install docker-ce-17.06.0.ce
mkdir -p /lib/systemd/system/docker.service.d
cat > /lib/systemd/system/docker.service.d/docker.conf << 'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --default-ulimit nofile=65535
EOF
mkdir -p /etc/docker/
mkdir -p  /srv/docker/
cat > /etc/docker/daemon.json << EOF
{
    "dns": ["114.114.114.114"],
    "hosts": ["unix:///var/run/docker.sock"],
    "registry-mirrors": ["https://0sr73mco.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl start docker
systemctl enable docker
docker info
```
#### Sqlserver安装
直接在微软下载sqlserver x64位安装程序即可，注意自己设置的sa密码
#### Mysql安装
可以在docker里跑mysql，方便，快捷！
```bash
sudo docker run --name first-mysql -p 3306:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d mysql
```
#### Nginx安装
```bash
docker service rm yougroup-test-nginx
cat > service.sh << 'EOF'
#!/bin/bash
set -xe
export IMAGE_NAME=test/nginx
docker service create \
    --name test-nginx \
    --network my-network \
    --replicas 1 \
    --publish "mode=host,published=8080,target=80" \
    --log-driver=fluentd \
    --log-opt=fluentd-address=192.168.2.212:24224 \
    --log-opt=tag=docker.{{.Name}} \
    --mount type=bind,src=/scripts/nginx/nginx.conf,dst=/etc/nginx/nginx.conf \
nginx
EOF
```
#### Redis安装
```bash
runoob@runoob:~/redis$ docker run -p 6379:6379 -v $PWD/data:/data  -d redis:3.2 redis-server --appendonly yes
```
#### Mongodb安装
```bash
runoob@runoob:~/mongo$ docker run -p 27017:27017 -v $PWD/db:/data/db -d mongo:3.2
cda8830cad5fe35e9c4aed037bbd5434b69b19bf2075c8626911e6ebb08cad51
runoob@runoob:~/mongo$
```
#### Rabbitmq安装
```bash
docker service create \
    --name lind-rabbitMq \
    --network yougroup-network \
    --replicas 1 \
   --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 \
   --publish 15671:15671 \
    --publish 15672:15672 \
  rabbitmq:management
```
#### Solr安装
```bash
mkdir mycores
$ sudo chown 8983:8983 mycores
$ docker run -d -p 8983:8983 -v $PWD/mycores:/opt/solr/server/solr/mycores solr solr-precreate hello
```
待续……
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)