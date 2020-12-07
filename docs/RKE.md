# Managing RKE Kubernetes cluster
 
## 0. Setup RKE:

 ### - Considering [official RKE installation guide](https://rancher.com/docs/rke/latest/en/installation/) and downloading from [GitHub RKE release v1.2.3 with K8s v1.19 support](https://github.com/rancher/rke/releases/tag/v1.2.3) we followed [automation guidelines](https://computingforgeeks.com/install-kubernetes-production-cluster-using-rancher-rke/)

 ### - download the RKE binary to a folder in your $PATH, like */usr/local/bin*, and rename it *rke*. Then verify supported versions (including v1.19 for CKAD as of Dec 2020)) <br/>
`sudo wget https://github.com/rancher/rke/releases/download/v1.2.3/rke_linux-amd64 --directory-prefix=/usr/local/bin` <br/>
`sudo mv /usr/local/bin/rke_linux-amd64 /usr/local/bin/rke` <br/>
`sudo chmod +x /usr/local/bin/rke` <br/>
`rke config --list-version --all` <br/>
```
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ sudo wget https://github.com/rancher/rke/releases/download/v1.2.3/rke_linux-amd64 --directory-prefix=/usr/local/bin
--2020-12-03 13:21:18--  https://github.com/rancher/rke/releases/download/v1.2.3/rke_linux-amd64
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/108337180/810be400-259f-11eb-87a1-3b9532499217?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20201203%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20201203T112111Z&X-Amz-Expires=300&X-Amz-Signature=f6514e8d769f7b56ce4f57441609c1480c3bfc3bf7329e4133ddb6c803b676ca&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=108337180&response-content-disposition=attachment%3B%20filename%3Drke_linux-amd64&response-content-type=application%2Foctet-stream [following]
--2020-12-03 13:21:18--  https://github-production-release-asset-2e65be.s3.amazonaws.com/108337180/810be400-259f-11eb-87a1-3b9532499217?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20201203%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20201203T112111Z&X-Amz-Expires=300&X-Amz-Signature=f6514e8d769f7b56ce4f57441609c1480c3bfc3bf7329e4133ddb6c803b676ca&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=108337180&response-content-disposition=attachment%3B%20filename%3Drke_linux-amd64&response-content-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 52.217.92.100
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|52.217.92.100|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 38811818 (37M) [application/octet-stream]
Saving to: ‘/usr/local/bin/rke_linux-amd64’

100%[==================================================================================>] 38,811,818  15.2MB/s   in 2.4s

2020-12-03 13:21:21 (15.2 MB/s) - ‘/usr/local/bin/rke_linux-amd64’ saved [38811818/38811818]

[boburciu@r220 KVM-notes-proj]$ 
[boburciu@r220 KVM-notes-proj]$ ls -lt  /usr/local/bin
total 49844
-rwxr-xr-x. 1 root root      397 Nov 20 20:42 livereload
-rwxr-xr-x. 1 root root 12218968 Nov 18 22:17 docker-compose
-rw-r--r--. 1 root root 38811818 Nov 13 20:00 rke_linux-amd64
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ sudo mv /usr/local/bin/rke_linux-amd64 /usr/local/bin/rke
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ stat /usr/local/bin/rke
  File: ‘/usr/local/bin/rke’
  Size: 38811818   Blocks: 75808      IO Block: 4096   regular file
Device: fd00h/64768d Inode: 134486326   Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:bin_t:s0
Access: 2020-12-03 13:21:21.000000000 +0200
Modify: 2020-11-13 20:00:57.000000000 +0200
Change: 2020-12-03 13:23:21.689372213 +0200
 Birth: -
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ sudo chmod +x /usr/local/bin/rke
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ stat /usr/local/bin/rke
  File: ‘/usr/local/bin/rke’
  Size: 38811818   Blocks: 75808      IO Block: 4096   regular file
Device: fd00h/64768d Inode: 134486326   Links: 1
Access: (0755/-rwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:bin_t:s0
Access: 2020-12-03 13:21:21.000000000 +0200
Modify: 2020-11-13 20:00:57.000000000 +0200
Change: 2020-12-03 13:23:40.328551891 +0200
 Birth: -
[boburciu@r220 KVM-notes-proj]$ ls -lt  /usr/local/bin
total 49844
-rwxr-xr-x. 1 root root      397 Nov 20 20:42 livereload
-rwxr-xr-x. 1 root root 12218968 Nov 18 22:17 docker-compose
-rwxr-xr-x. 1 root root 38811818 Nov 13 20:00 rke
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/boburciu/.local/bin:/home/boburciu/bin
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ /usr/local/bin/rke --version
rke version v1.2.3
[boburciu@r220 KVM-notes-proj]$ rke --version
rke version v1.2.3
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$
[boburciu@r220 KVM-notes-proj]$ rke config --list-version --all
v1.16.15-rancher1-3
v1.17.14-rancher1-1
v1.18.12-rancher1-1
v1.19.4-rancher1-1
[boburciu@r220 KVM-notes-proj]$
```

