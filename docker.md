1. Docker��ʲô��
Docker��һ����������ṩ����������
����Docker�⣬����CoreOS�Ŷ��Ƴ���rkt���������Ҳ�ṩ��������

2. ����������ʲô��

 
���ȣ�����ʲô����������Ǳ��밲װ�ڲ���ϵͳ֮�ϵģ�linux��windows
��ͼ���������ǲ���ϵͳ

�����������(����Docker)ͨ������linux����ϵͳ�ں˵�LXC,Cgroup,Namespace�ȼ�����ʵ���˽���������ϵͳ��Դ��Ч���ֵ����������У���Щ���������������(container)
��ͼ��������װ�ļ�װ���������


3.������ȴ�ͳ���������ʲô����
��ͳ����������������������ϵͳ����������Ĳ���ϵͳ�����������еĳ���ռ���ڴ��⣬����ϵͳ����ռ���ڴ棻�����˵���������Թ���Kernel�ķ�ʽʵ�֣�����û���������
��������Դ����Ч��Ҫ����������Ϊ�����ǹ����ں˵ģ�����������⻯����һ���������ں�
��������أ���Ϊ��Ҫ��װ�����Ĳ���ϵͳ�������˵�����ͺ��ᣬһ�������ľ���ֻ�м�ʮ����������


4. ������ʲô�ô�
���Կ��ٲ�����Ϊ������
���Ա�֤һ���Ե�Ӧ�ó��򽻸�����Ϊ�����ᣬ�����ܰѳ������л���(jvm,nginx��)�ʹ���һ�����������ڣ�ֱ�ӽ���������������

5. ʹ��������Ҫע��ʲô?��Ҫ�ǲ�Υ��������ʲô�ô�
��������Ӧ������������,��Ӧ���������ﰲװ���������,��ľ������Կ��ٷַ�����������Ӧ�������
��������Ӧ���ǿ���ʱ�����ݻ��ؽ���,����ʱ��(����Dockerfile��swarm��ʵ��)����������Ҫ��֤һ���Ե�Ӧ�ó��򽻸�
��������Ӧ������״̬��(�������Ϊĳ�������ҵ����ؽ���,��Ӧ��Ӱ�쵽����������)����������Ҫ��֤һ���Ե�Ӧ�ó��򽻸�
��Ҫ�����ݴ洢��������,���ǳ������Ӧ�÷��õ�������,��Ϊ���ս����İ汾����������Ҫ��֤һ���Ե�Ӧ�ó��򽻸�
��Ҫ���������е�������commit����,��Ϊ���ǲ����ظ��ģ�ʼ��ʹ��Dockerfile����������Ҫ��֤һ���Ե�Ӧ�ó��򽻸�
��Ҫֻʹ��"latest"��ǩ,�Ӵ�������ļ����м�������һ������ġ�latest���汾,ʹ�ñ�ǩ���Է�����ٺͻع��汾���⣻��������Ҫ��֤һ���Ե�Ӧ�ó��򽻸�
��Ҫ��һ�����������ж������,Docker�Ļ�����Healthy��ֻ֧�ֵ�һ������;��������������ȶ���
������Ҫ����IP��ַ,����docker swarmʹ��services name����ͨѶ����������������ȶ���

PS:��������˵�������Ҳ��ʵ�ֿ��ٲ����һ���Ե�Ӧ�ó��򽻸������ǰѳ������л���(jvm,nginx��)�������һ�������������,������������������˵��̫����

6.Docker��������̬����
�������Docker��;���ޣ��޷����ظ���ĸ��أ������Ҫ֧�ֲַ�ʽ��Ⱥ�������
Ŀǰ������Docker��Ⱥ�����У�k8s,swarm,mesos
���ǹ�˾�õ�swarm,����Docker�ٷ��ļ�Ⱥ�������
k8s,ȫ��Kubernetes�������������������֧������google
mesos,�����������������֧������ apache �����

���ǹ�˾ѡ�����Docker+swarm�Ľ��������ԭ����swarm������Լ򵥣�����Ŀǰ���������ǵ�ʹ������

PS:�������ַ�����Ϊ�˽��Docker�ķֲ�ʽ�������⣻����𽥷�չ�˸��Ե���̬Ȧ���������𣬸߿��ã�Ǩ�ƣ���־�Ѽ�����ص�


