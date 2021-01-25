# Managing manual RKE Kubernetes cluster creation
 
## 0. Setup RKE:

 ### * Considering [official RKE installation guide](https://rancher.com/docs/rke/latest/en/installation/) and downloading from [GitHub RKE release v1.2.3 with K8s v1.19 support](https://github.com/rancher/rke/releases/tag/v1.2.3) we followed [automation guidelines](https://computingforgeeks.com/install-kubernetes-production-cluster-using-rancher-rke/)

 ### * download the RKE binary to a folder in your $PATH, like */usr/local/bin*, and rename it *rke*. Then verify supported versions (including v1.19 for CKAD as of Dec 2020)) <br/>
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

2020-12-03 13:21:21 (15.2 MB/s) * ‘/usr/local/bin/rke_linux-amd64’ saved [38811818/38811818]

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

 ### * We have the KVMs to form the K8s cluster created and added in Ansible /etc/ansible/hosts:

[boburciu@r220 ~]$ ` nslookup rkem1; nslookup rkem2; nslookup rkew1; nslookup rkew2; nslookup rkew3; nslookup rkew4; `
```
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

 ### * First satisfy prerequisites for RKE on the KVMs:
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` cd /home/boburciu/Rancher_K8s_prereq ` <br/>
[boburciu@r220 Rancher_K8s_prereq]$ <br/>
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

 ### * Create the RKE config to be used for K8s cluster install, which has these [Kubernetes install options](https://rancher.com/docs/rke/latest/en/config-options/)
 #### * 1. running ` rke config ` in a dir will create the cluster.yml file.
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

 #### * 2. cluster.yml got created
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
 ### * Run ` rke up ` in the dir with cluster.yml
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
 #### * In case error _Failed to create Docker container [rke-etcd-port-listener] on host_ is returned you need to restart Docker service on KVMs:
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible ubuntu-rke -m command -a "sudo service docker restart" `
 #### * To verify Docker restart was done:
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible ubuntu-rke -m command -a "systemctl status docker.service" ` 

 #### * RKE runs succesfully if ended with:
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

 ### * I. You need the _kubectl_ command line tool to manage the RKE cluster:
 #### * Download the latest release, make the kubectl binary executable and move it in a dir to your path in to your PATH::
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

 #### * Or to have *kubectl* on the RKE master node 1, SSH to it and install
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

 #### * then create ~/.kube/ dir, the default location for KUBECONFIG

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

 ### * II. RKE K8s cluster install has created _kube_config_cluster.yml_ in the dir where cluster.yml was called, which can be use to connect the _kubectl_ utility with the cluster:
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

 #### * Or to have *kubectl* on the RKE master node 1, transfer */home/boburciu/K8s_cluster_RKE/kube_config_cluster.yml* to **rkem1** as ~/.kube/config
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

## 3. Adding RKE K8s cluster nodes:

 ### * I. First create the new KVMs to serve as K8s nodes:
  #### * - RKE Worker Node 5. 
` sudo virt-install --name=rkew5 --ram=4096 --vcpus=4 --cdrom=/home/boburciu/Desktop/ISOs/ubuntu-18.04-netboot-amd64-unattended.iso --os-type=linux --os-variant=ubuntu18.04 --network default --disk path=/BM_VMs/rkew5.qcow2,size=20 ` 

  #### * - RKE Worker Node 6. 
` sudo virt-install --name=rkew6 --ram=4096 --vcpus=4 --cdrom=/home/boburciu/Desktop/ISOs/ubuntu-18.04-netboot-amd64-unattended.iso --os-type=linux --os-variant=ubuntu18.04 --network default --disk path=/BM_VMs/rkew6.qcow2,size=20 ` 

 ### * II. Add the new KVM nodes under _/etc/ansible/hosts_ to run RKE prerequisites Ansible playbook
[boburciu@r220 ~]$ ` head -22 /etc/ansible/hosts `
```
[VMs:children]
rancheos
ubuntu-rke

[rancheros]
192.168.122.155

