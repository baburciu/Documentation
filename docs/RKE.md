# Managing RKE Kubernetes cluster
 
## 0. Setup RKE:
 >  - Considering [official RKE installation guide](https://rancher.com/docs/rke/latest/en/installation/) and downloading from [GitHub RKE release v1.2.3 with K8s v1.19 support](https://github.com/rancher/rke/releases/tag/v1.2.3) we followed [automation guidelines](https://computingforgeeks.com/install-kubernetes-production-cluster-using-rancher-rke/)
 >  - download the RKE binary to a folder in your $PATH, like */usr/local/bin*, and rename it *rke*. Then verify supported versions (including v1.19 for CKAD as of Dec 2020)) <br/>
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