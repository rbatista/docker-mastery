Docker Mastery: The Complete Tollset From a Docker Captain
==========================================================
Course: https://udemy.com/docker-mastery/

# Creating and Using Containers
## Manage Multiple Containers

```bash
# starts a mysql detached container with exposed port 3306 and name db
$ docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
1525b46272e238d0514de064b8a0419adb39a85ade795402268a8562e4f57c06

# get the logs by the container name (to get the password)
$ docker container logs db

# start an detached httpd container at port 8080
$ docker container run -d --name webserver -p 8080:80 httpd
2dfab7da62b53f7557fbf019a26234a4358cf1494f5a01ab40b205c217d0dd75

# list the running conatainers
$ docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                  PORTS                               NAMES
2dfab7da62b5        httpd                         "httpd-foreground"       52 seconds ago      Up 50 seconds           0.0.0.0:8080->80/tcp                webserver
1525b46272e2        mysql                         "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes            0.0.0.0:3306->3306/tcp, 33060/tcp   db

# start an detached nginx container at port 80
$ docker container run -d --name proxy -p 80:80 nginx:alpine
0d85bafd86f4b7da6ad300fa21e806849b97cf2fa8c6eac2a8fd0dc68d363990

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
0d85bafd86f4        nginx               "nginx -g 'daemon of…"   48 seconds ago      Up 47 seconds       0.0.0.0:80->80/tcp                  proxy
2dfab7da62b5        httpd               "httpd-foreground"       6 minutes ago       Up 6 minutes        0.0.0.0:8080->80/tcp                webserver
1525b46272e2        mysql               "docker-entrypoint.s…"   12 minutes ago      Up 12 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   db

# new command from docker ps
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
0d85bafd86f4        nginx               "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp                  proxy
2dfab7da62b5        httpd               "httpd-foreground"       10 minutes ago      Up 10 minutes       0.0.0.0:8080->80/tcp                webserver
1525b46272e2        mysql               "docker-entrypoint.s…"   16 minutes ago      Up 16 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   db

# test the nginx port
$ curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# test the apache port
$ curl localhost:8080
<html><body><h1>It works!</h1></body></html>

# stop the containers
$ docker container stop db webserver proxy

# list all containers (docker container ls -a)
$ docker ps -a
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS                      PORTS               NAMES
0d85bafd86f4        nginx                              "nginx -g 'daemon of…"   9 minutes ago       Exited (0) 44 seconds ago                       proxy
2dfab7da62b5        httpd                              "httpd-foreground"       15 minutes ago      Exited (0) 44 seconds ago                       webserver
1525b46272e2        mysql                              "docker-entrypoint.s…"   21 minutes ago      Exited (0) 41 seconds ago                       db

# remove containers
$ docker container rm db webserver proxy

# List images
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              62f816a209e6        6 days ago          109MB
mysql               latest              2dd01afbe8df        2 weeks ago         485MB
httpd               latest              55a118e2a010        2 weeks ago         132MB
```

### CLI Process Monitoring

docker container top # process list in one container
docker container inspect # details of one container config
docker container stats # performance stats for all containers

```bash
# list processes running inside a container
$ docker container top db
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
sbt                 20617               20597               0                   11:12               ?                   00:00:01            mysqld

# get info about the start and configuration of a container
$ docker container inspect db
[
    {
        "Id": "9dbf52ebb2270e4bc7064d4bf5dc8db885ad2095040f3ff843185ce48ab1d41b",
        "Created": "2018-11-13T13:11:59.097599877Z",
        "Path": "docker-entrypoint.sh"
        ...
    }
]

# get a live resources utilization (like top) from all containers
$ docker container stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT    MEM %               NET I/O             BLOCK I/O           PIDS
cad409ab534e        proxy               0.00%               2.949MiB / 23.5GiB   0.01%               4.47kB / 0B         0B / 0B             2
9dbf52ebb227        db                  0.40%               375.6MiB / 23.5GiB   1.56%               4.69kB / 0B         8.19kB / 1.13GB     37
```

### Getting a Shell Inside containers (no ssh)

docker container run -it # Start new container interactively
docker container exec -it # run aditional command in a existing container