[ubuntu-rke:children]
ubuntu-rke-masters
ubuntu-rke-workers

[ubuntu-rke-masters]
rkem1  ansible_user=root
rkem2  ansible_user=root

[ubuntu-rke-workers]
rkew1  ansible_user=root
rkew2  ansible_user=root
rkew3  ansible_user=root
rkew4  ansible_user=root
rkew5  ansible_user=root
rkew6  ansible_user=root
[boburciu@r220 ~]$
```
 ### * III. Run RKE prereq playbook for _rkew5_ & _rkew6_
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` cd /home/boburciu/Rancher_K8s_prereq ` <br/>
[boburciu@r220 Rancher_K8s_prereq]$
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible-playbook Rancher_prereq_setup.yml -v `
```
PLAY RECAP *******************************************************************************************************************
rkem1                      : ok=26   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkem2                      : ok=26   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew1                      : ok=26   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew2                      : ok=26   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew3                      : ok=26   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew4                      : ok=26   changed=7    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew5                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew6                      : ok=27   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

[boburciu@r220 Rancher_K8s_prereq]$
```

 ### * IV. Considering [official RKE guide](https://rancher.com/docs/rke/latest/en/managing-clusters/)
 #### * update the original */home/boburciu/K8s_cluster_RKE/cluster.yml* file with any additional nodes and specify their role in the Kubernetes cluster. 
```
 * address: rkew5
  port: "22"
  internal_address: 192.168.122.141
  role:
  * worker
  hostname_override: rkew5
  user: ubuntu
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
 * address: rkew6
  port: "22"
  internal_address: 192.168.122.88
  role:
  * worker
  hostname_override: rkew6
  user: ubuntu
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
```

[boburciu@r220 ~]$ ` sudo vi /home/boburciu/K8s_cluster_RKE/cluster.yml `
```
[sudo] password for boburciu:
[boburciu@r220 ~]$
[boburciu@r220 ~]$ cat /home/boburciu/K8s_cluster_RKE/cluster.yml | grep service -B28
- address: rkew5
  port: "22"
  internal_address: 192.168.122.141
  role:
  * worker
  hostname_override: rkew5
  user: ubuntu
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
- address: rkew6
  port: "22"
  internal_address: 192.168.122.88
  role:
  * worker
  hostname_override: rkew6
  user: ubuntu
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  ssh_cert: ""
  ssh_cert_path: ""
  labels: {}
  taints: []
services:
```

 #### * Run ` rke up --update-only ` and this will ignore everything else in the *cluster.yml* except for any worker nodes