## 1. Create the K8s cluster by Rancher K8s Engine (RKE):

 ### - We have the KVMs to form the K8s cluster created and added in Ansible /etc/ansible/hosts:
```
[boburciu@r220 ~]$ ` nslookup rkem1; nslookup rkem2; nslookup rkew1; nslookup rkew2; nslookup rkew3; nslookup rkew4; `
Server:         192.168.122.64
Address:        192.168.122.64#53

Name:   rkem1.boburciu.privatecloud.com
Address: 192.168.122.192

Server:         192.168.122.64
Address:        192.168.122.64#53

Name:   rkem2.boburciu.privatecloud.com
Address: 192.168.122.144

Server:         192.168.122.64
Address:        192.168.122.64#53

Name:   rkew1.boburciu.privatecloud.com
Address: 192.168.122.180

Server:         192.168.122.64
Address:        192.168.122.64#53

Name:   rkew2.boburciu.privatecloud.com
Address: 192.168.122.167

Server:         192.168.122.64
Address:        192.168.122.64#53

Name:   rkew3.boburciu.privatecloud.com
Address: 192.168.122.55

Server:         192.168.122.64
Address:        192.168.122.64#53

Name:   rkew4.boburciu.privatecloud.com
Address: 192.168.122.142

[boburciu@r220 ~]$
```
[boburciu@r220 ~]$ ` head -15 /etc/ansible/hosts `
```
[VMs:children]
rancheos
ubuntu-rke

[rancheros]
192.168.122.155

[ubuntu-rke]
rkem1  ansible_user=root
rkem2  ansible_user=root
rkew1  ansible_user=root
rkew2  ansible_user=root
rkew3  ansible_user=root
rkew4  ansible_user=root

[boburciu@r220 ~]$
```

 ### - First satisfy prerequisites for RKE on the KVMs:
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` cd /home/boburciu/Rancher_K8s_prereq ` <br/>
[boburciu@r220 Rancher_K8s_prereq]$
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible-playbook Rancher_prereq_setup.yml -v `
```
:

PLAY RECAP **********************************************************************************************************************
rkem1                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkem2                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew1                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew2                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew3                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew4                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