```bash
# execute a bash inside a new nginx container with a pseudo-tty (-t) and interactive (-i) kepping the session open to execute more commands
$ docker container run -it --name proxy2 nginx:alpine bash
root@4f4bcd2f689e:/# 
$ ls -al
total 72
drwxr-xr-x  33 root root 4096 Nov 13 13:30 .
drwxr-xr-x  33 root root 4096 Nov 13 13:30 ..
-rwxr-xr-x   1 root root    0 Nov 13 13:30 .dockerenv
drwxr-xr-x   2 root root 4096 Oct 11 00:00 bin
drwxr-xr-x   2 root root 4096 Jun 26 12:03 boot
...
$ exit

# the container stops when you exit the run command
$ docker container ls -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                      PORTS                               NAMES
4f4bcd2f689e        nginx               "bash"                   About a minute ago   Exited (0) 12 seconds ago                                       proxy2
cad409ab534e        nginx               "nginx -g 'daemon of…"   19 minutes ago       Up 18 minutes               0.0.0.0:80->80/tcp                  proxy
9dbf52ebb227        mysql               "docker-entrypoint.s…"   19 minutes ago       Up 19 minutes               0.0.0.0:3306->3306/tcp, 33060/tcp   db

# the default command executed when you start the nginx is 'nginx -g 'daemon of…', when we start a container with the command 'bash' we overwrite the default command. when we exit the shell the container stoped because the container run only while the start command is running

# running a ubuntu container
$ docker container run -it --name ubuntu ubuntu    
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
473ede7ed136: Pull complete 
c46b5fa4d940: Pull complete 
93ae3df89c92: Pull complete 
6b1eed27cade: Pull complete 
Digest: sha256:29934af957c53004d7fb6340139880d23fb1952505a15d69a03af0d1418878cb
Status: Downloaded newer image for ubuntu:latest
root@11063e9a2594:/#

$ apt-get update
$ apt-get install -y curl
$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>

# leave the shell and stop the container
$ exit

$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                               NAMES
11063e9a2594        ubuntu              "/bin/bash"              3 minutes ago       Exited (0) 49 seconds ago

# start a existent container interactively
docker container start -ai ubuntu
root@11063e9a2594:/# curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

Execute a interactively bash inside a running container
```bash

# docker exec run a additional process inside the container
docker container exec -it db bash
ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mysql        1  0.3  1.6 1882668 405024 ?      Ssl  13:12   0:08 mysqld
root       203  0.0  0.0  18188  3348 pts/0    Ss   13:48   0:00 bash
root       517  0.0  0.0  36640  2864 pts/0    R+   13:49   0:00 ps aux

# when you exit the bash with the exec, the container keeps running
root@9dbf52ebb227:/# exit
exit

# because when you exit the bash it will not affect the mysql command daemon
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
9dbf52ebb227        mysql               "docker-entrypoint.s…"   33 minutes ago      Up 32 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   db

# you can only run commands that exists inside the container, for example the alpine has no bash inside it, just have sh
```

## Docker Networks

- Each container connected to a private virtual network "bridge"
- Each virtual network routes through NAT firewall on host IP
- All containers ona a virtual network can talk to each other without bind the port (-p)
- Best practice is to create a new virtual network for each app:
 - network "my_web_app" for mysql and php/apache containers
 - network "my_api" for mongo and nodejs containers



```bash
# list open ports in a container
$ docker container port db           
3306/tcp -> 0.0.0.0:3306

# get the ip address from a container
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' db
10.11.1.1
```

### CLI

Show networks: docker network ls
Inspect a network: docker network inspect
create a network: docker network create --driver
Attach a network to a container: docker network connect
Detach a network from container: docker network disconnect

- bridge network: Default docker virtual network which is NAT'ed behind the Host IP
- host network: It gains performance by skiping virtual networks, but sacrifices security of container model
- none network: removes eth0 and only leaves you with localhost interface in container

network driver: Built-in or 3rd party extensions that give you virtual network features, default brigde

```bash
# create a new virtual network
$ docker network create my_app_net
e0b184f5829798650586fc5f1c2919d26142be1fc11a4d5b0ae8f8ca973553aa

$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
474b9aadb6cd        bridge              bridge              local
2091b04876df        host                host                local
e0b184f58297        my_app_net          bridge              local
174afc320f00        none                null                local

# create a new container with the my_app_net network
$ docker container run -d --name new_nginx --network my_app_net nginx:alpine

# list the containers in the my_app_net network
$ docker network inspect my_app_net --format '{{ .Containers }}'
map[45cd449939c1c8ea4f6fa23d3cd824806bc406dea28ff7cdee7ae587722ce35a:{new_nginx 13753e08ddd51db5c9f270a3ecb66410804b4e5d13277949e11c9e2310faad1b 02:42:0d:50:0a:02 13.80.10.2/24 }]

