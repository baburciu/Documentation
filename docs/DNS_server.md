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