7.����ʹ�����ּ�Ⱥ���������������Ȱ�װDocker���
   ����ʾ���ǻ���centos7,���ڰ����ƣ���װDocker�������ĵط���װע���޸������ɫ����
name=A01
hostnamectl set-hostname $name

yum remove -y docker docker-common docker-selinux docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum list docker-ce.x86_64  --showduplicates | sort -r
#�г����԰�װ��docker�汾
yum -y install docker-ce-17.06.1.ce
#��װָ���汾��docker

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
#"registry-mirrors": ["https://registry.docker-cn.com"]���й�docker hubר�õ�ַ��docker�ٷ��ṩ

systemctl daemon-reload
systemctl start docker
systemctl enable docker
docker info

7. Docker�İ汾

2016 �� 7 �� 29 ��  Docker 1.12����
2017 �� 1 �� 18 ��  Docker 1.13����

2017 �� 3 �� 2 �գ�Docker �ٷ�������һƪ blog ��������ҵ�浽�����汾Ҳ��1.13.xһԾ��17.03��
֮��Docker ��ÿ�·���һ�� edge �汾(17.03, 17.04, 17.05...)��ÿ�����·���һ�� stable �汾(17.03, 17.06, 17.09...)��
��ҵ��(EE) �� stable �汾�ű���һ�£���ÿ���汾�ṩһ��ά����


Docker �� Linux ���а������ֿ�Ҳ����ǰ��https://apt.dockerproject.org / https://yum.dockerproject.org 
���ΪĿǰ�� https://download.docker.com/������������Ϊ docker-ce(������) �� docker-ee(��ҵ��)��
�ɵĲֿ�Ͱ���(docker-engine)���ɿ���ʹ�ã�����ȷ��ʲôʱ��ᱻ������docker-engine �İ汾��Ҳ�����17.03.0~ce-0���ֵİ汾�š�

Docker v17.03.0-ce �汾�������ݺ����ص�ַ��鿴������־��
��Դ�� http://www.oschina.net/news/82496/docker-1-17-03
��Ϣ��Դ��http://www.oschina.net/news/82496/docker-1-17-03


��������
docker engine   docker�������
docker machine  docker�ٷ����𹤾ߣ����cloud���������簢�aws�ȣ�����Զ�̲���docker,swarm�ȣ�������cloud�Ľӿ�
docker compose �ٷ����Ź��ߣ�����ʵ��ͬʱ������������������ϵ����������дDockerfile�ķ�ʽ������������ϵд��yaml�ļ���
docker swarm �ٷ���Ⱥ�����ߣ��ӿں�docker��������һ��
docker registry �ٷ��ľ���ֿ⣬�������Harbor/Portusʹ�ã�����web�������
images ����һ���ɸ����ٷ���վ�ṩ�ģ�����alpine����centos����nginx���񣬿������Ϊ����ƽʱװϵͳʱ��ϵͳ��ISO�ļ�
container ����������һ��������Ŀ�����ṩ������������������Ҫ��װһЩ���ݣ���Щ���ݾ�����images�ṩ��

PS�� images��ʽ��nginx:1.13,nginx:latest ǰ����image����,������tag,tag�����Զ���,��һ�㶼�Ǹ��汾��;
                                ���û��ָ��tag,��Ĭ����latest,��nginx,��nginx:latestһ����˼


8.������Docker�ĳ�������

ps -ef | grep dockerd
#docker����Ľ��̣�dockerd��Docker�����server�˽���

docker info
#docker������Docker�����client�˽���,ͨ�������й���dockerd server�˵�

docker pull alpine
#�������µ�alpine image

docker run -i -t alpine echo "hello world"
docker run -i -t alpine /bin/sh
#docker run����������һ������
#����alpine��shell�󣬲�������cat /etc/os-release�鿴alpine�汾��
#-i�������Ƕ������ڵ�STDIN���н���,-t��������ָ��һ��α�ն˻��ն�

docker images
docker images | grep alpine
docker images --no-trunc
docker images -q
docker images --help
#�鿴images��Ϣ

docker run -d alpine /bin/sh -c "while true; do echo hello world; sleep 1; done"
#docker run����������һ��������-d�ں�̨����

