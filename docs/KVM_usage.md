# Managing KVM machines in CentOS 7.2 
 
## 0. Setup notes:

 ### - virt-manager (a GUI tool), which is a API client of **libvirtd** service (same as `virsh` and `virt-install`) is not synced with `virsh`
` virsh uri ` <br/>

 ### - If it returns qemu:///session, but you're using a qemu:///system connection in Virt-Manager, you found the cause <br/>

 ### - To change your connection URI and use qemu:///system as default is to edit your $HOME/.bahsrc and add <br/>
 ` export LIBVIRT_DEFAULT_URI="qemu:///system" `  <br/>

 ### - virt-manager will start with "QEMU/KVM user session" connection <br/>

## I. How to create KVM:

 ### - Commands for [KVM networking](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-managing_guest_virtual_machines_with_virsh-managing_virtual_networks)

 ### - Commands for [interface statistics](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-statlists)

[boburciu@r220 ~]$ ` sudo virt-install --name=ubuntu1804cmd --ram=1024 --vcpus=1 --cdrom=/home/boburciu/Desktop/ISOs/ubuntu-18.04.5-live-server-amd64.iso --os-type=linux --os-variant=ubuntu18.04 --network default --disk path=/BM_VMs/rkeM1cmd.qcow2,size=20 `<br/>
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` sudo virsh list --all `<br/>
```
 Id    Name                           State
----------------------------------------------------
 1     DNSubuntu18.04                 running
 -     RancherOSm2                    shut off
 -     rke_master1                    shut off
 -     rke_master2                    shut off
 -     rke_worker1                    shut off
 -     rke_worker2                    shut off
 -     rke_worker3                    shut off
 -     rke_worker4                    shut off
 -     ubuntu20.04M1                  shut off
```
[boburciu@r220 ~]$ ` sudo virsh domiflist DNSubuntu18.04 `
```
Interface  Type       Source     Model       MAC<br/>
-------------------------------------------------------
vnet0      network    default    virtio      52:54:00:69:8a:08

[boburciu@r220 ~]$
```

## II. How to create Linux iso for unnatended install:

 ### - Using the project [linux-unattended-installation](https://github.com/coreprocess/linux-unattended-installation)
[boburciu@r220 ~]$ ` ssh boburciu@192.168.122.64 `
```
boburciu@192.168.122.64's password:
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Dec  3 16:11:00 UTC 2020

  System load:  0.35               Processes:              112
  Usage of /:   35.2% of 18.57GB   Users logged in:        0
  Memory usage: 22%                IP address for ens3:    192.168.122.64
  Swap usage:   0%                 IP address for docker0: 172.17.0.1

  => There is 1 zombie process.

 * Introducing self-healing high availability clusters in MicroK8s.
   Simple, hardened, Kubernetes for production, from RaspberryPi to DC.

     https://microk8s.io/high-availability

38 packages can be updated.
0 updates are security updates.

New release '20.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Thu Dec  3 13:04:24 2020 from 192.168.122.1
boburciu@dns:~$
boburciu@dns:~$
```
boburciu@dns:~$ ` mkdir Linux_unattended-install_iso `
boburciu@dns:~$ ` cd Linux_unattended-install_iso/ `
boburciu@dns:~/Linux_unattended-install_iso$ git version
git version 2.17.1
boburciu@dns:~/Linux_unattended-install_iso$
boburciu@dns:~/Linux_unattended-install_iso$ ` git clone https://github.com/coreprocess/linux-unattended-installation.git `
```
boburciu@dns:~/Linux_unattended-install_iso$ ls
linux-unattended-installation
boburciu@dns:~/Linux_unattended-install_iso$ cd linux-unattended-installation/
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ls -lt
total 24
drwxrwxr-x 5 boburciu boburciu 4096 Dec  3 16:12 ubuntu
-rw-rw-r-- 1 boburciu boburciu  323 Dec  3 16:12 Dockerfile
-rw-rw-r-- 1 boburciu boburciu 1073 Dec  3 16:12 LICENSE
-rwxrwxr-x 1 boburciu boburciu  183 Dec  3 16:12 docker-entrypoint.sh
-rw-rw-r-- 1 boburciu boburciu 5536 Dec  3 16:12 readme.md
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
```
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ` sudo apt-get upgrade `
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ` sudo docker build -t ubuntu-unattended . `
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ` sudo docker image ls `
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu-unattended   latest              9908488a91fd        9 minutes ago       222MB
ubuntu              latest              f643c72bc252        7 days ago          72.9MB
sameersbn/bind      latest              55516ab380dc        6 months ago        343MB
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
```

 ### - Create the one-off container that will create the unattended-install iso
```  
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ pwd
/home/boburciu/Linux_unattended-install_iso/linux-unattended-installation
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
```
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ` sudo docker run \ 
 --rm \
 -t \
 -v "$HOME/.ssh/id_rsa.pub:/root/.ssh/id_rsa.pub:ro" \
 -v "$(pwd):/iso" \
 ubuntu-unattended \
 18.04 `
``` 
--2020-12-03 17:08:37--  http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/mini.iso
Resolving archive.ubuntu.com (archive.ubuntu.com)... 91.189.88.142, 91.189.88.152
Connecting to archive.ubuntu.com (archive.ubuntu.com)|91.189.88.142|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 67108864 (64M) [application/x-iso9660-image]
Saving to: './netboot.iso'