[boburciu@r220 Rancher_K8s_prereq]$
```

 ### - Create the RKE config to be used for K8s cluster install, which has these [Kubernetes install options](https://rancher.com/docs/rke/latest/en/config-options/)
 #### - 1. running ` rke config ` in a dir will create the cluster.yml file.
[boburciu@r220 ~]$
[boburciu@r220 ~]$ ` rke config `
```
[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]:
[+] Number of Hosts [1]: 6
[+] SSH Address of host (1) [none]: rkem1
[+] SSH Port of host (1) [22]:
[+] SSH Private Key Path of host (rkem1) [none]:
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (rkem1) [none]:
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
[+] SSH User of host (rkem1) [ubuntu]:
[+] Is host (rkem1) a Control Plane host (y/n)? [y]: y
[+] Is host (rkem1) a Worker host (y/n)? [n]: n
[+] Is host (rkem1) an etcd host (y/n)? [n]: y
[+] Override Hostname of host (rkem1) [none]: rkem1
[+] Internal IP of host (rkem1) [none]: 192.168.122.192
[+] Docker socket path on host (rkem1) [/var/run/docker.sock]:
[+] SSH Address of host (2) [none]: rkem2
[+] SSH Port of host (2) [22]:
[+] SSH Private Key Path of host (rkem2) [none]:
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (rkem2) [none]:
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
[+] SSH User of host (rkem2) [ubuntu]:
[+] Is host (rkem2) a Control Plane host (y/n)? [y]: y
[+] Is host (rkem2) a Worker host (y/n)? [n]: n
[+] Is host (rkem2) an etcd host (y/n)? [n]: y
[+] Override Hostname of host (rkem2) [none]: rkem2
[+] Internal IP of host (rkem2) [none]: 192.168.122.144
[+] Docker socket path on host (rkem2) [/var/run/docker.sock]:
[+] SSH Address of host (3) [none]: rkew1
[+] SSH Port of host (3) [22]:
[+] SSH Private Key Path of host (rkew1) [none]:
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (rkew1) [none]:
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
[+] SSH User of host (rkew1) [ubuntu]:
[+] Is host (rkew1) a Control Plane host (y/n)? [y]: n
[+] Is host (rkew1) a Worker host (y/n)? [n]: y
[+] Is host (rkew1) an etcd host (y/n)? [n]: n
[+] Override Hostname of host (rkew1) [none]: rkew1
[+] Internal IP of host (rkew1) [none]: 192.168.122.180
[+] Docker socket path on host (rkew1) [/var/run/docker.sock]:
:
:
[+] SSH Address of host (6) [none]: rkew4
[+] SSH Port of host (6) [22]:
[+] SSH Private Key Path of host (rkew4) [none]:
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (rkew4) [none]:
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
[+] SSH User of host (rkew4) [ubuntu]:
[+] Is host (rkew4) a Control Plane host (y/n)? [y]: n
[+] Is host (rkew4) a Worker host (y/n)? [n]: y
[+] Is host (rkew4) an etcd host (y/n)? [n]: n
[+] Override Hostname of host (rkew4) [none]: rkew4
[+] Internal IP of host (rkew4) [none]: 192.168.122.142
[+] Docker socket path on host (rkew4) [/var/run/docker.sock]:
[+] Network Plugin Type (flannel, calico, weave, canal) [canal]: calico
[+] Authentication Strategy [x509]:
[+] Authorization Mode (rbac, none) [rbac]:
[+] Kubernetes Docker image [rancher/hyperkube:v1.19.4-rancher1]:
[+] Cluster domain [cluster.local]:
[+] Service Cluster IP Range [10.43.0.0/16]:
[+] Enable PodSecurityPolicy [n]:
[+] Cluster Network CIDR [10.42.0.0/16]:
[+] Cluster DNS Service IP [10.43.0.10]:
[+] Add addon manifest URLs or YAML files [no]:
[boburciu@r220 ~]$
```

 #### - 2. cluster.yml got created
```  
[boburciu@r220 ~]$ date
Sat Dec  5 23:33:27 EET 2020
[boburciu@r220 ~]$ ls -lt
total 12
-rw-r-----. 1 boburciu boburciu 6005 Dec  5 23:31 cluster.yml
drwx------. 1 boburciu boburciu    0 Dec  5 21:11 thinclient_drives
drwxrwxr-x. 3 boburciu boburciu 4096 Dec  3 18:49 Rancher_K8s_prereq
lrwxrwxrwx. 1 boburciu boburciu   21 Nov 25 21:36 qemu-kvm -> /usr/libexec/qemu-kvm
drwxrwxr-x. 2 boburciu boburciu   74 Nov 23 23:29 K8s_cluster_RKE
drwxr-xr-x. 2 boburciu boburciu    6 Nov 21 14:39 Downloads
drwxr-xr-x. 4 boburciu boburciu   58 Nov 21 14:23 Desktop
drwxrwxr-x. 3 boburciu boburciu   27 Nov 20 23:15 Work_documenting
drwxr-xr-x. 2 boburciu boburciu    6 Nov 14 13:29 Documents
drwxr-xr-x. 2 boburciu boburciu    6 Nov 14 13:29 Music
drwxr-xr-x. 2 boburciu boburciu    6 Nov 14 13:29 Pictures
drwxr-xr-x. 2 boburciu boburciu    6 Nov 14 13:29 Public
drwxr-xr-x. 2 boburciu boburciu    6 Nov 14 13:29 Templates
drwxr-xr-x. 2 boburciu boburciu    6 Nov 14 13:29 Videos
[boburciu@r220 ~]$
[boburciu@r220 ~]$
[boburciu@r220 ~]$ mv cluster.yml K8s_cluster_RKE/
[boburciu@r220 ~]$ ls -lt K8s_cluster_RKE/
total 12
-rw-r-----. 1 boburciu boburciu 6005 Dec  5 23:31 cluster.yml
-rw-rw-r--. 1 boburciu boburciu  805 Nov 23 23:18 external_vars.yml
-rw-rw-r--. 1 boburciu boburciu    0 Nov 23 23:14 install_kubectl.yml
[boburciu@r220 ~]$
[boburciu@r220 ~]$ cd K8s_cluster_RKE/
[boburciu@r220 K8s_cluster_RKE]$ cat cluster.yml | grep system_images
system_images:
[boburciu@r220 K8s_cluster_RKE]$ cat cluster.yml | grep kubernetes_version:
kubernetes_version: ""
[boburciu@r220 K8s_cluster_RKE]$ cat cluster.yml | grep 1.19
  kubernetes: rancher/hyperkube:v1.19.4-rancher1