[boburciu@r220 ~]$ ` cd ~/K8s_cluster_RKE/ ` <br/>
[boburciu@r220 K8s_cluster_RKE]$ <br/>
[boburciu@r220 K8s_cluster_RKE]$ ` rke up --update-only `
```
INFO[0000] Running RKE version: v1.2.3
INFO[0000] Initiating Kubernetes cluster
:
INFO[0173] Finished building Kubernetes cluster successfully
[boburciu@r220 K8s_cluster_RKE]$


ubuntu@device:~$ kubectl get nodes
NAME    STATUS   ROLES               AGE     VERSION
rkem1   Ready    controlplane,etcd   3d20h   v1.19.4
rkem2   Ready    controlplane,etcd   3d20h   v1.19.4
rkew1   Ready    worker              3d20h   v1.19.4
rkew2   Ready    worker              3d20h   v1.19.4
rkew3   Ready    worker              3d20h   v1.19.4
rkew4   Ready    worker              3d20h   v1.19.4
ubuntu@device:~$ kubectl get nodes
NAME    STATUS   ROLES               AGE     VERSION
rkem1   Ready    controlplane,etcd   3d20h   v1.19.4
rkem2   Ready    controlplane,etcd   3d20h   v1.19.4
rkew1   Ready    worker              3d20h   v1.19.4
rkew2   Ready    worker              3d20h   v1.19.4
rkew3   Ready    worker              3d20h   v1.19.4
rkew4   Ready    worker              3d20h   v1.19.4
rkew5   Ready    worker              11m     v1.19.4
rkew6   Ready    worker              11m     v1.19.4
ubuntu@device:~$
ubuntu@device:~$ kubectl get pods --all-namespaces
NAMESPACE       NAME                                      READY   STATUS         RESTARTS   AGE
ingress-nginx   default-http-backend-65dd5949d9-csjv7     1/1     Running        2          2d4h
ingress-nginx   default-http-backend-65dd5949d9-djs9t     0/1     NodeAffinity   0          3d20h
ingress-nginx   nginx-ingress-controller-b9gcr            1/1     Running        3          3d20h
ingress-nginx   nginx-ingress-controller-cdghc            1/1     Running        0          10m
ingress-nginx   nginx-ingress-controller-lftcm            1/1     Running        0          10m
ingress-nginx   nginx-ingress-controller-np27v            1/1     Running        3          3d20h
ingress-nginx   nginx-ingress-controller-pf6hc            1/1     Running        3          3d20h
ingress-nginx   nginx-ingress-controller-zkvgj            1/1     Running        3          3d20h
kube-system     calico-kube-controllers-87d89ff98-df25p   1/1     Running        3          3d20h
kube-system     calico-node-2b4f8                         1/1     Running        3          3d20h
kube-system     calico-node-8dxsj                         1/1     Running        0          11m
kube-system     calico-node-9zzm2                         1/1     Running        3          3d20h
kube-system     calico-node-b8wh5                         1/1     Running        3          3d20h
kube-system     calico-node-nw6tp                         1/1     Running        3          3d20h
kube-system     calico-node-qxqlq                         1/1     Running        5          3d20h
kube-system     calico-node-sth2x                         1/1     Running        0          11m
kube-system     calico-node-zwvq6                         1/1     Running        3          3d20h
kube-system     coredns-6f85d5fb88-fwdgr                  1/1     Running        3          3d20h
kube-system     coredns-6f85d5fb88-t7nfs                  1/1     Running        3          3d20h
kube-system     coredns-autoscaler-79599b9dc6-6phrb       1/1     Running        3          3d20h
kube-system     ingress-error-pages-667b646495-6llsm      0/1     Pending        0          116m
kube-system     ingress-error-pages-667b646495-fgbh7      0/1     Pending        0          116m
kube-system     metrics-server-8449844bf-2zpxm            1/1     Running        2          2d4h
kube-system     metrics-server-8449844bf-8smnh            0/1     NodeAffinity   0          3d20h
kube-system     rke-coredns-addon-deploy-job-g4jff        0/1     Completed      0          3d20h
kube-system     rke-ingress-controller-deploy-job-5h47x   0/1     Completed      0          3d20h
kube-system     rke-metrics-addon-deploy-job-2zlml        0/1     Completed      0          3d20h
kube-system     rke-network-plugin-deploy-job-vnh4w       0/1     Completed      0          3d20h
kube-system     tiller-deploy-7b56c8dfb7-7jsjl            1/1     Running        1          117m
ubuntu@device:~$
```

