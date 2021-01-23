# Create a docker-compose container on an Ubuntu20.04 VM
 
## 0. How to install:
 ### - uninstall current Docker Engine:<br/>
boburciu@dns:~/DNS_server$ ` sudo apt-get remove docker docker-engine docker.io containerd runc  `  

 ### - install docker from upstream Docker by script:<br/>
boburciu@dns:~/DNS_server$ ` curl -fsSL https://get.docker.com -o get-docker.sh  `  <br/>
boburciu@dns:~/DNS_server$ ` sudo sh get-docker.sh  `  <br/>

 ### - install Docker Compose (run this command to download the current stable release of Docker Compose):<br/> 
boburciu@dns:~/DNS_server$ ` sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose  `   <br/>
[sudo] password for boburciu: <br/>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current <br/>
                                 Dload  Upload   Total   Spent    Left  Speed <br/>
100   651  100   651    0     0   2206      0 --:--:-- --:--:-- --:--:--  2214 <br/>
100 11.6M  100 11.6M    0     0  3735k      0  0:00:03  0:00:03 --:--:-- 4521k   <br/>


 ### - apply executable permissions to the docker-compose binary:<br/>
boburciu@dns:~/DNS_server$ ` sudo chmod +x /usr/local/bin/docker-compose  `  <br/>
boburciu@dns:~/DNS_server$<br/>


 ### - test docker-compose installation:<br/>
boburciu@dns:~/DNS_server$ ` docker-compose --version  `  <br/>
docker-compose version 1.27.4, build 40524192 <br/>
boburciu@dns:~/DNS_server$<br/>


 ### - create docker.compose.yml<br/>
boburciu@dns:~/DNS_server$ pwd <br/>
/home/boburciu/DNS_server <br/>
boburciu@dns:~/DNS_server$ ls -l <br/>
total 4 <br/>
-rw-rw-r-- 1 boburciu boburciu 317 Nov 22 15:01 docker-compose.yml <br/>
boburciu@dns:~/DNS_server$ <br/>
boburciu@dns:~/DNS_server$ ` cat docker-compose.yml  `   <br/>
```
    # https://hub.docker.com/r/sameersbn/bind <br/>
    bind_DNS_server_container: <br/>
      restart: always <br/> 
      image: sameersbn/bind:latest <br/>
      dns: 127.0.0.1 <br/>
      # environment: <br/>
      #   - ROOT_PASSWORD=SecretPassword <br/>
      ports: <br/>
        - "192.168.122.64:53:53/udp" <br/>
        - "192.168.122.64:53:53/tcp" <br/>
        - "10000:10000/tcp" <br/>
      volumes: <br/>
        - /srv/docker/bind:/BM_VMs/data <br/>
boburciu@dns:~/DNS_server$ <br/>
```


 ### - create DNS server as container: <br/>
boburciu@dns:~/DNS_server$ <br/>
boburciu@dns:~/DNS_server$ ` sudo docker-compose up  `   <br/>
```
Pulling bind_DNS_server_container (sameersbn/bind:latest)... <br/>
latest: Pulling from sameersbn/bind <br/>
d51af753c3d3: Pull complete <br/>
fc878cd0a91c: Pull complete <br/>
6154df8ff988: Pull complete <br/>
fee5db0ff82f: Pull complete <br/>
0f1b127174c9: Pull complete <br/>
235721daf324: Pull complete <br/>
f4928d097319: Pull complete <br/>
556e764c01c5: Pull complete <br/>
0c0db0fec9f4: Pull complete <br/>
Digest: sha256:d115ce58bf4666666ad1d328ba49c291085452b5cacb910f087ee12e37d76ca7 <br/>
Status: Downloaded newer image for sameersbn/bind:latest <br/>
Creating dns_server_bind_DNS_server_container_1 ... done <br/>
Attaching to dns_server_bind_DNS_server_container_1 <br/>
bind_DNS_server_container_1  | Starting webmin... <br/>
bind_DNS_server_container_1  | Starting named... <br/>
bind_DNS_server_container_1  | 22-Nov-2020 15:09:50.912 starting BIND 9.16.1-Ubuntu (Stable Release) <id:d497c32> <br/>
bind_DNS_server_container_1  | 22-Nov-2020 15:09:50.912 running on Linux x86_64t <br/>
```


 ### - Connect to https://192.168.122.64:10000/ from CentOS host with username `root` and password `password`.<br/>


 ### - stop container with `CTRL+C`:<br/>
