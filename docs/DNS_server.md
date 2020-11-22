# Create a docker-compose container on an Ubuntu20.04 VM
 
## 0. How to install:
  uninstall current Docker Engine:
boburciu@dns:~/DNS_server$ ` sudo apt-get remove docker docker-engine docker.io containerd runc  `  <br/>

  install docker from upstream Docker by script:
boburciu@dns:~/DNS_server$ ` curl -fsSL https://get.docker.com -o get-docker.sh  `  <br/>
boburciu@dns:~/DNS_server$ ` sudo sh get-docker.sh  `  <br/>

 >  - install Docker Compose (run this command to download the current stable release of Docker Compose):
[boburciu@r220 ~]$ ` sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose  `  <br/>
[sudo] password for boburciu:
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   651  100   651    0     0   2206      0 --:--:-- --:--:-- --:--:--  2214
100 11.6M  100 11.6M    0     0  3735k      0  0:00:03  0:00:03 --:--:-- 4521k  

 >  - apply executable permissions to the docker-compose binary:
[boburciu@r220 ~]$ ` sudo chmod +x /usr/local/bin/docker-compose  `  <br/>
[boburciu@r220 ~]$

 >  - test docker-compose installation:
[boburciu@r220 ~]$ ` docker-compose --version  `  <br/>
docker-compose version 1.27.4, build 40524192
[boburciu@r220 ~]$

 >  - create docker.compose.yml
boburciu@dns:~/DNS_server$ pwd
/home/boburciu/DNS_server
boburciu@dns:~/DNS_server$ ls -l
total 4
-rw-rw-r-- 1 boburciu boburciu 317 Nov 22 15:01 docker-compose.yml
boburciu@dns:~/DNS_server$
boburciu@dns:~/DNS_server$ ` cat docker-compose.yml  `  <br/>
    # https://hub.docker.com/r/sameersbn/bind
    bind_DNS_server_container:
      image: sameersbn/bind:latest
      dns: 127.0.0.1
      # environment:
      #   - ROOT_PASSWORD=SecretPassword
      ports:
        - "192.168.122.64:53:53/udp"
        - "192.168.122.64:53:53/tcp"
        - "10000:10000/tcp"
      volumes:
        - /srv/docker/bind:/BM_VMs/data
boburciu@dns:~/DNS_server$

 >  - create DNS server as container:

boburciu@dns:~/DNS_server$
boburciu@dns:~/DNS_server$ ` sudo docker-compose up  `  <br/>
Pulling bind_DNS_server_container (sameersbn/bind:latest)...
latest: Pulling from sameersbn/bind
d51af753c3d3: Pull complete
fc878cd0a91c: Pull complete
6154df8ff988: Pull complete
fee5db0ff82f: Pull complete
0f1b127174c9: Pull complete
235721daf324: Pull complete
f4928d097319: Pull complete
556e764c01c5: Pull complete
0c0db0fec9f4: Pull complete
Digest: sha256:d115ce58bf4666666ad1d328ba49c291085452b5cacb910f087ee12e37d76ca7
Status: Downloaded newer image for sameersbn/bind:latest
Creating dns_server_bind_DNS_server_container_1 ... done
Attaching to dns_server_bind_DNS_server_container_1
bind_DNS_server_container_1  | Starting webmin...
bind_DNS_server_container_1  | Starting named...
bind_DNS_server_container_1  | 22-Nov-2020 15:09:50.912 starting BIND 9.16.1-Ubuntu (Stable Release) <id:d497c32>
bind_DNS_server_container_1  | 22-Nov-2020 15:09:50.912 running on Linux x86_64t

 >  - Connect to https://192.168.122.64:10000/ from CentOS host with username `root` and password `password`.

 >  - stop container with `CTRL+C`:

^CGracefully stopping... (press Ctrl+C again to force)
Stopping dns_server_bind_DNS_server_container_1 ... done
boburciu@dns:~/DNS_server$

## II. 

## Further sections