## 4. In order to remove the Kubernetes components from nodes, you use the `rke remove` command (and if you have other workloads/pods in cluster, you can delete their namespaces to delete them).
```
[boburciu@r220 ~]$
[boburciu@r220 ~]$ cd K8s_cluster_RKE
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$ rke version
INFO[0000] Running RKE version: v1.2.3
Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:09:17Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
[boburciu@r220 K8s_cluster_RKE]$
[boburciu@r220 K8s_cluster_RKE]$
```
[boburciu@r220 K8s_cluster_RKE]$ ` rke remove `
```
INFO[0000] Running RKE version: v1.2.3
Are you sure you want to remove Kubernetes cluster [y/n]: y
INFO[0005] Tearing down Kubernetes cluster
INFO[0005] [dialer] Setup tunnel for host [rkew1]
INFO[0005] [dialer] Setup tunnel for host [rkew5]
INFO[0005] [dialer] Setup tunnel for host [rkew4]
INFO[0005] [dialer] Setup tunnel for host [rkem2]
INFO[0005] [dialer] Setup tunnel for host [rkew2]
INFO[0005] [dialer] Setup tunnel for host [rkew3]
INFO[0005] [dialer] Setup tunnel for host [rkem1]
INFO[0005] [dialer] Setup tunnel for host [rkew6]
:
INFO[0022] Removing local admin Kubeconfig: ./kube_config_cluster.yml
INFO[0022] Local admin Kubeconfig removed successfully
INFO[0022] Removing state file: ./cluster.rkestate
INFO[0022] State file removed successfully
INFO[0022] Cluster removed successfully
[boburciu@r220 K8s_cluster_RKE]$
```
## 5. Clearing out leftovers of an RKE cluster (deleted with either __rke remove__ or via Rancher server UI), following [official guide](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/cleaning-cluster-nodes/)