docker ps
#�鿴�������е�����
docker ps -a
#�鿴���е������������������еģ��Ѿ��˳��ģ���ͣ�ĵȵ�

docker ps -q
#ֻ��ʾ�������е������Ķ�ID,ǰ12λ

docker ps --no-trunc
#���ض���ʾ���е���Ϣ

docker run -d  --name testnginx -m 1g -p 8080:80 -v /volume/nginx.conf:/etc/nginx/nginx.conf nginx
docker images | grep nginx
#ֱ������һ��nginx���������Զ�����nginx����
#nginx �����ṩnginx����Ĭ�϶˿���80�˿ڣ��������80�˿�ֻ����������е������ڲ��õ�
#��Ȼ���nginx����������һ������ϵͳ��(������centos7)�������������ϵͳҲ�޷�ֱ�ӷ������nginx������80�˿�
#-p 8080:80�ǰ�nginx������80�˿�ӳ�䵽����ϵͳ��8080�˿���
#��˿���ֱ�ӷ��ʲ���ϵͳ��8080�˿�������nginx,curl 127.0.0.1:8080
#-m 1g�������nginx����������ʹ������������ϵͳ1G�������ڴ�
#--name testnginx �����������nginx������һ������
#-v /volume/nginx.conf:/etc/nginx/nginx.conf������������ϵͳ��ĳ���ļ����ص������ڲ���ĳ��Ŀ¼����ȻҲ���Թ���Ŀ¼

docker logs e23dd06d1c50
docker logs testnginx
#��ӡ��һ�������ı�׼�����־,������������ƻ�id

docker logs -f testnginx
#��ͷ��ʼ�����������־������shell��tail -f 1.txt����

docker logs --tail 100 -f testnginx
#ֻ��ʾ�����100����־����������������ɵ���־

docker logs --tail 100 -f -t testnginx
#ֻ��ʾ�����100����־����������������ɵ���־,-t��ÿ����־����һ��ʱ���

docker exec -it testnginx /bin/sh
#����һ���������е�������shell

docker diff  testnginx
#���Բ鿴һ���������ڲ��ļ��仯������ǳ�����
#A-added,D-deleted,c-changed


docker rm affectionate_kepler
docker rm -f testnginx
#-f ǿ��ɾ��һ���������е�����

docker system df
#�鿴docker�и���ռ�ö��ٿռ䣬

docker system df -v
#���Բ鿴������ϸ����Ϣ

docker system events
docker system events --since 2016
#�鿴dockerϵͳ��־����������������������Ϣ

docker system prune -af
#����docker���õľ���������������

docker system prune -af --volumes
#17.06.1Ĭ�ϲ���ɾ��volumes����Ҫ���ⶨ��

docker image prune
#ɾ�����õľ���
docker container prune
#ɾ�����õ�����
docker volume prune
#ɾ�����õľ�
docker network prune
#ɾ�����õ�����


docker stats $(docker ps --format={{.Names}})
docker stats --format 'table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}\t{{.PIDs}}'
docker stats --all --format "table {{.ID}}\t{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker stats --all --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
#�鿴docker stats״̬
#--format��ʾ��Щ��
#-all��ʾ����������Ϣ��Running and Stopped

docker stats $(docker ps --format={{.Names}}) --no-stream  
#�鿴docker stats״̬����ˢ��

docker stats testnginx
#��ʾָ����������Ϣ



9. docker swarm��Ⱥ

swarm��ʲô��
swarm��Ⱥ����һ����Ⱥ������ƽ�swarm����װ��֮���ٰ���Щ�����汾��dockerd���뵽��Ⱥ��������swarm����ͳһ����
����dockerd��������swarm���ڰ�װdocker�������ʱ���Ѿ���װ��swarm��˭��swarm��docker�Լ��Ƴ��ļ�Ⱥ��

1. ����swarm��Ⱥ

docker swarm init --advertise-addr 192.168.2.212
#��ʼ��docker swarm��Ⱥ�����ȴ�����һ���ڵ㣬ip��ַ�ǽڵ�IP����һ���ڵ���swarm��manager�ڵ㣬����swarm��Ⱥ��Ϣ

docker swarm join-token manager
#���ɼ���swarm manager�ڵ��������Ҫ����swarm��dockerd�ڵ�������

docker swarm join-token worker
#���ɼ���swarm work�ڵ�����,work�ڵ㲻���漯Ⱥ��Ϣ������Ҫ����swarm��dockerd�ڵ�������

PS:
��Ȼ��swarm����ͳһ������,������ô���dockerd,�ͱ��뱣֤swarm�ĸ߿�����,����swarm�ļ�Ⱥ��Ϣ����Ҫ�����������ڵ�������֤�߿���
����swarm�ļ�Ⱥ��Ϣֻ������manager�ڵ��ϣ�������ٴ�������manager�ڵ�;
manager�ڵ��״̬��Leader��Reachable����״̬��״̬��Leader�Ľڵ������ڵ㣬ֻ��һ���������manager�ڵ㶼��Reachable״̬
work�ڵ㲻���漯Ⱥ��Ϣ��ֻ����ܲ�����manager�ڵ㷢�������������紴����������������
manager�ڵ��work�ڵ�Ĭ�϶����Դ�����������������

docker swarm leave
#�˳�swarm��Ⱥ������Ҫ�˳�swarm��dockerd�ڵ�������

���������ֻ����manager�ڵ���ִ��
-----------------------------------------------------------------------------------------------------------
docker node ls
#�г�����swarm�Ľڵ㣬manager��work�ڵ�

docker node inspect z48ts3fx22c8snzazjac99lnh --pretty
#�鿴ĳ���ڵ����Ϣ

docker node update --availability drain z48ts3fx22c8snzazjac99lnh
docker node update --availability active z48ts3fx22c8snzazjac99lnh
docker node update --availability pause z48ts3fx22c8snzazjac99lnh
#����ĳ���ڵ�Ŀ����ԣ�drain��ʾ����ڵ㲻�ٽ�������manager�ڵ�����񣬲��ҽ���ǰ�ڵ������е�����ȫ�����٣��ڵ�ά��ʱʹ��
#����ĳ���ڵ�Ŀ����ԣ�pause��ʾ����ڵ㲻�ٽ�������manager�ڵ������,�����е��������ᱻ����
#����ĳ���ڵ�Ŀ����ԣ�active��ʾ����ڵ��������manager�ڵ������,��������������������������

docker node promote z48ts3fx22c8snzazjac99lnh
#��һ���ڵ�������manager�ڵ�
docker node demote z48ts3fx22c8snzazjac99lnh
#��һ���ڵ㽵����work�ڵ�
docker node rm z48ts3fx22c8snzazjac99lnh
#�Ӽ�Ⱥ������ɾ��һ���ڵ㣬���ͬʱ��Ҫ�˳�swarm��dockerd�ڵ�������docker swarm leave

2. ����swarm����

docker network ls
docker network create --driver overlay my-network
docker network ls
docker network inspect my-network
#ע�⿴����ʱ��������Driver��overlay������
#overlay���������swarm��Ⱥ��λ�ڲ�ͬ�������ϵ�����ֱ�ӻ���ͨѶ
#ֻ�д�����ͬ��overlay������������ܻ���ͨѶ����ͬ��overlay������������ܻ���ͨѶ


3. ����һ��swarm service

������dockerd���������ķ�����docker runֱ�Ӵ���һ����������
swarm���������ķ������ȴ���һ��service,Ȼ��ָ��������������е������ĸ����ĸ��������ı���Ҳ������dockerd�ڵ��ϴ�����������������swarm���й����

docker service create --replicas 3 --name helloworld nginx
docker service ls
docker service ps helloworld
docker service logs -f 5bwrqvtr8fty
docker ps
docker service inspect helloworld --pretty
docker network inspect ingress
#����serviceʱ��û��ָ��--network networknameʱ��Ĭ��ʹ��ingress��Ĭ��overlay����

docker service scale helloworld=5
#��̬����һ��service��replicas������������

docker service ls
docker service ps helloworld
docker service update --image nginx:1.12 helloworld
#����һ��service�������Ǹ���helloworld���service��image����
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
#json��ʽ
docker service ps yougroup-test-nginx --no-trunc 
#��ʾps��������Ϣ

docker service update --image nginx:1.12 yougroup-test-nginx
#��yougroup-test-nginx��image�޸ĳ�nginx:1.12�����´���汾ʱҲ��ֱ�Ӹ��´���õ��µ�image

