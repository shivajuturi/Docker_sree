# All Docker Commands

#### Docker Day1

```
curl https://get.docker.com/ | bash

docker version
docker -H ip_address ps ( for connect to remote server )
docker network ls
docker images
docker rmi  images_name
docker images --digests ( it shows image and size of image )
(nginx:stable-perl
DIGEST:sha256:75adbf6a592c3d79572069cfad0b799cb6734a1f5951ea3cbafe98241d45e44a
)
docker images inspect  nginx:latest
docker pull nginx@sha256:75adbf6a592c3d79572069cfad0b799cb6734a1f5951ea3cbafe98241d45e44a

docker image inspect <Image_id>
docker image history <Image_id>
```
### Docker Day 2 and Day 3
- create image file
```

docker build -t juturi/ubuntu-nginx:latest -f mydockerfile .
docker history  <Image_Id>
docker ps  ( show only running Containers)
docker ps -a ( show running and  existed  Containers)

docker stop $(docker ps -aq)
docker rm $(docker ps -aq)

docker run -dit juturi/ubuntu-nginx:latest

docker run -it juturi/ubuntu-nginx:latest

docker run --rm -dit juturi/ubuntu-nginx:latest

docker exec -it <nameofcontainer> /bin/bash ( CTRL + p + q)

docker run -dit --name  myutils -p 8000:80  image_name

docker stop Container_name
docker start Container_name
docker pause  Container_name
docker top Container_name

docker login -u repositery_name Login_server  -p xxx(access_key)
docker push Login_server/shiva-juturi/utils:latest  ( it push to repositery )

```

#### Docker Day 4
- Docker File
```
FROM ubuntu:20.04
LABEL  owner="shiva.juturi@gmail.com"
LABEL version="1.0"
ARG  VERSION='1.2.9'
RUN apt update
RUN apt install -y nginx jq unzip curl wget
COPY index.html /var/www/html/index.nginx-debian.html
COPY style.css /var/www/html/style.css
COPY scorekeeper.js /var/www/html/scorekeeper.js
add  https://releases.hashicorp.com/terraform/${VERSION}/terraform_${VERSION}_linux_amd64.zip  /usr/local/bin/terraform.zip
RUN cd  /usr/local/bin/  &&  unzip terraform.zip && rm -rf *zip
RUN terraform --version
EXPOSE 80
CMD ["nginx", "-g","daemon off;"]
ENV  AWS_ACCESS_KEY_ID="AKIAV5KD6HDRHTQ2TWPDDDDD"
ENV AWS_SECRET_ACCESS_KEY="XtIcXV200IaSSjaihp0KRGGGGGGGGGGGGG+H0KQFrlRjyjDR3MLDz"
RUN mkdir /app
RUN groupadd -r user && useradd -r -g user user
WORKDIR /app
ENTRYPOINT ["ping","-c 3"]
CMD [ "wwww.facebook.com"]

```

#### Docker 5
- bind Mounts
```
docker run --rm -dit --name myutils -v /root:/ihacked --network mysql

/var/run/docker.sock  ( docker install location )

inside container we will mount docker daemon
docker run --rm -dit --name myutils  -v /var/run/docker.sock:/var/docker/docker.sock --network none  shiva/utils:latest
docker exec -it Container_name /bin/bash
# docker ps it will work

```

- While network  type  is HOST no need to port expose ( what ever port is there same port it will exopse )
```
docker run --rm -dit --name myutils --network host
netstat -nltp
```
#### Docker Network

```
docker run --rm -dit --name mysql -v mysql:/var/lib/mysql --network mysql_network -e MYSQL_ROOT_PASSWORD=India@23 -p 3306:3306 mysql:8.0.30

docker inspect mysql | grep -i "IPAddress"

docker network ls

docker network create mysql_network --driver=bridge

docker network connect mysql_network  <Container_name>
docker network prune(  remove unsused n/w)

```

#### Docker Volume
```
docker volume ls

systemctl stop docker.service

vi /usr/lib/systemd/system/docker.service ( added 2nd line  )
#ExecStart=/usr/bindockerd -H fd://  --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bindockerd -H fd:// -g DockerData --containerd=/run/containerd/containerd.sock

DockerData ( Mount point name )

sudo rsync -aqxP  /var/lib/docker /DockerData ( copy the data to /Dockerdata)

sudo systemctl daemon-reload
sudo systemctl start docker

docker volume ls
docker run --rm -dit --name nginx -v Volume_name:/usr/share/nginx/html -p 8000:80 image_name:version

docker volume create mysqldb
docker volume ls

```

## Docker Swarm

```
docker info
docker info | grep -i Swarm ( active = swarm is enabled , inactive=swarm is disabled  )
docker node ls ( this cmd will work on master node only )


docker swarm init

docker node ls ( this cmd will work on master node only )
docker swarm join-token (worker/manager)

docker swarm join-token manager (enter it will give code and execute in maser node )

docker swarm join-token worker  (enter it will give code and execute in worker  node )

docker service create  --name nginx002 --replica 3 --constraint node.hostname== ip of node  -p 8090:80 nginx:latest
docker service ls


docker ps

docker node demote  ip_of_node
docker service create  --name nginx002 --replica 3 -p 8077:80 nginx:latest

docker service scale nginx002=6
docker service scale nginx002=2





```

### Docker swarm SECRETS -configs - GlobalMode



```
secret ( sesitive data)
openssl rand -base64 12 | docker secret create db_root_password -
openssl rand -base64 12 | docker secret create db_dba_password -

 echo 'India@234' | docker secret create india
 docker secret ls

 docker secret inspect  india

 docker service create  --name nginx01  --secret india --p 8000:80 replicas 3 image_name:VERSION

 docker service ls

 docker ps -a

 docker exec -it Container_name /bin/bash

 cat /run/secret/india

 docker stack deploy  -c file_name_of yaml  stack_name
 docker stack ls
 docker stack service service_name
 docker stack ps


 docker config create nginxindex index.examples ( non sesitive data)
 docker config inspect nginxindex index.examples

 ConfigMaps : daemonsets





```