[boburciu@r220 Rancher_K8s_prereq]$ ` ansible-playbook Rancher_leftovers_remove.yml -v `
```
PLAY RECAP *******************************************************************************************************************
rkem1                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=5
rkem2                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=5
rkew1                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
rkew2                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
rkew3                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
rkew4                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
rkew5                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
rkew6                      : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2

[boburciu@r220 Rancher_K8s_prereq]$
[boburciu@r220 Rancher_K8s_prereq]$ cat Rancher_leftovers_remove.yml
---
- name: Clear leftover containers and dirs after an RKE cluster delete (with either __rke remove__ or via Rancher server UI)
    # per https://rancher.com/docs/rancher/v2.x/en/cluster-admin/cleaning-cluster-nodes/
  hosts: ubuntu-rke
  tasks:
    * name: stop all docker containers
      shell: docker stop $(docker ps -a -q)
      ignore_errors: true
    * name: remove all docker containers
      shell: docker rm -v $(docker ps -a -q)
      ignore_errors: true
    * name: remove all docker volumes
      shell: docker volume rm $(docker volume ls -q)
      ignore_errors: true
    * name: remove all docker images
      shell: docker rmi $(docker images -q)
      ignore_errors: true

    * name: clean Kubernetes components and secrets mounts
      shell: for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do sudo umount $mount; done
      ignore_errors: true

    * name: clean directories
      shell: sudo rm -rf /etc/ceph \ /etc/cni \ /etc/kubernetes \ /opt/cni \ /opt/rke \ /run/secrets/kubernetes.io \ /run/calico \ /run/flannel \ /var/lib/calico \ /var/lib/etcd \ /var/lib/cni \ /var/lib/kubelet \ /var/lib/rancher/rke/log \ /var/log/containers \ /var/log/kube-audit \ /var/log/pods \ /var/run/calico
      ignore_errors: true

[boburciu@r220 Rancher_K8s_prereq]$
```
  ### To remove leftovers to one specific node (a drained one from Rancher server UI):
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible-playbook Rancher_leftovers_remove.yml -v -i /etc/ansible/hosts --limit rkem2 `


  ### The above playbook takes care of:  
  #### * Check what Docker containers are still running on former RKE cluster members: 
[boburciu@r220 K8s_cluster_RKE]$ ` ansible ubuntu-rke -m command -a "sudo docker ps" `
```[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo
rkew3 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
9023076b68e2        rancher/rancher-agent:v2.5.5         "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           musing_almeida
20428899d501        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kube-proxy
760c172b5f30        rancher/rke-tools:v0.1.68            "nginx-proxy CP_HOST…"   20 hours ago        Up 35 minutes                           nginx-proxy
895ddd23e2fc        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kubelet
rkem2 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkew1 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
89c01436a712        rancher/rancher-agent:v2.5.5         "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           dazzling_elbakyan
f0ff67e5f4e7        rancher/rke-tools:v0.1.68            "nginx-proxy CP_HOST…"   20 hours ago        Up 35 minutes                           nginx-proxy
e14377e84177        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kubelet
3bc9fd181c11        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kube-proxy
rkew2 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
73fd1fb96b45        rancher/rancher-agent:v2.5.5         "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           dreamy_herschel
c6b49645b050        rancher/rke-tools:v0.1.68            "nginx-proxy CP_HOST…"   20 hours ago        Up 35 minutes                           nginx-proxy
160e08f81a19        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kubelet
84b678ee68b2        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kube-proxy
rkem1 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS               NAMES
e6a0459b48e7        rancher/rancher-agent:v2.5.5   "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           hopeful_ptolemy
rkew4 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
7ba95379b768        rancher/rancher-agent:v2.5.5         "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           charming_bell
153e8e7daa14        rancher/rke-tools:v0.1.68            "nginx-proxy CP_HOST…"   20 hours ago        Up 35 minutes                           nginx-proxy
1bc49c67e5c9        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 36 minutes                           kubelet
2a9ae646190c        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 36 minutes                           kube-proxy
rkew5 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
6e609854d026        rancher/rancher-agent:v2.5.5         "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           nostalgic_poincare
833e7b69d013        rancher/rke-tools:v0.1.68            "nginx-proxy CP_HOST…"   20 hours ago        Up 36 minutes                           nginx-proxy
c30d1a8a3221        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 36 minutes                           kubelet
41e50aa0a9b6        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 36 minutes                           kube-proxy
rkew6 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
0b789adf8896        rancher/rancher-agent:v2.5.5         "run.sh --server htt…"   35 minutes ago      Up 35 minutes                           practical_shamir
4d223fc060db        rancher/rke-tools:v0.1.68            "nginx-proxy CP_HOST…"   20 hours ago        Up 35 minutes                           nginx-proxy
0dd351977919        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kubelet
0f8d2cefb9b7        rancher/hyperkube:v1.19.6-rancher1   "/opt/rke-tools/entr…"   20 hours ago        Up 35 minutes                           kube-proxy
[boburciu@r220 K8s_cluster_RKE]$
```
  #### * Then SSH to each former RKE member and stop active containers and then remove all containers, then check if nothing remained:
```  
 1015  ssh ubuntu@rkem1
 1016  ssh ubuntu@rkem2
 1017  ssh ubuntu@rkew1
 1018  ssh ubuntu@rkew2
 1019  ssh ubuntu@rkew3
 1020  ssh ubuntu@rkew4
 1021  ssh ubuntu@rkew5
 1022  ssh ubuntu@rkew6
 1023  history
[boburciu@r220 ~]$
[boburciu@r220 ~]$ ssh ubuntu@rkew6

Last login: Fri Dec 11 20:04:05 2020 from 192.168.122.1
ubuntu@device:~$
```
ubuntu@device:~$  ` sudo docker stop $(sudo docker ps -aq) ` <br/>

ubuntu@device:~$  ` sudo docker rm -v $(sudo docker ps -aq) ` <br/>

ubuntu@device:~$  ` for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do sudo umount $mount; done ` <br/>

[boburciu@r220 K8s_cluster_RKE]$ ` ansible ubuntu-rke -m command -a "sudo docker ps" ` 

  #### * Check all Docker volumes on former RKE members:
[boburciu@r220 K8s_cluster_RKE]$ ` ansible ubuntu-rke -m command -a "docker volume ls -q" `
```
[WARNING]: I..
rkem2 | CHANGED | rc=0 >>

rkew1 | CHANGED | rc=0 >>

rkew3 | CHANGED | rc=0 >>

rkew2 | CHANGED | rc=0 >>

rkem1 | CHANGED | rc=0 >>

rkew4 | CHANGED | rc=0 >>

rkew6 | CHANGED | rc=0 >>

rkew5 | CHANGED | rc=0 >>

[boburciu@r220 K8s_cluster_RKE]$
```

  #### * Remove the used directories (important for removing certificates used initially, as those will create problem at new RKE install on same node):
[boburciu@r220 K8s_cluster_RKE]$ ` ansible ubuntu-rke -m command -a "sudo rm -rf /etc/ceph \
      /etc/cni \
      /etc/kubernetes \
      /opt/cni \
      /opt/rke \
      /run/secrets/kubernetes.io \
      /run/calico \
      /run/flannel \
      /var/lib/calico \
      /var/lib/etcd \
      /var/lib/cni \
      /var/lib/kubelet \
      /var/lib/rancher/rke/log \
      /var/log/containers \
      /var/log/kube-audit \
      /var/log/pods \
      /var/run/calico " `
```      
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo
rkem2 | CHANGED | rc=0 >>

rkew2 | CHANGED | rc=0 >>

rkem1 | CHANGED | rc=0 >>

rkew1 | CHANGED | rc=0 >>

rkew3 | CHANGED | rc=0 >>

rkew4 | CHANGED | rc=0 >>

rkew5 | CHANGED | rc=0 >>

rkew6 | CHANGED | rc=0 >>

[boburciu@r220 Rancher_K8s_prereq]$
```

# Managing automatic RKE Kubernetes cluster creation via Rancher server

## 0. Intall Rancher Server as container on VM:
ubuntu@device:~$ ` ssh boburciu@DNSServer `
```
boburciu@dnsserver's password: password
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-128-generic x86_64)