[boburciu@r220 K8s_cluster_RKE]$
```
 ### - Run ` rke up ` in the dir with cluster.yml
```
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$ rke up
INFO[0000] Running RKE version: v1.2.3
INFO[0000] Initiating Kubernetes cluster
INFO[0000] [certificates] GenerateServingCertificate is disabled, checking if there are unused kubelet certificates
INFO[0000] [certificates] Generating Kubernetes API server certificates
INFO[0000] [certificates] Generating admin certificates and kubeconfig
INFO[0000] [certificates] Generating kube-etcd-192-168-122-192 certificate and key
INFO[0000] [certificates] Generating kube-etcd-192-168-122-144 certificate and key
INFO[0000] Successfully Deployed state file at [./cluster.rkestate]
INFO[0000] Building Kubernetes cluster
INFO[0000] [dialer] Setup tunnel for host [rkew1]
INFO[0000] [dialer] Setup tunnel for host [rkew3]
INFO[0000] [dialer] Setup tunnel for host [rkew2]
INFO[0000] [dialer] Setup tunnel for host [rkem1]
INFO[0000] [dialer] Setup tunnel for host [rkem2]
INFO[0000] [dialer] Setup tunnel for host [rkew4]
INFO[0000] [network] Deploying port listener containers
INFO[0000] Pulling image [rancher/rke-tools:v0.1.66] on host [rkem2], try #1
INFO[0000] Pulling image [rancher/rke-tools:v0.1.66] on host [rkem1], try #1
INFO[0004] Pulling image [rancher/rke-tools:v0.1.66] on host [rkem2], try #1
INFO[0004] Pulling image [rancher/rke-tools:v0.1.66] on host [rkem1], try #1
INFO[0011] Pulling image [rancher/rke-tools:v0.1.66] on host [rkem2], try #1
INFO[0014] Pulling image [rancher/rke-tools:v0.1.66] on host [rkem1], try #1
WARN[0017] Failed to create Docker container [rke-etcd-port-listener] on host [rkem2]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66
WARN[0017] Failed to create Docker container [rke-etcd-port-listener] on host [rkem2]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66
WARN[0017] Failed to create Docker container [rke-etcd-port-listener] on host [rkem2]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66
WARN[0018] Failed to create Docker container [rke-etcd-port-listener] on host [rkem1]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66
WARN[0018] Failed to create Docker container [rke-etcd-port-listener] on host [rkem1]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66
WARN[0018] Failed to create Docker container [rke-etcd-port-listener] on host [rkem1]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66
FATA[0018] [Failed to create [rke-etcd-port-listener] container on host [rkem2]: Failed to create Docker container [rke-etcd-port-listener] on host [rkem2]: Error response from daemon: No such image: rancher/rke-tools:v0.1.66]
[boburciu@r220 K8s_cluster_RKE]$
``` 
 #### - In case error _Failed to create Docker container [rke-etcd-port-listener] on host_ is returned you need to restart Docker service on KVMs:
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible ubuntu-rke -m command -a "sudo service docker restart" `
 #### - To verify Docker restart was done:
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible ubuntu-rke -m command -a "systemctl status docker.service" ` 

 #### - RKE runs succesfully if ended with:
``` 
INFO[0217] [remove/rke-log-cleaner] Successfully removed container on host [rkew3]
INFO[0217] [remove/rke-log-cleaner] Successfully removed container on host [rkew4]
INFO[0217] [remove/rke-log-cleaner] Successfully removed container on host [rkem1]
INFO[0217] [remove/rke-log-cleaner] Successfully removed container on host [rkew2]
INFO[0217] [remove/rke-log-cleaner] Successfully removed container on host [rkem2]
INFO[0217] [remove/rke-log-cleaner] Successfully removed container on host [rkew1]
INFO[0217] [sync] Syncing nodes Labels and Taints
INFO[0217] [sync] Successfully synced nodes Labels and Taints
INFO[0217] [network] Setting up network plugin: calico
INFO[0217] [addons] Saving ConfigMap for addon rke-network-plugin to Kubernetes
INFO[0217] [addons] Successfully saved ConfigMap for addon rke-network-plugin to Kubernetes
INFO[0217] [addons] Executing deploy job rke-network-plugin
INFO[0233] [addons] Setting up coredns
INFO[0233] [addons] Saving ConfigMap for addon rke-coredns-addon to Kubernetes
INFO[0233] [addons] Successfully saved ConfigMap for addon rke-coredns-addon to Kubernetes
INFO[0233] [addons] Executing deploy job rke-coredns-addon
INFO[0243] [addons] CoreDNS deployed successfully
INFO[0243] [dns] DNS provider coredns deployed successfully
INFO[0243] [addons] Setting up Metrics Server
INFO[0243] [addons] Saving ConfigMap for addon rke-metrics-addon to Kubernetes
INFO[0243] [addons] Successfully saved ConfigMap for addon rke-metrics-addon to Kubernetes
INFO[0243] [addons] Executing deploy job rke-metrics-addon
INFO[0248] [addons] Metrics Server deployed successfully
INFO[0248] [ingress] Setting up nginx ingress controller
INFO[0248] [addons] Saving ConfigMap for addon rke-ingress-controller to Kubernetes
INFO[0248] [addons] Successfully saved ConfigMap for addon rke-ingress-controller to Kubernetes
INFO[0248] [addons] Executing deploy job rke-ingress-controller
INFO[0253] [ingress] ingress controller nginx deployed successfully
INFO[0253] [addons] Setting up user addons
INFO[0253] [addons] no user addons defined
INFO[0253] Finished building Kubernetes cluster successfully
[boburciu@r220 K8s_cluster_RKE]$
``` 

## 2. Manage RKE K8s cluster

 ### - I. You need the _kubectl_ command line tool to manage the RKE cluster:
 #### - Download the latest release, make the kubectl binary executable and move it in a dir to your path in to your PATH::
` curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" ` <br/>
` chmod +x ./kubectl ` <br/>
` sudo mv ./kubectl /usr/local/bin/kubectl ` <br/>
` kubectl version --client ` <br/>
``` 
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 41.0M  100 41.0M    0     0  26.1M      0  0:00:01  0:00:01 --:--:-- 26.1M
[boburciu@r220 K8s_cluster_RKE]$ chmod +x ./kubectl
[boburciu@r220 K8s_cluster_RKE]$ sudo mv ./kubectl /usr/local/bin/kubectl
[sudo] password for boburciu:
[boburciu@r220 K8s_cluster_RKE]$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:17:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
[boburciu@r220 K8s_cluster_RKE]$
``` 

 #### - Or to have *kubectl* on the RKE master node 1, SSH to it and install