./netboot.iso                  100%[====================================================>]  64.00M  30.6MB/s    in 2.1s

2020-12-03 17:08:39 (30.6 MB/s) - './netboot.iso' saved [67108864/67108864]


7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=C,Utf16=off,HugeFiles=on,64 bits,2 CPUs Intel Core Processor (Haswell, IBRS) (306C4),ASM,AES-NI)

Scanning the drive for archives:
1 file, 67108864 bytes (64 MiB)

Extracting archive: ./netboot.iso

WARNINGS:
There are data after the end of archive

--
Path = ./netboot.iso
Type = Iso
WARNINGS:
There are data after the end of archive
Physical Size = 60459008
Tail Size = 6649856
Created = 2018-04-25 20:11:06
Modified = 2018-04-25 20:11:06

Everything is Ok

Archives with Warnings: 1

Warnings: 1
Folders: 4
Files: 278
Size:       62427580
Compressed: 67108864
dos2unix: converting file ./isolinux.cfg to Unix format...
patching file isolinux.cfg
patching file boot/grub/grub.cfg
20 blocks
4 blocks
xorriso 1.5.2 : RockRidge filesystem manipulator, libburnia project.

xorriso : NOTE : Local character set is now assumed as: 'utf-8'
Drive current: -outdev 'stdio:/iso/ubuntu-18.04-netboot-amd64-unattended.iso'
Media current: stdio file, overwriteable
Media status : is blank
Media summary: 0 sessions, 0 data blocks, 0 data, 10.4g free
xorriso : WARNING : -volid text does not comply to ISO 9660 / ECMA 119 rules
Added to ISO image: directory '/'='/tmp/tmp.UexHWYFBwD'
xorriso : UPDATE :     280 files added in 1 seconds
xorriso : UPDATE :     280 files added in 1 seconds
xorriso : NOTE : Copying to System Area: 432 bytes from file '/ubuntu/18.04/custom/isohdpfx.bin'
libisofs: NOTE : Aligned image size to cylinder size by 12 blocks
ISO image produced: 29696 sectors
Written to medium : 29696 sectors at LBA 0
Writing to 'stdio:/iso/ubuntu-18.04-netboot-amd64-unattended.iso' completed successfully.

Next steps: install system, login via root, adjust the authorized keys, set a root password (if you want to), deploy via ansible (if applicable), enjoy!
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ 
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ls -lht
total 59M
-rw-r--r-- 1 root     root      58M Dec  3 17:08 ubuntu-18.04-netboot-amd64-unattended.iso
drwxrwxr-x 5 boburciu boburciu 4.0K Dec  3 16:12 ubuntu
-rw-rw-r-- 1 boburciu boburciu  323 Dec  3 16:12 Dockerfile
-rw-rw-r-- 1 boburciu boburciu 1.1K Dec  3 16:12 LICENSE
-rwxrwxr-x 1 boburciu boburciu  183 Dec  3 16:12 docker-entrypoint.sh
-rw-rw-r-- 1 boburciu boburciu 5.5K Dec  3 16:12 readme.md
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
```
 ### - The ISO /home/boburciu/Linux_unattended-install_iso/linux-unattended-installation/ubuntu-18.04-netboot-amd64-unattended.iso was created on VM running Docker
 ### - Transfer the ISO to KVM CentOs7 host:
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$ ` scp -3 -r ubuntu-18.04-netboot-amd64-unattended.iso boburciu@192.168.100.10:/home/boburciu/Desktop/ISOs/ `
```
The authenticity of host '192.168.100.10 (192.168.100.10)' can't be established.
ECDSA key fingerprint is SHA256:xxgIU8G+wvTR3tebY1EvBpmwdHsoHTyNY2Hgxs+0ADo.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.100.10' (ECDSA) to the list of known hosts.
boburciu@192.168.100.10's password:
ubuntu-18.04-netboot-amd64-unattended.iso                                                 100%   58MB  83.0MB/s   00:00
boburciu@dns:~/Linux_unattended-install_iso/linux-unattended-installation$
```

[boburciu@r220 ~]$
[boburciu@r220 ~]$ ` ls -lth /home/boburciu/Desktop/ISOs/ `
```
total 24G
-rw-r--r--. 1 boburciu boburciu  58M Dec  3 19:17 ubuntu-18.04-netboot-amd64-unattended.iso
-rw-rw-r--. 1 boburciu boburciu 1.7G Nov 21 14:37 lubuntu-20.04.1-desktop-amd64.iso
-rw-rw-r--. 1 qemu     qemu     945M Nov 16 19:49 ubuntu-18.04.5-live-server-amd64.iso
-rw-rw-r--. 1 qemu     qemu     998M Nov 15 19:00 ubuntu-20.10-live-server-amd64.iso
-rw-------. 1 qemu     qemu      21G Nov 14 21:14 RancherOSm2.qcow2
-rw-------. 1 root     root      21G Nov 14 21:13 RancherOSm1.qcow2
[boburciu@r220 ~]$
```