# attach a network to a existent 'proxy' container
$ docker network connect my_app_net proxy

$ docker container inspect proxy --format '{{ .NetworkSettings.Networks }}'
map[bridge:0xc4201aab40 my_app_net:0xc4201aac00]

# remove a network interface from a container
docker network disconnect my_app_net proxy

$ docker container inspect proxy --format '{{ .NetworkSettings.Networks }}'
map[bridge:0xc4201a4a80]
```

### DNS

*Forget IP's*: Static IP's and using IP's for talking to containers is an anti-pattern. Do your best to avoid it.
Docker DNS: Docker daemon has a built-in DNS server that containers use by default, the container name is automatically resolved to the container ip inside the virtual network

The default bridge network has no automatically DNS name resolution, so you cold use the --link when create a new container or create your own network.

Since Docker Wngine 1.11, we can have multiple containers on a created network respond to the same DNS address (--net-alias name)

```bash
# create a new container in our custom network 'my_app_net'
$ docker container run -d --name my_nginx --network my_app_net nginx:alpine
60edd15bb58b7cfdbb650a096215a359dffbb2c4c472b0052c32e86e17b7586f

# list the 2 containers inside the 'my_app_net' network
$ docker network inspect my_app_net --format '{{ .Containers }}'
map[45cd449939c1c8ea4f6fa23d3cd824806bc406dea28ff7cdee7ae587722ce35a:{new_nginx 13753e08ddd51db5c9f270a3ecb66410804b4e5d13277949e11c9e2310faad1b 02:42:0d:50:0a:02 13.80.10.2/24 } 60edd15bb58b7cfdbb650a096215a359dffbb2c4c472b0052c32e86e17b7586f:{my_nginx cd4afcadfe8eb9821919254f541890bc3689388bd913efeb46fa4a3d431af445 02:42:0d:50:0a:03 13.80.10.3/24 }]


docker container exec -it my_nginx ping new_nginx                        
PING new_nginx (13.80.10.2): 56 data bytes
64 bytes from 13.80.10.2: seq=0 ttl=64 time=0.079 ms
64 bytes from 13.80.10.2: seq=1 ttl=64 time=0.125 ms
64 bytes from 13.80.10.2: seq=2 ttl=64 time=0.108 ms

```

Assignment: CLI App Testing
```bash
$ docker container run --rm -it centos:7 bash
$ yum update curl
$ curl --version
curl 7.29.0 (x86_64-redhat-linux-gnu)

$ docker container run --rm -it ubuntu:14.04 bash
$ apt-get update && apt-get install -y curl
$ curl --version
curl 7.35.0
```

Assignment: DNS Round Robin Test
```bash
# create a virtual network
$ docker network create search_net
# create the first container attached to the created network with a network alias 'search'
docker container run -d --rm --name elastic01 --network search_net --network-alias search elasticsearch:2

# create the second container attached to the created network with a network alias 'search'
$ docker container run -d --rm --name elastic02 --network search_net --network-alias search elasticsearch:2

# list the containers
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
bdb8306851ba        elasticsearch:2     "/docker-entrypoint.…"   4 seconds ago       Up 2 seconds        9200/tcp, 9300/tcp   elastic02
5dd2560e972f        elasticsearch:2     "/docker-entrypoint.…"   27 seconds ago      Up 25 seconds       9200/tcp, 9300/tcp   elastic01

# run the command nslookup for the dns alias 'search' inside a new alpine container attached to the search_net network
$ docker container run --rm --network search_net alpine nslookup search
Name:      search
Address 1: 13.80.11.2 elastic01.search_net
Address 2: 13.80.11.3 elastic02.search_net

# execute a curl to the dns alias to get the first response from 'Hitman' elasticsearch container
$ docker container run --rm --network search_net centos:7 curl -s search:9200
{
  "name" : "Hitman",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "l8dStosLQO-job7fhOOwYg",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}

# the same curl call to the dns alias now are got from the 'Iguana' elasticsearch container
$ docker container run --rm --network search_net centos:7 curl -s search:9200
{
  "name" : "Iguana",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "F-Lh-OfVTre3is6Fw5Ba9w",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}

# remove the elasticseach containers
docker container rm -f elastic01 elastic02
```