[boburciu@r220 OpenStackHelm_prereq]$ ` ssh ubuntu@rkem1 `
``` 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
New release '20.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Mon Dec  7 15:05:27 2020 from 192.168.122.1
ubuntu@device:~$
ubuntu@device:~$ pwd
/home/ubuntu
ubuntu@device:~$
ubuntu@device:~$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
> ^C
ubuntu@device:~$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 41.0M  100 41.0M    0     0  26.8M      0  0:00:01  0:00:01 --:--:-- 26.8M
ubuntu@device:~$ chmod +x ./kubectl
ubuntu@device:~$ sudo mv ./kubectl /usr/local/bin/kubectl
ubuntu@device:~$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:17:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
ubuntu@device:~$
``` 

 #### - then create ~/.kube/ dir, the default location for KUBECONFIG

ubuntu@device:~$ ` mkdir ~/.kube `
``` 
ubuntu@device:~$ 
ubuntu@device:~$
ubuntu@device:~$ stat ~/.kube
  File: /home/ubuntu/.kube
  Size: 4096            Blocks: 8          IO Block: 4096   directory
Device: fd00h/64768d    Inode: 663812      Links: 2
Access: (0775/drwxrwxr-x)  Uid: ( 1000/  ubuntu)   Gid: ( 1000/  ubuntu)
Access: 2020-12-07 20:49:45.614488346 +0000
Modify: 2020-12-07 20:49:37.654408911 +0000
Change: 2020-12-07 20:49:37.654408911 +0000
 Birth: -
ubuntu@device:~$
``` 

 ### - II. RKE K8s cluster install has created _kube_config_cluster.yml_ in the dir where cluster.yml was called, which can be use to connect the _kubectl_ utility with the cluster:
```
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$ pwd
/home/boburciu/K8s_cluster_RKE
[boburciu@r220 K8s_cluster_RKE]$ ls -lt
total 128
-rw-r-----. 1 boburciu boburciu 114303 Dec  6 00:41 cluster.rkestate
-rw-r-----. 1 boburciu boburciu   5376 Dec  6 00:38 kube_config_cluster.yml
-rw-r-----. 1 boburciu boburciu   6005 Dec  5 23:31 cluster.yml
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$ cat kube_config_cluster.yml
apiVersion: v1
kind: Config
clusters:
- cluster:
    api-version: v1
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN3akNDQWFxZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFTTVJBd0RnWURWUVFERXdkcmRXSmwKTFdOaE1CNFhEVEl3TVR:
    UkVUc01TQUhXTTJFa3UKYjg4THcwNWVRaXBtemJjb0NCYkFxdnZkMkIwaWJJNFFyRVk5SEtZb2xaMjh6L1lPS2EyRThOY25tUnI0M25PSgpqTlhrMGdzd2prQ1BxQ3phOTRENnpJUUhNangzZTFYTWRxMUcyU3pYY1lIRDJkRHBvS2s9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: "https://rkem2:6443"
  name: "local"
contexts:
- context:
    cluster: "local"
    user: "kube-admin-local"
  name: "local"
current-context: "local"
users:
- name: "kube-admin-local"
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM2VENDQWRHZ0F3SUJBZ0lJYWIrUkpyTUZCdzR3RFFZSktvWklodmNOQVFFTEJRQXdFakVRTUE0R0ExVUUKQXhNSGEzVmlaUzFqWVR:
    09rK2E0UEUrTGNzYkRMdkN0ZDlTLzlhYm9jQ1JBNm1EMmc9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
[boburciu@r220 K8s_cluster_RKE]$
```
[boburciu@r220 K8s_cluster_RKE]$ ` export KUBECONFIG=/home/boburciu/K8s_cluster_RKE/kube_config_cluster.yml `
```
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$ kubectl get nodes
NAME    STATUS   ROLES               AGE   VERSION
rkem1   Ready    controlplane,etcd   36m   v1.19.4
rkem2   Ready    controlplane,etcd   36m   v1.19.4
rkew1   Ready    worker              36m   v1.19.4
rkew2   Ready    worker              36m   v1.19.4
rkew3   Ready    worker              36m   v1.19.4
rkew4   Ready    worker              36m   v1.19.4
[boburciu@r220 K8s_cluster_RKE]$
```

 #### - Or to have *kubectl* on the RKE master node 1, transfer */home/boburciu/K8s_cluster_RKE/kube_config_cluster.yml* to **rkem1** as ~/.kube/config
[boburciu@r220 K8s_cluster_RKE]$ ` scp -3 -r ./kube_config_cluster.yml ubuntu@rkem1:~/.kube/config `
```
kube_config_cluster.yml                                                                     100% 5376    11.5MB/s   00:00
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$

ubuntu@device:~$ ls -lt ~/.kube
total 8
-rw-r----- 1 ubuntu ubuntu 5376 Dec  7 20:53 config
ubuntu@device:~$
ubuntu@device:~$ date
Mon Dec  7 20:53:34 UTC 2020
ubuntu@device:~$
ubuntu@device:~$
ubuntu@device:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:17:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
ubuntu@device:~$ kubectl get nodes
NAME    STATUS   ROLES               AGE   VERSION
rkem1   Ready    controlplane,etcd   46h   v1.19.4
rkem2   Ready    controlplane,etcd   46h   v1.19.4
rkew1   Ready    worker              46h   v1.19.4
rkew2   Ready    worker              46h   v1.19.4
rkew3   Ready    worker              46h   v1.19.4
rkew4   Ready    worker              46h   v1.19.4
ubuntu@device:~$
```