boburciu@dns:~$ 
boburciu@dns:~$ 
boburciu@dns:~$ 
```
boburciu@dns:~$ ` sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher `
```
[sudo] password for boburciu: 
Unable to find image 'rancher/rancher:latest' locally
latest: Pulling from rancher/rancher
:
:
2e9b10fe3352: Pull complete 
Digest: sha256:961980e4d64e2c9b4c6830f61b0a75b6b86695516303a2fc5e053d642e91e958
Status: Downloaded newer image for rancher/rancher:latest
75e9e928a10318c096dbbb668d5a99eab9420939af770ddd797321554c75fdc4
boburciu@dns:~$ 
boburciu@dns:~$ 
boburciu@dns:~$ boburciu@dns:~$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                      NAMES
75e9e928a103        rancher/rancher     "entrypoint.sh"     8 minutes ago       Up 8 minutes        0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   keen_jones
boburciu@dns:~$  
```
  ### Connect to https://192.168.122.64 or https://RancherServer.boburciu.privatecloud.com, from inside KVM network (not from Win), with user _admin_ & pass _password_

## 1. Apply prerequisites for nodes:
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` cd /home/boburciu/Rancher_K8s_prereq ` <br/>
[boburciu@r220 Rancher_K8s_prereq]$ <br/>
[boburciu@r220 Rancher_K8s_prereq]$ ` ansible-playbook Rancher_prereq_setup.yml -v `
```
PLAY [Set DNS server for resolving the Rancher server URL, 8th] **************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [test]

TASK [Set DNS server and search domain in /etc/netplan/01-netcfg.yaml] *******************************************************
ok: [test] => {"changed": false, "msg": ""}

TASK [Run "sudo netplan apply"] **********************************************************************************************
changed: [test] => {"changed": true, "cmd": ["sudo", "netplan", "apply"], "delta": "0:00:00.169147", "end": "2021-01-24 15:55:37.467471", "rc": 0, "start": "2021-01-24 15:55:37.298324", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}