<br/>
^CGracefully stopping... (press Ctrl+C again to force)<br/>
Stopping dns_server_bind_DNS_server_container_1 ... done<br/>
boburciu@dns:~/DNS_server$<br/>

## I. How to make BIND container autostart once VM boots:
 

 ### -  have docker.service enabled on system startup: <br/>
boburciu@dns:~/DNS_server$ `sudo systemctl enable docker` <br/>
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install. <br/>
Executing: /lib/systemd/systemd-sysv-install enable docker <br/>
boburciu@dns:~/DNS_server$ <br/>
 

 ### -  have `restart: always` added in docker-compose.yml <br/>
 

 ### -  run once (prior to the reboot) the `docker-compose up -d`: <br/>
boburciu@dns:~/DNS_server$ <br/>
boburciu@dns:~/DNS_server$ sudo docker-compose up -d<br/>
Recreating dns_server_bind_DNS_server_container_1 ... done<br/>
boburciu@dns:~/DNS_server$<br/>
boburciu@dns:~/DNS_server$ reboot<br/>
<br/>
Last login: Mon Nov 23 17:50:42 2020 from 192.168.122.1<br/>
boburciu@dns:~$<br/>
boburciu@dns:~$ sudo docker ps -all<br/>
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                                                            NAMES<br/>
b4b574172210        sameersbn/bind:latest   "/sbin/entrypoint.sh…"   11 minutes ago      Created             192.168.122.64:53->53/tcp, 192.168.122.64:53->53/udp, 0.0.0.0:10000->10000/tcp   dns_server_bind_DNS_server_container_1<br/>
boburciu@dns:~$<br/>
boburciu@dns:~$<br/>

## II. How to configure DNS record via Webmin GUI:
 

 ### -  1. BIND Server => Create Master zone (boburciu.privatecloud.com) <br/>
 

 ### -  2. Address => add new record <br/>
 

 ### -  3. Return to zone, then Apply Configuration (reload button on top left side) <br/>
 

 ### -  4. On DNS clients: add the DNS server in /etc/resolv.conf, add "dns=none" under "[main]" in /etc/NetworkManager/NetworkManager.conf and restart NetworkManager service <br/>
[boburciu@r220 KVM-notes-proj]$ ` cat /etc/resolv.conf ` <br/>
search local boburciu <br/>
*nameserver 192.168.122.64* <br/>
nameserver 2a04:2416:640:9080:20b:ff:fe00:add0 <br/>
[boburciu@r220 KVM-notes-proj]$ <br/>
[boburciu@r220 KVM-notes-proj]$ <br/>
[boburciu@r220 KVM-notes-proj]$ cat /etc/NetworkManager/NetworkManager.conf | grep -B1 dns <br/>
[main] <br/>
dns=none <br/>
 <br/>
[logging] <br/>
[boburciu@r220 KVM-notes-proj]$ <br/>
[boburciu@r220 KVM-notes-proj]$ ` sudo systemctl restart NetworkManager.service ` <br/>
[boburciu@r220 KVM-notes-proj]$ <br/>

## III. How to configure public DNS resolving for the DNS server container on Ubuntu18.04:
 
 ### -  Following [guide](https://datawookie.netlify.app/blog/2018/10/dns-on-ubuntu-18.04/)
` sudo apt install resolvconf `  
` sudo vi /etc/resolvconf/resolv.conf.d/head ` and add the following:
```
# Make edits to /etc/resolvconf/resolv.conf.d/head.
nameserver 8.8.4.4
nameserver 8.8.8.8
```
` sudo service resolvconf restart `

 ### -  To have the DNS clients resolve short names and not FQDN, either have the DHCP server for those clients be set for DHCP Option 119, defined in [RFC3397](https://tools.ietf.org/html/rfc3397) or manually configure a ` search ` domain in ` /etc/resolv.conf `:
``` 
[boburciu@r220 ~]$ cat /etc/resolv.conf | grep search
search local boburciu
[boburciu@r220 ~]$ sudo vi /etc/resolv.conf
[sudo] password for boburciu:
[boburciu@r220 ~]$
[boburciu@r220 ~]$ sudo systemctl restart NetworkManager.service
[boburciu@r220 ~]$
[boburciu@r220 ~]$ cat /etc/resolv.conf | grep search
search local boburciu boburciu.privatecloud.com
[boburciu@r220 ~]$
[boburciu@r220 ~]$ ping rkem1
PING rkem1.boburciu.privatecloud.com (192.168.122.192) 56(84) bytes of data.
64 bytes from 192.168.122.192 (192.168.122.192): icmp_seq=1 ttl=64 time=0.170 ms
64 bytes from 192.168.122.192 (192.168.122.192): icmp_seq=2 ttl=64 time=0.096 ms
^C
--- rkem1.boburciu.privatecloud.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.096/0.133/0.170/0.037 ms
[boburciu@r220 ~]$
[boburciu@r220 ~]$ ping rkew1
PING rkew1.boburciu.privatecloud.com (192.168.122.180) 56(84) bytes of data.
64 bytes from 192.168.122.180 (192.168.122.180): icmp_seq=1 ttl=64 time=0.127 ms
64 bytes from 192.168.122.180 (192.168.122.180): icmp_seq=2 ttl=64 time=0.139 ms
^C
--- rkew1.boburciu.privatecloud.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.127/0.133/0.139/0.006 ms
[boburciu@r220 ~]$
```

 ### -  Ubuntu 18.04's _/etc/resolv.conf_ changes won't stick between reboots, so you need to add _search: [boburciu.privatecloud.com]_ in /etc/netplan/01-netcfg.yaml:
```
ubuntu@device:~$ sudo nano /etc/netplan/01-netcfg.yaml
ubuntu@device:~$ cat /etc/netplan/01-netcfg.yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: yes
      nameservers:
        addresses: [192.168.122.64, 8.8.8.8]
        search: [boburciu.privatecloud.com]
ubuntu@device:~$
ubuntu@device:~$ sudo netplan apply
ubuntu@device:~$
ubuntu@device:~$
ubuntu@device:~$ cat /etc/resolv.conf
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "systemd-resolve --status" to see details about the actual nameservers.

nameserver 192.168.122.64
nameserver 127.0.0.53
search boburciu.privatecloud.com
ubuntu@device:~$
```

## IV. T-shoot and restart DNS server with same config:
 ### -  If the ports needed are said to be already used, kill those processes:

boburciu@dns:~/DNS_server$ ` sudo docker-compose up `
```
Starting 8340069eaf31_dns_server_bind_DNS_server_container_1 ... 
Starting 8340069eaf31_dns_server_bind_DNS_server_container_1 ... error

ERROR: for 8340069eaf31_dns_server_bind_DNS_server_container_1  Cannot start service bind_DNS_server_container: driver failed programming external connectivity on endpoint 8340069eaf31_dns_server_bind_DNS_server_container_1 (35cb9b2155ce6b393bb721a04878cecb764c0f3ab5838af174b00249a648a2a3): Error starting userland proxy: listen tcp 192.168.122.64:53: bind: address already in use

ERROR: for bind_DNS_server_container  Cannot start service bind_DNS_server_container: driver failed programming external connectivity on endpoint 8340069eaf31_dns_server_bind_DNS_server_container_1 (35cb9b2155ce6b393bb721a04878cecb764c0f3ab5838af174b00249a648a2a3): Error starting userland proxy: listen tcp 192.168.122.64:53: bind: address already in use
ERROR: Encountered errors while bringing up the project.
```
boburciu@dns:~/DNS_server$ 
boburciu@dns:~/DNS_server$ ` sudo lsof -t -i:53 `
```
1446
1459
boburciu@dns:~/DNS_server$ ps aux | grep 1446
root      1446  0.0  0.2 405648  4192 ?        Sl   16:49   0:00 /usr/bin/docker-proxy -proto tcp -host-ip 192.168.122.64 -host-port 53 -container-ip 172.17.0.2 -container-port 53
boburciu  2870  0.0  0.0  13220  1036 pts/0    S+   17:04   0:00 grep --color=auto 1446
boburciu@dns:~/DNS_server$ 
boburciu@dns:~/DNS_server$ ps aux | grep 1459
root      1459  0.0  0.2 700576  4436 ?        Sl   16:49   0:00 /usr/bin/docker-proxy -proto udp -host-ip 192.168.122.64 -host-port 53 -container-ip 172.17.0.2 -container-port 53
boburciu  2872  0.0  0.0  13220  1040 pts/0    S+   17:05   0:00 grep --color=auto 1459
```
boburciu@dns:~/DNS_server$ ` sudo kill -9 $(sudo lsof -t -i:53) `
```
boburciu@dns:~/DNS_server$ 
boburciu@dns:~/DNS_server$ netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::9090                 :::*                    LISTEN      -                   
udp        0      0 192.168.122.64:68       0.0.0.0:*                           -                   
boburciu@dns:~/DNS_server$ 
```

 ### -  If _/srv/docker_ file-system is returned as read-only, remove and reinstall Docker:
 