docker service update --rollback yougroup-test-nginx
#--rollback���ֶ��ع�����ѷ���ع��������docker service update֮ǰ�����ã������Ѹ��º��image nginx:1.12�ع���nginx:1.13
#��ʵ��֤������ʹ����ǰ��ʹ�õ�image����tagһ�£���Ȼ���Իع���֮ǰ�����ã���������ǰ��serviceʹ�õ�image,����Ȼ����ÿ��buildʱʹ�ò�ͬ��tag


10. Dockerfile��д�淶
ʲô��Dockerfile��
Dockerfile����һϵ������Ͳ������ɵĽű�����Щ����Ӧ���ڻ����������մ���һ���µľ������Ǽ��˴�ͷ��β�����̲�����ļ��˲�����
���ǿ�����Dockerfile�ﶨ��һЩ�����Լ�������,���绷������,�����ļ�,����ȵ����մ����һ���µ�image���������ǵ�ʹ��
������дDockerfile��Ŀ�ľ��ǰ��������image��������ʹ��

Dockerfile��FROM���ʼ�������Ÿ����߸��ַ���������Ͳ����������Ϊһ���µĿ������ڴ��������ľ���


FROM nginx:1.12.1-alpine
LABEL "maintainer"="14339989@qq.com" \
           "aliyun.logs.nginx=/var/log/nginx/access2.log"
#LABEL��ͨ��docker inspect containerid���鿴��
EXPOSE 80 443
ENV myName="John Doe" \
        myDog=Rex \
        TZ=Asia/Shanghai \
        myCat=fluffy
#ͨ��image���������������Կ���ENV�Ļ�����������������Ҳ����ͨ��docker service --env������
WORKDIR /tmp/    
#ADD,COPY,CMD,ENTRYPOINT,RUN����������Ŀ¼��ִ��
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

#ADD,COPY�������Կ���Dockerfile����Ŀ¼����ļ�����Ҫ�Ļ�����ͨ�����������ȿ�����Dockerfile����Ŀ¼
#���Ŀ��λ�ò���"/"��β����Ŀ����ᱻ��Ϊ��һ�������ļ������ᵱ��Ŀ¼����
#���Ŀ��λ�ò����ڣ�����Զ�����
#����ֻ��COPY

HEALTHCHECK --interval=5s --timeout=3s CMD curl -f http://localhost/ || exit 1
#���5s���һ�Σ�ÿ�μ��3s��û�������������жϼ��ʧ�ܣ�Ĭ�ϼ������3�κ��ж�����Ϊunhealthy״̬,Ȼ������ٵ�ǰ�������ؽ��µ��������������


HEALTHCHECK --start-period=30s --interval=5s --timeout=3s --retries=16 CMD /bin/sh /scripts/check.sh $URL
#--start-period=30s�������30s��ʼ�����5s���һ�Σ�ÿ�μ��3s��û�������������жϼ��ʧ�ܣ�Ĭ�ϼ������3�κ��ж�����Ϊunhealthy״̬
#$URL�����������������Dockerfile��ֱ�Ӷ��壬Ҳ�����ڴ���serviceʱ����

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
#./Ŀ¼��ı������������Dockerfile���ļ����ļ����ݾ�������д����Щ����
#--pull����ʱ������ȥ���µľ���
# -t�������µ�image������

docker tag mynginx ciregistry.yourhost.cn:8443/yougroup/test/mynginx
#���¹�����image mynginx��һ������

docker push ciregistry.yourhost.cn:8443/yougroup/test/mynginx
#����������������һ��˽�вֿ�ĵ�ַ���Ϳ��԰����image���͵�˽�вֿ���
#docker˽�вֿ����֧��https�����ܱ�docker����ʹ��


docker run -d -p 5000:5000 --name registry -v /data/registry:/tmp/registry registry
#����registry server docker����������һ��docker˽�вֿ�
#docker˽�вֿ����֧��https�����ܱ�docker����ʹ�ã����Կ���ͨ��nginx upstream�������һ��docker registry



11.��Ŀʵս���ELK��־�Ѽ�ϵͳ

��Ҫ�Ȱ�װfluentd
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



��װnginx
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


��װelasticsearch
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


��װkibana
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