PLAY RECAP *******************************************************************************************************************
rkem1                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkem2                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew1                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew2                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew3                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew4                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew5                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
rkew6                      : ok=26   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
test                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[boburciu@r220 Rancher_K8s_prereq]$
```

## 2. Create RKE cluster from Rancher server
  ### Copy the command specific for a cluster role from Rancher server:
From the Clusters page, click Add Cluster. <br/>
Choose Custom. <br/>
Enter a Cluster Name. <br/>
Skip Member Roles and Cluster Options. We’ll tell you about them later. <br/>
Click Next. <br/>
From Node Role, select the roles = Worker and copy the node-attachement command <br/>

  ### Verify the role distribution between KVMs soon to be RKE cluster members:
[boburciu@r220 K8s_cluster_RKE]$ ` ansible ubuntu-rke-workers -m command -a "docker ps" `
```
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: We were unable to read either as JSON nor YAML, these are the errors we got from
each: JSON: No JSON object could be decoded  Syntax Error while loading YAML.   found unexpected ':'  The error appears to be in '/etc/ansible/hosts':
line 1, column 5, but may be elsewhere in the file depending on the exact syntax problem.  The offending line appears to be:   [VMs:children]     ^
here
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:2: Section [VMs:children] includes undefined group: rancheos
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
rkew1 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkew5 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkew2 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkew3 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkew4 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkew6 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[boburciu@r220 K8s_cluster_RKE]$
```
[boburciu@r220 K8s_cluster_RKE]$ ` ansible ubuntu-rke-masters -m command -a "docker ps" `
```
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: We were unable to read either as JSON nor YAML, these are the errors we got from
each: JSON: No JSON object could be decoded  Syntax Error while loading YAML.   found unexpected ':'  The error appears to be in '/etc/ansible/hosts':
line 1, column 5, but may be elsewhere in the file depending on the exact syntax problem.  The offending line appears to be:   [VMs:children]     ^
here
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:2: Section [VMs:children] includes undefined group: rancheos
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
rkem2 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
rkem1 | CHANGED | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[boburciu@r220 K8s_cluster_RKE]$
```

  ### Rancher cluster will fail with error _Cluster health check failed: cluster agent is not ready_ if there is no worker node added prior to control plane + etch node addition, per [Rancher issue](https://github.com/rancher/rancher/issues/29715), so we need to add the first node with all 3 roles:
[boburciu@r220 ~]$ ` ssh ubuntu@rkem1 "sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.5 --server https://RancherServer.boburciu.privatecloud.com --token zkqvtg8wp9gc8vwx5czw88n9hxr2h8zk47w6wk4sx6dzhrp8nbnxvv --ca-checksum d7f16d4b95d23022ad944d9670e7c83bf0d8971fa4212c0824cf9e906250fd23 --etcd --controlplane --worker" `

  ### Wait for cluster to become Active

  ### Note that a different hostname needs to be passed for different nodes to be attached, otherwise the current one (already added, rkem1) will get its role modified

  ### Then use a loop to run command to all hosts to fill in the role of RKE --worker node:
[boburciu@r220 ~]$ ` for i in rkew1 rkew2 rkew3 rkew4 rkew5 rkew6; do ssh ubuntu@$i "sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.5 --server https://RancherServer.boburciu.privatecloud.com --token zkqvtg8wp9gc8vwx5czw88n9hxr2h8zk47w6wk4sx6dzhrp8nbnxvv --ca-checksum d7f16d4b95d23022ad944d9670e7c83bf0d8971fa4212c0824cf9e906250fd23 --node-name $i --worker"; done `
```
b26f054c9107975d351293fa52155396fe973dc52a48ac9b11c9a181c650cb91
b2ae34c165712e47d18f5e2710fbcee95ad46d4f5fd22b1160af2096ce22c1f0
43ef32b04242f4e141df2b2662c55053baadaa58c4ed616205537bbd195802f2
9ded2a7ad77cbdae85ea6588562528aab20474b0f598f21f35d78185ae306977
0f20f5b7018bd21c9d8fe2ef3b95c70e7055bde0db1269011405695cd091c9d1
[boburciu@r220 ~]$
```
  ### Last, add another control plane and etcd node:
[boburciu@r220 ~]$ ` ssh ubuntu@rkem2 "sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.5 --server https://RancherServer.boburciu.privatecloud.com --token zkqvtg8wp9gc8vwx5czw88n9hxr2h8zk47w6wk4sx6dzhrp8nbnxvv --ca-checksum d7f16d4b95d23022ad944d9670e7c83bf0d8971fa4212c0824cf9e906250fd23 --node-name rkem2 --etcd --controlplane" `