boburciu@dns:~/DNS_server$ ` sudo docker-compose up `
```
Starting 8340069eaf31_dns_server_bind_DNS_server_container_1 ... error

ERROR: for 8340069eaf31_dns_server_bind_DNS_server_container_1  Cannot start service bind_DNS_server_container: error while creating mount source path '/srv/docker/bind': mkdir /srv/docker: read-only file system

ERROR: for bind_DNS_server_container  Cannot start service bind_DNS_server_container: error while creating mount source path '/srv/docker/bind': mkdir /srv/docker: read-only file system
ERROR: Encountered errors while bringing up the project.
boburciu@dns:~/DNS_server$ 
boburciu@dns:~/DNS_server$ sudo docker ps -a
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS                      PORTS                                                                            NAMES
75e9e928a103        rancher/rancher                  "entrypoint.sh"          50 minutes ago      Exited (1) 32 minutes ago                                                                                    keen_jones
89652d1a94d5        4a87daf63151                     "/bin/sh -c 'apt-get…"   7 weeks ago         Exited (100) 7 weeks ago                                                                                     hardcore_ellis
a45caef7d612        4a87daf63151                     "/bin/sh -c 'apt-get…"   7 weeks ago         Exited (100) 7 weeks ago                                                                                     bold_mclaren
b4e3800bbc0f        4a87daf63151                     "/bin/sh -c 'apt-get…"   7 weeks ago         Exited (100) 7 weeks ago                                                                                     peaceful_sammet
75fd92648814        4a87daf63151                     "/bin/sh -c 'apt-get…"   7 weeks ago         Exited (100) 7 weeks ago                                                                                     condescending_turing
8340069eaf31        sameersbn/bind:latest            "/sbin/entrypoint.sh…"   2 months ago        Created                     192.168.122.64:53->53/tcp, 192.168.122.64:53->53/udp, 0.0.0.0:10000->10000/tcp   8340069eaf31_dns_server_bind_DNS_server_container_1
1c43cb02601c        sameersbn/bind:9.16.1-20200524   "/sbin/entrypoint.sh…"   2 months ago        Created                                                                                                      bind_DNS_container
fcaa48ca098f        hello-world                      "/hello"                 2 months ago        Exited (0) 2 months ago                                                                                      sweet_grothendieck
boburciu@dns:~/DNS_server$ sudo docker start 8340069eaf31
Error response from daemon: error while creating mount source path '/srv/docker/bind': mkdir /srv/docker: read-only file system
Error: failed to start containers: 8340069eaf31
boburciu@dns:~/DNS_server$  
```
boburciu@dns:~/DNS_server$ ` sudo apt-get remove docker docker-engine docker.io containerd runc  `  
boburciu@dns:~/DNS_server$ ` curl -fsSL https://get.docker.com -o get-docker.sh  `  <br/>
boburciu@dns:~/DNS_server$ ` sudo sh get-docker.sh  `  <br/>
boburciu@dns:~/DNS_server$ ` sudo docker ps `
```
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                                                            NAMES
c6dc56569d50        sameersbn/bind:latest   "/sbin/entrypoint.sh…"   2 months ago        Up 2 minutes        192.168.122.64:53->53/tcp, 192.168.122.64:53->53/udp, 0.0.0.0:10000->10000/tcp   dns_server_bind_DNS_server_container_1
boburciu@dns:~/DNS_server$ 
```