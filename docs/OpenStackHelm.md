# Managing OpenStacK as Helm
 
## Prepare OSH deployment:
 ### - I. First satisfy prerequisites for OSH on the KVMs (K8s cluster):
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` cd OpenStackHelm_prereq/ ` <br/>
[boburciu@r220 OpenStackHelm_prereq]$  <br/>
[boburciu@r220 OpenStackHelm_prereq]$ ` ansible-playbook OpenStackHelm_prereq_setup.yml -v `
```  
[boburciu@r220 ~]$ cd OpenStackHelm_prereq/
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ ls -lt
total 20
-rwx------. 1 boburciu boburciu 7160 Dec  7 21:31 get_helm.sh
-rw-rw-r--. 1 boburciu boburciu  483 Dec  7 20:41 OpenStackHelm_prereq_setup.yml
-rw-rw-r--. 1 boburciu boburciu  908 Dec  7 20:40 install_osh_packages.yml
-rw-rw-r--. 1 boburciu boburciu  573 Dec  7 20:31 set_passwordless_sudo.yml
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ cat OpenStackHelm_prereq_setup.yml
---
- name: satisfy Common Deployment Requirements prereq for OSH install
  # per https://docs.openstack.org/openstack-helm/latest/install/common-requirements.html
  hosts: ubuntu-rke-masters

# ======= How to run
# [boburciu@r220 ~]$ cd OpenStackHelm_prereq/
# [boburciu@r220 OpenStackHelm_prereq]$ ansible-playbook OpenStackHelm_prereq_setup.yml -v

- name: 1st, Passwordless sudo
  import_playbook: set_passwordless_sudo.yml

- name: 2nd, Latest Version Installs
  import_playbook: install_osh_packages.yml

[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ cat set_passwordless_sudo.yml
---
- name: Passwordless sudo
    # per https://docs.openstack.org/openstack-helm/latest/install/common-requirements.html
    # add to /etc/sudoers for each node:

    # root    ALL=(ALL) NOPASSWD: ALL
    # ubuntu  ALL=(ALL) NOPASSWD: ALL
  hosts: ubuntu-rke
  tasks:
    - name: make ubuntu user able to sudo without password check and check visudo for errors
      lineinfile:
        path: /etc/sudoers
        state: present
        line: 'ubuntu  ALL=(ALL) NOPASSWD: ALL'
        line: 'root    ALL=(ALL) NOPASSWD: ALL'
        validate: '/usr/sbin/visudo -cf %s'

[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ cat install_osh_packages.yml
---
- name: Latest Version Installs
    # per https://docs.openstack.org/openstack-helm/latest/install/common-requirements.html
    # on K8s master nodes install the latest versions of Git, CA Certs & Make if necessary

    # #!/bin/bash
    # sudo apt-get update
    # sudo apt-get install --no-install-recommends -y \
    #         ca-certificates \
    #         git \
    #         make \
    #         jq \
    #         nmap \
    #         curl \
    #         uuid-runtime \
    #         bc \
    #         python3-pip
  hosts: ubuntu-rke-masters
  tasks:
    - name: Update all packages to their latest version
      apt:
        name: "*"
        state: latest
    - name: Install a list of packages
      apt:
        pkg:
        - ca-certificates
        - git
        - make
        - jq
        - nmap
        - curl
        - uuid-runtime
        - bc
        - python3-pip

[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ ansible-playbook OpenStackHelm_prereq_setup.yml -v
Using /etc/ansible/ansible.cfg as config file
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: We were unable to read either as JSON nor YAML, these are
the errors we got from each: JSON: No JSON object could be decoded  Syntax Error while loading YAML.   found unexpected ':'
The error appears to be in '/etc/ansible/hosts': line 1, column 5, but may be elsewhere in the file depending on the exact
syntax problem.  The offending line appears to be:   [VMs:children]     ^ here
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:2: Section [VMs:children] includes
undefined group: rancheos
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
[WARNING]: While constructing a mapping from /home/boburciu/OpenStackHelm_prereq/set_passwordless_sudo.yml, line 12, column
9, found a duplicate dict key (line). Using last defined value only.
ERROR! the field 'hosts' is required but was not set
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ ansible-playbook OpenStackHelm_prereq_setup.yml -v
Using /etc/ansible/ansible.cfg as config file
:
PLAY RECAP *******************************************************************************************************************
rkem1                      : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rkem2                      : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rkew1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rkew2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rkew3                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rkew4                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[boburciu@r220 OpenStackHelm_prereq]$
```  

 ### - II. Install helm. OSH uses Helm v2 with Tiller to deploy OpenStack services as containers. There are two parts to Helm: the Helm client (*helm*) and the Helm server (*Tiller*).

 #### - 1. to install Helm client:
` curl -LO https://git.io/get_helm.sh `   <br/>
` chmod 700 get_helm.sh `<br/>
` ./get_helm.sh `<br/>

 #### - 2. the easiest way to install *Tiller* into the cluster is simply to run ` helm init `. This will validate that helm’s local environment is set up correctly (and set it up if necessary) and then it will connect to whatever cluster *kubectl* connects to by default (` kubectl config view `).
```  
[boburciu@r220 OpenStackHelm_prereq]$ pwd
/home/boburciu/OpenStackHelm_prereq
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ ls -lt
total 12
-rw-rw-r--. 1 boburciu boburciu 483 Dec  7 20:41 OpenStackHelm_prereq_setup.yml
-rw-rw-r--. 1 boburciu boburciu 908 Dec  7 20:40 install_osh_packages.yml
-rw-rw-r--. 1 boburciu boburciu 573 Dec  7 20:31 set_passwordless_sudo.yml
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ curl -LO https://git.io/get_helm.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:02 --:--:--     0
100  7160  100  7160    0     0   2428      0  0:00:02  0:00:02 --:--:-- 25211
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ chmod 700 get_helm.sh
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ ./get_helm.sh
Downloading https://get.helm.sh/helm-v2.17.0-linux-amd64.tar.gz
Preparing to install helm and tiller into /usr/local/bin
[sudo] password for boburciu:
helm installed into /usr/local/bin/helm
tiller installed into /usr/local/bin/tiller
Run 'helm init' to configure helm.
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://rkem2:6443
  name: local
contexts:
- context:
    cluster: local
    user: kube-admin-local
  name: local
current-context: local
kind: Config
preferences: {}
users:
- name: kube-admin-local
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ helm init
Creating /home/boburciu/.helm
Creating /home/boburciu/.helm/repository
Creating /home/boburciu/.helm/repository/cache
Creating /home/boburciu/.helm/repository/local
Creating /home/boburciu/.helm/plugins
Creating /home/boburciu/.helm/starters
Creating /home/boburciu/.helm/cache/archive
Creating /home/boburciu/.helm/repository/repositories.yaml
Adding stable repo with URL: https://charts.helm.sh/stable
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /home/boburciu/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://v2.helm.sh/docs/securing_installation/
[boburciu@r220 OpenStackHelm_prereq]$
```
 #### - 3. after `helm init`, you should be able to run `kubectl get pods --namespace kube-system` and see *Tiller* running:
[boburciu@r220 OpenStackHelm_prereq]$ ` kubectl get pods --namespace kube-system -o=wide | grep tiller `
```
tiller-deploy-69c484895f-wmvxn            1/1     Running        0          6m17s   10.42.11.4        rkew3   <none>           <none>
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ kubectl get pods --namespace kube-system -o=wide | head -1
NAME                                      READY   STATUS         RESTARTS   AGE     IP                NODE    NOMINATED NODE   READINESS GATES
```

 ### - III. Clone the OpenStack-Helm Repos
ubuntu@device:~$ ` sudo git clone https://opendev.org/openstack/openstack-helm.git /opt/openstack-helm `
```
Cloning into '/opt/openstack-helm'...
remote: Enumerating objects: 34989, done.
remote: Counting objects: 100% (34989/34989), done.
remote: Compressing objects: 100% (11799/11799), done.
remote: Total 34989 (delta 26576), reused 30294 (delta 22577)
Receiving objects: 100% (34989/34989), 7.37 MiB | 5.61 MiB/s, done.
Resolving deltas: 100% (26576/26576), done.
ubuntu@device:~$
ubuntu@device:~$
```
ubuntu@device:~$ ` sudo git clone https://opendev.org/openstack/openstack-helm-infra.git /opt/openstack-helm-infra `
```
Cloning into '/opt/openstack-helm-infra'...
remote: Enumerating objects: 24032, done.
remote: Counting objects: 100% (24032/24032), done.
remote: Compressing objects: 100% (10069/10069), done.
remote: Total 24032 (delta 17245), reused 19383 (delta 13209)
Receiving objects: 100% (24032/24032), 4.38 MiB | 7.00 MiB/s, done.
Resolving deltas: 100% (17245/17245), done.
ubuntu@device:~$
ubuntu@device:~$
```

 ## Deployment of OSH

 ### - 0. Need to prepare KVM hosts in K8s cluster with proper labeling for pod selector.
 #### In the case of Ceph,it is important to note that Ceph monitors and OSDs are each deployed as a DaemonSet. Be aware that labeling an even number of monitor nodes can result in trouble when trying to reach a quorum. Nodes are labeled according to their Openstack roles:
   ####   **Ceph MON Nodes: ceph-mon**
   ####   **Ceph OSD Nodes: ceph-osd**
   ####   **Ceph MDS Nodes: ceph-mds**
   ####   **Ceph RGW Nodes: ceph-rgw**
   ####   **Control Plane: openstack-control-plane**
   ####   **Compute Nodes: openvswitch, openstack-compute-node**

ubuntu@device:~$ ` kubectl label nodes rkew4 rkew5 rkew6 openstack-control-plane=enabled `
```
node/rkew4 labeled
node/rkew5 labeled
node/rkew6 labeled
ubuntu@device:~$
```
ubuntu@device:~$ ` kubectl label nodes rkew1 rkew2 rkew3 openstack-compute-node=enabled `
```
node/rkew1 labeled
node/rkew2 labeled
node/rkew3 labeled
ubuntu@device:~$
```
ubuntu@device:~$ ` kubectl label nodes rkew1 rkew2 rkew3 openvswitch=enabled `
```
node/rkew1 labeled
node/rkew2 labeled
node/rkew3 labeled
```
ubuntu@device:~$ ` kubectl label nodes rkew1 rkew2 rkew3 linuxbridge=enabled `
```
node/rkew1 labeled
node/rkew2 labeled
node/rkew3 labeled
```
ubuntu@device:~$ ` kubectl label nodes rkew4 rkew5 rkew6 ceph-mon=enabled ceph-osd=enabled ceph-mds=enabled ceph-rgw=enabled ceph-mgr=enabled `
```
node/rkew4 labeled
node/rkew5 labeled
node/rkew6 labeled
ubuntu@device:~$
```
ubuntu@device:~$ ` kubectl get nodes --show-labels `
```
NAME    STATUS   ROLES               AGE     VERSION   LABELS
rkem1   Ready    controlplane,etcd   3d20h   v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkem1,kubernetes.io/os=linux,node-role.kubernetes.io/controlplane=true,node-role.kubernetes.io/etcd=true
rkem2   Ready    controlplane,etcd   3d20h   v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkem2,kubernetes.io/os=linux,node-role.kubernetes.io/controlplane=true,node-role.kubernetes.io/etcd=true
rkew1   Ready    worker              3d20h   v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkew1,kubernetes.io/os=linux,linuxbridge=enabled,node-role.kubernetes.io/worker=true,openstack-compute-node=enabled,openvswitch=enabled
rkew2   Ready    worker              3d20h   v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkew2,kubernetes.io/os=linux,linuxbridge=enabled,node-role.kubernetes.io/worker=true,openstack-compute-node=enabled,openvswitch=enabled
rkew3   Ready    worker              3d20h   v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkew3,kubernetes.io/os=linux,linuxbridge=enabled,node-role.kubernetes.io/worker=true,openstack-compute-node=enabled,openvswitch=enabled
rkew4   Ready    worker              3d20h   v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ceph-mds=enabled,ceph-mgr=enabled,ceph-mon=enabled,ceph-osd=enabled,ceph-rgw=enabled,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkew4,kubernetes.io/os=linux,node-role.kubernetes.io/worker=true,openstack-control-plane=enabled
rkew5   Ready    worker              16m     v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ceph-mds=enabled,ceph-mgr=enabled,ceph-mon=enabled,ceph-osd=enabled,ceph-rgw=enabled,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkew5,kubernetes.io/os=linux,node-role.kubernetes.io/worker=true,openstack-control-plane=enabled
rkew6   Ready    worker              15m     v1.19.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ceph-mds=enabled,ceph-mgr=enabled,ceph-mon=enabled,ceph-osd=enabled,ceph-rgw=enabled,kubernetes.io/arch=amd64,kubernetes.io/hostname=rkew6,kubernetes.io/os=linux,node-role.kubernetes.io/worker=true,openstack-control-plane=enabled
```

 ### - I. Per [OSH multi-node install docs](https://docs.openstack.org/openstack-helm/latest/install/multinode.html) 
ubuntu@device:~$ ` cd /opt/openstack-helm ` <br/> 
ubuntu@device:/opt/openstack-helm$ ` ./tools/deployment/multinode/010-setup-client.sh `
```
:
+ sudo -H mkdir -p /etc/openstack
++ id -un
+ sudo -H chown -R ubuntu: /etc/openstack
+ FEATURE_GATE=tls
+ [[ '' =~ (^|[[:space:]])tls($|[[:space:]]) ]]
+ tee /etc/openstack/clouds.yaml
  clouds:
    openstack_helm:
      region_name: RegionOne
      identity_api_version: 3
      auth:
        username: 'admin'
        password: 'password'
        project_name: 'admin'
        project_domain_name: 'default'
        user_domain_name: 'default'
        auth_url: 'http://keystone.openstack.svc.cluster.local/v3'
+ make helm-toolkit
ubuntu@device:/opt/openstack-helm$
```

 #### -  Doc is incomplete, running _/opt/openstack-helm/tools/deployment/multinode/010-setup-client.sh_ script will not assemble chart and you need to:  <br/>
ubuntu@device:~$ ` cd / ` <br/>
ubuntu@device:/$ ` helm serve & `
```
[1] 6056
ubuntu@device:/$ Regenerating index. This may take a moment.
Now serving you on 127.0.0.1:8879
```
` helm repo add local http://localhost:8879/charts `
```
"local" has been added to your repositories
ubuntu@device:/$
```
ubuntu@device:/$  ` cd /opt/openstack-helm-infra `  <br/>
ubuntu@device:/opt/openstack-helm-infra$ ` sudo make ` 
```
===== Processing [helm-toolkit] chart =====
make[1]: Entering directory '/opt/openstack-helm-infra'
if [ -f helm-toolkit/Makefile ]; then make -C helm-toolkit; fi
if [ -f helm-toolkit/requirements.yaml ]; then helm dep up helm-toolkit; fi
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete.
Saving 0 charts
Deleting outdated charts
if [ -d helm-toolkit ]; then helm lint helm-toolkit; fi
==> Linting helm-toolkit
Lint OK

1 chart(s) linted, no failures
if [ -d helm-toolkit ]; then helm package helm-toolkit; fi
Successfully packaged chart and saved it to: /opt/openstack-helm-infra/helm-toolkit-0.1.5.tgz
make[1]: Leaving directory '/opt/openstack-helm-infra'

===== Processing [helm-toolkit] chart =====
===== Processing [gnocchi] chart =====
===== Processing [elasticsearch] chart =====
===== Processing [calico] chart =====
===== Processing [fluentd] chart =====
===== Processing [ceph-rgw] chart =====
===== Processing [powerdns] chart =====
===== Processing [prometheus-kube-state-metrics] chart =====
===== Processing [nfs-provisioner] chart =====
===== Processing [prometheus-process-exporter] chart =====
===== Processing [flannel] chart =====
===== Processing [mongodb] chart =====
===== Processing [kibana] chart =====
===== Processing [libvirt] chart =====
===== Processing [podsecuritypolicy] chart =====
===== Processing [ldap] chart =====
===== Processing [fluentbit] chart =====
===== Processing [ceph-osd] chart =====
===== Processing [zookeeper] chart =====
===== Processing [kafka] chart =====
===== Processing [redis] chart =====
===== Processing [grafana] chart =====
===== Processing [nagios] chart =====
===== Processing [rabbitmq] chart =====
===== Processing [daemonjob-controller] chart =====
===== Processing [prometheus-node-exporter] chart =====
===== Processing [local-storage] chart =====
===== Processing [kubernetes-keystone-webhook] chart =====
===== Processing [openvswitch] chart =====
===== Processing [elastic-apm-server] chart =====
===== Processing [prometheus-openstack-exporter] chart =====
===== Processing [ceph-mon] chart =====
===== Processing [mariadb] chart =====
===== Processing [ca-issuer] chart =====
===== Processing [prometheus] chart =====
===== Processing [namespace-config] chart =====
===== Processing [kube-dns] chart =====
===== Processing [prometheus-blackbox-exporter] chart =====
===== Processing [prometheus-alertmanager] chart =====
===== Processing [ceph-client] chart =====
===== Processing [falco] chart =====
===== Processing [elastic-filebeat] chart =====
===== Processing [alerta] chart =====
===== Processing [metacontroller] chart =====
===== Processing [ceph-provisioners] chart =====
===== Processing [ingress] chart =====
===== Processing [memcached] chart =====
===== Processing [etcd] chart =====
===== Processing [tiller] chart =====
===== Processing [lockdown] chart =====
===== Processing [registry] chart =====
===== Processing [kubernetes-node-problem-detector] chart =====
===== Processing [postgresql] chart =====
===== Processing [elastic-packetbeat] chart =====
===== Processing [elastic-metricbeat] chart =====
```

 ### - II. Then _/opt/openstack-helm/./tools/deployment/component/common/ingress.sh_ can be run
 ##### -  Obs, if it returns the below error:
ubuntu@device:/opt/openstack-helm-infra$ ` cd /opt/openstack-helm ` <br/>
ubuntu@device:/opt/openstack-helm$ ` sudo OSH_DEPLOY_MULTINODE=True ./tools/deployment/component/common/ingress.sh `
```
:
:
+ helm upgrade --install ingress-kube-system ../openstack-helm-infra/ingress --namespace=kube-system --values=/tmp/ingress-kube-system.yaml
UPGRADE FAILED
Error: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list resource "configmaps" in API group "" in the namespace "kube-system"
Error: UPGRADE FAILED: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list resource "configmaps" in API group "" in the namespace "kube-system"
ubuntu@device:/opt/openstack-helm$ kubectl [--namespace kube-system] get serviceaccount
Error: unknown command "[--namespace" for "kubectl"
Run 'kubectl --help' for usage.
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$
```
 ##### -  You need to:
 ubuntu@device:/opt/openstack-helm$ ` kubectl create serviceaccount --namespace kube-system tiller `
```
serviceaccount/tiller created
ubuntu@device:/opt/openstack-helm$
```
ubuntu@device:/opt/openstack-helm$ ` kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller `
```
clusterrolebinding.rbac.authorization.k8s.io/tiller-cluster-rule created
ubuntu@device:/opt/openstack-helm$
```
ubuntu@device:/opt/openstack-helm$ ` kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' `
```
deployment.apps/tiller-deploy patched (no change)
ubuntu@device:/opt/openstack-helm$
```
ubuntu@device:/opt/openstack-helm$ ` helm init --upgrade --service-account tiller `
```
$HELM_HOME has been configured at /home/ubuntu/.helm.

Tiller (the Helm server-side component) has been updated to ghcr.io/helm/tiller:v2.17.0 .
ubuntu@device:/opt/openstack-helm$
```
 ##### -  Run the script
ubuntu@device:/opt/openstack-helm$ ` sudo OSH_DEPLOY_MULTINODE=True ./tools/deployment/component/common/ingress.sh `

 ##### -  In case scaling down and then up the ds (DaemonSet) is needed:
```
ubuntu@device:~$
ubuntu@device:~$ kubectl get daemonset -n kube-system
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
calico-node   8         8         8       8            8           kubernetes.io/os=linux            3d21h
ingress       3         3         0       3            0           openstack-control-plane=enabled   133m
ubuntu@device:~$
ubuntu@device:~$
ubuntu@device:~$ kubectl -n kube-system patch daemonset ingress -p '{"spec": {"template": {"spec": {"nodeSelector": {"non-existing": "true"}}}}}'
daemonset.apps/ingress patched
ubuntu@device:~$
ubuntu@device:~$
ubuntu@device:~$ kubectl get daemonset  -n kube-system
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                       AGE
calico-node   8         8         8       8            8           kubernetes.io/os=linux                              3d21h
ingress       0         0         0       0            0           non-existing=true,openstack-control-plane=enabled   136m
ubuntu@device:~$
ubuntu@device:~$ kubectl -n kube-system patch daemonset ingress  --type json -p='[{"op": "remove", "path": "/spec/template/spec/nodeSelector/non-existing"}]'
daemonset.apps/ingress patched
ubuntu@device:~$
ubuntu@device:~$ kubectl get daemonset  -n kube-system
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
calico-node   8         8         8       8            8           kubernetes.io/os=linux            3d21h
ingress       3         3         0       3            0           openstack-control-plane=enabled   136m
ubuntu@device:~$
```

 ### - III. Create loopback devices for CEPH
 ##### -  Check the K8s nodes for CEPH-OSD:
```
ubuntu@device:/opt/openstack-helm$ kubectl get nodes -o=wide
NAME    STATUS   ROLES               AGE     VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
rkem1   Ready    controlplane,etcd   3d21h   v1.19.4   192.168.122.192   <none>        Ubuntu 18.04.5 LTS   4.15.0-126-generic   docker://19.3.14
rkem2   Ready    controlplane,etcd   3d21h   v1.19.4   192.168.122.144   <none>        Ubuntu 18.04.5 LTS   4.15.0-126-generic   docker://19.3.14
rkew1   Ready    worker              3d21h   v1.19.4   192.168.122.180   <none>        Ubuntu 18.04.5 LTS   4.15.0-126-generic   docker://19.3.14
rkew2   Ready    worker              3d21h   v1.19.4   192.168.122.167   <none>        Ubuntu 18.04.5 LTS   4.15.0-126-generic   docker://19.3.14
rkew3   Ready    worker              3d21h   v1.19.4   192.168.122.55    <none>        Ubuntu 18.04.5 LTS   4.15.0-126-generic   docker://19.3.14
rkew4   Ready    worker              3d21h   v1.19.4   192.168.122.142   <none>        Ubuntu 18.04.5 LTS   4.15.0-126-generic   docker://19.3.14
rkew5   Ready    worker              52m     v1.19.4   192.168.122.141   <none>        Ubuntu 18.04.5 LTS   4.15.0-20-generic    docker://19.3.14
rkew6   Ready    worker              52m     v1.19.4   192.168.122.88    <none>        Ubuntu 18.04.5 LTS   4.15.0-20-generic    docker://19.3.14
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$
```
ubuntu@device:/opt/openstack-helm$ ` kubectl get nodes --selector="ceph-osd=enabled" `
```
NAME    STATUS   ROLES    AGE     VERSION
rkew4   Ready    worker   3d21h   v1.19.4
rkew5   Ready    worker   55m     v1.19.4
rkew6   Ready    worker   54m     v1.19.4
ubuntu@device:/opt/openstack-helm$
```

 ##### -  Transfer the Ansible inventory file and the script to be run on all targe nodes (CEPH-labeled) from _rkem1_ in order to run Ansible ad-hoc from CentOS host, since all K8s nodes are SSH reachable from this host:
[boburciu@r220 K8s_cluster_RKE]$ ` cd ~/OpenStackHelm_prereq/ `
```
[boburciu@r220 OpenStackHelm_prereq]$ ls -lt
total 20
-rw-rw-r--. 1 boburciu boburciu  512 Dec  7 21:48 OpenStackHelm_prereq_setup.yml
-rw-rw-r--. 1 boburciu boburciu  571 Dec  7 21:47 set_passwordless_sudo.yml
-rw-rw-r--. 1 boburciu boburciu  917 Dec  7 21:47 install_osh_packages.yml
-rwx------. 1 boburciu boburciu 7160 Dec  7 21:31 get_helm.sh
[boburciu@r220 OpenStackHelm_prereq]$
```
[boburciu@r220 OpenStackHelm_prereq]$ ` scp -3 -r ubuntu@rkem1:/opt/openstack-helm-infra/tools/gate/devel/multinode-inventory_modifiedBurciu.yaml ./multinode-inventory_modifiedBurciu.yaml `
```
multinode-inventory_modifiedBurciu.yaml                                                     100% 1103   806.2KB/s   00:00
```
[boburciu@r220 OpenStackHelm_prereq]$ `scp -3 -r ubuntu@rkem1:/opt/openstack-helm/tools/deployment/common/setup-ceph-loopback-device.sh ./setup-ceph-loopback-device.sh `
```
setup-ceph-loopback-device.sh                                                               100% 1727     1.5MB/s   00:00
[boburciu@r220 OpenStackHelm_prereq]$ ls -lt
total 28
-rwxr-xr-x. 1 boburciu boburciu 1727 Dec  9 22:27 setup-ceph-loopback-device.sh
-rw-r--r--. 1 boburciu boburciu 1309 Dec  9 22:24 multinode-inventory_modifiedBurciu.yaml
-rw-rw-r--. 1 boburciu boburciu  512 Dec  7 21:48 OpenStackHelm_prereq_setup.yml
-rw-rw-r--. 1 boburciu boburciu  571 Dec  7 21:47 set_passwordless_sudo.yml
-rw-rw-r--. 1 boburciu boburciu  917 Dec  7 21:47 install_osh_packages.yml
-rwx------. 1 boburciu boburciu 7160 Dec  7 21:31 get_helm.sh
[boburciu@r220 OpenStackHelm_prereq]$

[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ vi multinode-inventory_modifiedBurciu.yaml
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ cat multinode-inventory_modifiedBurciu.yaml
---
all:
  children:
    primary:
      hosts:
        rkew4:
          ansible_port: 22
          ansible_host: rke4
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_rsa
          ansible_ssh_extra_args: -o StrictHostKeyChecking=no
    nodes:
      hosts:
        rkew5:
          ansible_port: 22
          ansible_host: rkew5
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_rsa
          ansible_ssh_extra_args: -o StrictHostKeyChecking=no
        rkew6:
          ansible_port: 22
          ansible_host: rkew6
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_rsa
          ansible_ssh_extra_args: -o StrictHostKeyChecking=no
...
[boburciu@r220 OpenStackHelm_prereq]$
```

 ##### -  Since a script needs to be run on all target KVMs (worker nodes), we transfer it and the run it with Ansible ad-hoc:

```
ubuntu@device:~$
ubuntu@device:~$ ls -lt
total 20
-r--r----- 1 root root 17251 Dec  9 19:08 rancher_docker_install.sh
ubuntu@device:~$
ubuntu@device:~$
```
[boburciu@r220 ~]$ ` cd /home/boburciu/OpenStackHelm_prereq ` <br/>
[boburciu@r220 OpenStackHelm_prereq]$ ` ansible all -i ./multinode-inventory_modifiedBurciu.yaml -m ansible.builtin.copy -a "src=./setup-ceph-loopback-device.sh dest=~/setup-ceph-loopback-device.sh mode=600 owner=ubuntu group=ubuntu" `
```
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew6 should use /usr/bin/python3, but is using /usr/bin/python for backward
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host.
See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
rkew6 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "a71b98a281ccea721435fdd1a2bc857d5caf6f79",
    "dest": "/home/ubuntu/setup-ceph-loopback-device.sh",
    "gid": 1000,
    "group": "ubuntu",
    "mode": "0770",
    "owner": "ubuntu",
    "path": "/home/ubuntu/setup-ceph-loopback-device.sh",
    "size": 1727,
    "state": "file",
    "uid": 1000
}
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew5 should use /usr/bin/python3, but is using /usr/bin/python for backward
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host.
See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
rkew5 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "a71b98a281ccea721435fdd1a2bc857d5caf6f79",
    "dest": "/home/ubuntu/setup-ceph-loopback-device.sh",
    "gid": 1000,
    "group": "ubuntu",
    "mode": "0770",
    "owner": "ubuntu",
    "path": "/home/ubuntu/setup-ceph-loopback-device.sh",
    "size": 1727,
    "state": "file",
    "uid": 1000
}
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew4 should use /usr/bin/python3, but is using /usr/bin/python for backward
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host.
See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
rkew4 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "a71b98a281ccea721435fdd1a2bc857d5caf6f79",
    "dest": "/home/ubuntu/setup-ceph-loopback-device.sh",
    "gid": 1000,
    "group": "ubuntu",
    "mode": "0770",
    "owner": "ubuntu",
    "path": "/home/ubuntu/setup-ceph-loopback-device.sh",
    "size": 1727,
    "state": "file",
    "uid": 1000
}
[boburciu@r220 OpenStackHelm_prereq]$

ubuntu@device:~$
ubuntu@device:~$ ls -lt
total 24
-rwxrwx--- 1 ubuntu ubuntu  1727 Dec 11 19:54 setup-ceph-loopback-device.sh
-r--r----- 1 root   root   17251 Dec  9 19:08 rancher_docker_install.sh
ubuntu@device:~$
ubuntu@device:~$
```
[boburciu@r220 OpenStackHelm_prereq]$ ` ansible all -i ./multinode-inventory_modifiedBurciu.yaml -m shell -a '~/setup-ceph-loopback-device.sh --ceph-osd-data /dev/loop0 --ceph-osd-dbwal /dev/loop1' -vv `
```
ansible 2.9.15
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/boburciu/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Oct 14 2020, 14:45:30) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
Using /etc/ansible/ansible.cfg as config file
META: ran handlers
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew4 should use /usr/bin/python3, but is using /usr/bin/python for backward
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host.
See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
rkew4 | CHANGED | rc=0 >>
/dev/loop1: [64768]:538475 (/var/lib/openstack-helm/ceph/ceph-osd-db-wal-loopbackfile.img)
/dev/loop0: [64768]:538474 (/var/lib/openstack-helm/ceph/ceph-osd-data-loopbackfile.img)
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew6 should use /usr/bin/python3, but is using /usr/bin/python for backward
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host.
See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
rkew6 | CHANGED | rc=0 >>
/dev/loop1: [64768]:541195 (/var/lib/openstack-helm/ceph/ceph-osd-db-wal-loopbackfile.img)
/dev/loop0: [64768]:541194 (/var/lib/openstack-helm/ceph/ceph-osd-data-loopbackfile.img)
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew5 should use /usr/bin/python3, but is using /usr/bin/python for backward
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host.
See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be
removed in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
rkew5 | CHANGED | rc=0 >>
/dev/loop1: [64768]:278361 (/var/lib/openstack-helm/ceph/ceph-osd-db-wal-loopbackfile.img)
/dev/loop0: [64768]:278360 (/var/lib/openstack-helm/ceph/ceph-osd-data-loopbackfile.img)
META: ran handlers
META: ran handlers
[boburciu@r220 OpenStackHelm_prereq]$
```

 ### - IV. Deploy Ceph
 ##### - Run the script

ubuntu@device:~$ ` cd /opt/openstack-helm ` <br/>
ubuntu@device:/opt/openstack-helm$ ` sudo ./tools/deployment/multinode/030-ceph.sh `
```
+ '[' -s /tmp/ceph-fs-uuid.txt ']'
+ uuidgen
++ ./tools/deployment/multinode/kube-node-subnet.sh
Flag --generator has been deprecated, has no effect and will be removed in the future.
Flag --generator has been deprecated, has no effect and will be removed in the future.
+ CEPH_PUBLIC_NETWORK=192.168.122.0/24
+ CEPH_CLUSTER_NETWORK=192.168.122.0/24
++ cat /tmp/ceph-fs-uuid.txt
:
:

+ ./tools/deployment/common/wait-for-pods.sh ceph 1200
Containers failed to start after 1200 seconds

NAME                                   READY   STATUS      RESTARTS   AGE   IP                NODE    NOMINATED NODE   READINESS GATES
ceph-bootstrap-x88wx                   0/1     Completed   0          26m   10.42.176.7       rkew5   <none>           <none>
ceph-checkdns-758fd57744-bbf8k         1/1     Running     0          20m   192.168.122.88    rkew6   <none>           <none>
ceph-mds-59d4564c65-bndvd              0/1     Init:0/2    0          20m   10.42.176.9       rkew5   <none>           <none>
ceph-mds-59d4564c65-hlfmb              0/1     Init:0/2    0          20m   10.42.212.138     rkew6   <none>           <none>
ceph-mds-keyring-generator-9wpzt       0/1     Completed   0          26m   10.42.212.134     rkew6   <none>           <none>
ceph-mgr-857b56f6dc-g9mw6              1/1     Running     0          20m   192.168.122.88    rkew6   <none>           <none>
ceph-mgr-857b56f6dc-lh82p              1/1     Running     0          20m   192.168.122.141   rkew5   <none>           <none>
ceph-mgr-keyring-generator-mpzl5       0/1     Completed   0          26m   10.42.212.135     rkew6   <none>           <none>
ceph-mon-49jfp                         1/1     Running     0          26m   192.168.122.88    rkew6   <none>           <none>
ceph-mon-5mkrl                         1/1     Running     0          26m   192.168.122.142   rkew4   <none>           <none>
ceph-mon-check-fc8b6f6cf-ztp5q         1/1     Running     0          26m   10.42.176.8       rkew5   <none>           <none>
ceph-mon-keyring-generator-l9zc6       0/1     Completed   0          26m   10.42.212.137     rkew6   <none>           <none>
ceph-mon-pwphw                         1/1     Running     0          26m   192.168.122.141   rkew5   <none>           <none>
ceph-osd-default-83945928-4tl62        2/2     Running     0          22m   192.168.122.141   rkew5   <none>           <none>
ceph-osd-default-83945928-jv5m2        2/2     Running     0          22m   192.168.122.88    rkew6   <none>           <none>
ceph-osd-default-83945928-qhbxh        2/2     Running     0          22m   192.168.122.142   rkew4   <none>           <none>
ceph-osd-keyring-generator-r9pw8       0/1     Completed   0          26m   10.42.212.136     rkew6   <none>           <none>
ceph-pool-checkpgs-1607717700-6rslp    0/1     Init:0/1    0          18m   192.168.122.141   rkew5   <none>           <none>
ceph-rbd-pool-xd65f                    1/1     Running     3          20m   10.42.212.139     rkew6   <none>           <none>
ceph-storage-keys-generator-rlq49      0/1     Completed   0          26m   10.42.176.6       rkew5   <none>           <none>
ingress-8c574f9b4-gnl5s                1/1     Running     0          97m   10.42.44.83       rkew4   <none>           <none>
ingress-8c574f9b4-jq4k6                1/1     Running     1          2d    10.42.44.78       rkew4   <none>           <none>
ingress-error-pages-6fb6d67c57-r8w9w   1/1     Running     1          2d    10.42.44.79       rkew4   <none>           <none>
ingress-error-pages-6fb6d67c57-rj58x   1/1     Running     0          96m   10.42.44.87       rkew4   <none>           <none>

Some pods are in pending state:
NAME                                  READY   STATUS     RESTARTS   AGE   IP                NODE    NOMINATED NODE   READINESS GATES
ceph-mds-59d4564c65-bndvd             0/1     Init:0/2   0          20m   10.42.176.9       rkew5   <none>           <none>
ceph-mds-59d4564c65-hlfmb             0/1     Init:0/2   0          20m   10.42.212.138     rkew6   <none>           <none>
ceph-pool-checkpgs-1607717700-6rslp   0/1     Init:0/1   0          18m   192.168.122.141   rkew5   <none>           <none>
Some jobs have not succeeded
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$
```

 ### - V. Deploy Ceph
 ##### - Run the script

ubuntu@device:/opt/openstack-helm$ ` sudo ./tools/deployment/multinode/040-ceph-ns-activate.sh `
```
++ ./tools/deployment/multinode/kube-node-subnet.sh
Flag --generator has been deprecated, has no effect and will be removed in the future.
Flag --generator has been deprecated, has no effect and will be removed in the future.
+ CEPH_PUBLIC_NETWORK=192.168.122.0/24
+ CEPH_CLUSTER_NETWORK=192.168.122.0/24
+ tee /tmp/ceph-openstack-config.yaml
endpoints:
  ceph_mon:
    namespace: ceph
network:
  public: 192.168.122.0/24
  cluster: 192.168.122.0/24
deployment:
  ceph: false
  rbd_provisioner: false
  cephfs_provisioner: false
  client_secrets: true
bootstrap:
  enabled: false
storageclass:
  cephfs:
    provision_storage_class: false
+ : ../openstack-helm-infra
+ helm upgrade --install ceph-openstack-config ../openstack-helm-infra/ceph-provisioners --namespace=openstack --values=/tmp/ceph-openstack-config.yaml
Release "ceph-openstack-config" does not exist. Installing it now.
NAME:   ceph-openstack-config
LAST DEPLOYED: Fri Dec 11 21:15:47 2020
NAMESPACE: openstack
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                        CREATED AT
ceph-openstack-config-test  2020-12-11T21:15:48Z

==> v1/ClusterRoleBinding
NAME                        ROLE                                    AGE
ceph-openstack-config-test  ClusterRole/ceph-openstack-config-test  0s

==> v1/ConfigMap
NAME                                         DATA  AGE
ceph-etc                                     1     0s
ceph-openstack-config-ceph-prov-bin-clients  3     0s

==> v1/Job
NAME                                         COMPLETIONS  DURATION  AGE
ceph-openstack-config-ceph-ns-key-generator  0/1          0s        0s

==> v1/Pod(related)
NAME                                               READY  STATUS    RESTARTS  AGE
ceph-openstack-config-ceph-ns-key-generator-q72fm  0/1    Init:0/1  0         0s

==> v1/Role
NAME                                               CREATED AT
ceph-openstack-config-ceph-ns-key-cleaner          2020-12-11T21:15:48Z
ceph-openstack-config-ceph-ns-key-generator        2020-12-11T21:15:48Z
ceph-openstack-config-ceph-ns-key-generator-0h7zc  2020-12-11T21:15:48Z

==> v1/RoleBinding
NAME                                               ROLE                                                    AGE
ceph-openstack-config-ceph-ns-key-cleaner          Role/ceph-openstack-config-ceph-ns-key-cleaner          0s
ceph-openstack-config-ceph-ns-key-generator        Role/ceph-openstack-config-ceph-ns-key-generator        0s
ceph-openstack-config-ceph-ns-key-generator-0h7zc  Role/ceph-openstack-config-ceph-ns-key-generator-0h7zc  0s

==> v1/ServiceAccount
NAME                                         SECRETS  AGE
ceph-openstack-config-ceph-ns-key-cleaner    1        0s
ceph-openstack-config-ceph-ns-key-generator  1        0s
ceph-openstack-config-test                   1        0s


+ ./tools/deployment/common/wait-for-pods.sh openstack
+ helm status ceph-openstack-config
LAST DEPLOYED: Fri Dec 11 21:15:47 2020
NAMESPACE: openstack
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                        CREATED AT
ceph-openstack-config-test  2020-12-11T21:15:48Z

==> v1/ClusterRoleBinding
NAME                        ROLE                                    AGE
ceph-openstack-config-test  ClusterRole/ceph-openstack-config-test  6s

==> v1/ConfigMap
NAME                                         DATA  AGE
ceph-etc                                     1     6s
ceph-openstack-config-ceph-prov-bin-clients  3     6s

==> v1/Job
NAME                                         COMPLETIONS  DURATION  AGE
ceph-openstack-config-ceph-ns-key-generator  1/1          3s        6s

==> v1/Pod(related)
NAME                                               READY  STATUS     RESTARTS  AGE
ceph-openstack-config-ceph-ns-key-generator-q72fm  0/1    Completed  0         6s

==> v1/Role
NAME                                               CREATED AT
ceph-openstack-config-ceph-ns-key-cleaner          2020-12-11T21:15:48Z
ceph-openstack-config-ceph-ns-key-generator        2020-12-11T21:15:48Z
ceph-openstack-config-ceph-ns-key-generator-0h7zc  2020-12-11T21:15:48Z

==> v1/RoleBinding
NAME                                               ROLE                                                    AGE
ceph-openstack-config-ceph-ns-key-cleaner          Role/ceph-openstack-config-ceph-ns-key-cleaner          6s
ceph-openstack-config-ceph-ns-key-generator        Role/ceph-openstack-config-ceph-ns-key-generator        6s
ceph-openstack-config-ceph-ns-key-generator-0h7zc  Role/ceph-openstack-config-ceph-ns-key-generator-0h7zc  6s

==> v1/ServiceAccount
NAME                                         SECRETS  AGE
ceph-openstack-config-ceph-ns-key-cleaner    1        6s
ceph-openstack-config-ceph-ns-key-generator  1        6s
ceph-openstack-config-test                   1        6s


ubuntu@device:/opt/openstack-helm$
```

 ### - V. Deploy MariaDB
 ##### - Run the script

ubuntu@device:/opt/openstack-helm$ ` sudo ./tools/deployment/multinode/050-mariadb.sh `
```
+ tee /tmp/mariadb.yaml
pod:
  replicas:
    server: 3
    ingress: 3
+ : ../openstack-helm-infra
+ helm upgrade --install mariadb ../openstack-helm-infra/mariadb --namespace=openstack --values=/tmp/mariadb.yaml
Release "mariadb" does not exist. Installing it now.
NAME:   mariadb
LAST DEPLOYED: Fri Dec 11 21:17:55 2020
NAMESPACE: openstack
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                  DATA  AGE
mariadb-bin           5     1s
mariadb-etc           3     1s
mariadb-ingress-conf  1     1s
mariadb-ingress-etc   1     1s
mariadb-services-tcp  1     1s

==> v1/Deployment
NAME                         READY  UP-TO-DATE  AVAILABLE  AGE
mariadb-ingress              0/3    3           0          1s
mariadb-ingress-error-pages  0/1    1           0          1s

==> v1/Pod(related)
NAME                                         READY  STATUS    RESTARTS  AGE
mariadb-ingress-7d8c66db8f-4v2cd             0/1    Init:0/1  0         1s
mariadb-ingress-7d8c66db8f-k8ljv             0/1    Init:0/1  0         1s
mariadb-ingress-7d8c66db8f-zs5bd             0/1    Init:0/1  0         1s
mariadb-ingress-error-pages-856566b76-hfwzn  0/1    Init:0/1  0         1s
mariadb-server-0                             0/1    Pending   0         2s
mariadb-server-1                             0/1    Pending   0         2s
mariadb-server-2                             0/1    Pending   0         1s

==> v1/Role
NAME                               CREATED AT
mariadb-ingress                    2020-12-11T21:17:56Z
mariadb-mariadb                    2020-12-11T21:17:56Z
mariadb-openstack-mariadb-ingress  2020-12-11T21:17:56Z
mariadb-openstack-mariadb-test     2020-12-11T21:17:56Z

==> v1/RoleBinding
NAME                     ROLE                                    AGE
mariadb-ingress          Role/mariadb-ingress                    1s
mariadb-mariadb          Role/mariadb-mariadb                    1s
mariadb-mariadb-ingress  Role/mariadb-openstack-mariadb-ingress  1s
mariadb-mariadb-test     Role/mariadb-openstack-mariadb-test     1s

==> v1/Secret
NAME                      TYPE    DATA  AGE
mariadb-dbadmin-password  Opaque  1     1s
mariadb-dbaudit-password  Opaque  1     1s
mariadb-dbsst-password    Opaque  1     1s
mariadb-secrets           Opaque  2     1s

==> v1/Service
NAME                         TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)            AGE
mariadb                      ClusterIP  10.43.92.165  <none>       3306/TCP           1s
mariadb-discovery            ClusterIP  None          <none>       3306/TCP,4567/TCP  1s
mariadb-ingress-error-pages  ClusterIP  None          <none>       80/TCP             1s
mariadb-server               ClusterIP  10.43.208.9   <none>       3306/TCP           1s

==> v1/ServiceAccount
NAME                         SECRETS  AGE
mariadb-ingress              1        1s
mariadb-ingress-error-pages  1        1s
mariadb-mariadb              1        1s
mariadb-test                 1        1s

==> v1/StatefulSet
NAME            READY  AGE
mariadb-server  0/3    1s

==> v1beta1/PodDisruptionBudget
NAME            MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
mariadb-server  0              N/A              0                    1s


+ ./tools/deployment/common/wait-for-pods.sh openstack
Containers failed to start after 900 seconds

NAME                                                READY   STATUS      RESTARTS   AGE    IP              NODE     NOMINATED NODE   READINESS GATES
ceph-openstack-config-ceph-ns-key-generator-q72fm   0/1     Completed   0          17m    10.42.212.140   rkew6    <none>           <none>
ingress-6cbc96b8fd-5n6bn                            1/1     Running     0          156m   10.42.44.82     rkew4    <none>           <none>
ingress-6cbc96b8fd-lwtkl                            1/1     Running     0          156m   10.42.44.85     rkew4    <none>           <none>
ingress-error-pages-97b98747c-gfjx2                 1/1     Running     0          156m   10.42.44.81     rkew4    <none>           <none>
ingress-error-pages-97b98747c-wq9tf                 1/1     Running     0          156m   10.42.44.86     rkew4    <none>           <none>
mariadb-ingress-7d8c66db8f-4v2cd                    0/1     Running     0          15m    10.42.176.12    rkew5    <none>           <none>
mariadb-ingress-7d8c66db8f-k8ljv                    0/1     Running     0          15m    10.42.212.141   rkew6    <none>           <none>
mariadb-ingress-7d8c66db8f-zs5bd                    0/1     Running     0          15m    10.42.44.90     rkew4    <none>           <none>
mariadb-ingress-error-pages-856566b76-hfwzn         1/1     Running     0          15m    10.42.176.11    rkew5    <none>           <none>
mariadb-server-0                                    0/1     Pending     0          15m    <none>          <none>   <none>           <none>
mariadb-server-1                                    0/1     Pending     0          15m    <none>          <none>   <none>           <none>
mariadb-server-2                                    0/1     Pending     0          15m    <none>          <none>   <none>           <none>

Some pods are in pending state:
NAME               READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
mariadb-server-0   0/1     Pending   0          15m   <none>   <none>   <none>           <none>
mariadb-server-1   0/1     Pending   0          15m   <none>   <none>   <none>           <none>
mariadb-server-2   0/1     Pending   0          15m   <none>   <none>   <none>           <none>
Some pods are not ready
ubuntu@device:/opt/openstack-helm$
```

 ### - T-Shoot tries
 ##### - Verifying OpenStack pods state:
```
[boburciu@r220 ~]$ ssh ubuntu@rkem1
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Fri Dec 11 21:16:08 2020 from 192.168.122.1
ubuntu@device:~$
ubuntu@device:~$
```
ubuntu@device:~$ ` kubectl get po --all-namespaces -o=wide | gawk 'match($3, /([0-9])+\/([0-9])+/, a) {if (a[1] < a [2] && $4 != "Completed") print $0}' | cat -n ` 
```
     1  ceph            ceph-mds-59d4564c65-4ljd7                           0/1     Init:0/2                0          3h53m   10.42.176.17      rkew5    <none>           <none>
     2  ceph            ceph-mds-59d4564c65-kr5qx                           0/1     Init:0/2                0          3h53m   10.42.212.143     rkew6    <none>           <none>
     3  ceph            ceph-osd-default-83945928-4j9lw                     0/2     Init:CrashLoopBackOff   50         3h51m   192.168.122.88    rkew6    <none>           <none>
     4  ceph            ceph-osd-default-83945928-fvshr                     0/2     Init:CrashLoopBackOff   50         3h51m   192.168.122.141   rkew5    <none>           <none>
     5  ceph            ceph-osd-default-83945928-mbhkr                     0/2     Init:CrashLoopBackOff   50         3h51m   192.168.122.142   rkew4    <none>           <none>
     6  ceph            ceph-pool-checkpgs-1607717700-9tg9j                 0/1     Init:0/1                0          4h17m   192.168.122.88    rkew6    <none>           <none>
     7  ceph            ceph-pool-checkpgs-1607717700-rgl8n                 0/1     Init:Error              0          5d20h   192.168.122.88    rkew6    <none>           <none>
     8  ingress-nginx   default-http-backend-65dd5949d9-djs9t               0/1     NodeAffinity            0          11d     <none>            rkew3    <none>           <none>
     9  kube-system     coredns-6f85d5fb88-t7nfs                            0/1     NodeAffinity            0          11d     <none>            rkew2    <none>           <none>
    10  kube-system     ingress-cm57s                                       0/1     Pending                 0          7d22h   <none>            <none>   <none>           <none>
    11  kube-system     ingress-klsxx                                       0/1     Pending                 0          7d22h   <none>            <none>   <none>           <none>
    12  kube-system     ingress-ptqfz                                       0/1     Pending                 0          7d22h   <none>            <none>   <none>           <none>
    13  kube-system     metrics-server-8449844bf-8smnh                      0/1     NodeAffinity            0          11d     <none>            rkew4    <none>           <none>
    14  openstack       mariadb-ingress-7d8c66db8f-4v2cd                    0/1     Running                 1          5d20h   10.42.176.15      rkew5    <none>           <none>
    15  openstack       mariadb-ingress-7d8c66db8f-k8ljv                    0/1     Running                 1          5d20h   10.42.212.142     rkew6    <none>           <none>
    16  openstack       mariadb-ingress-7d8c66db8f-zs5bd                    0/1     Running                 1          5d20h   10.42.44.92       rkew4    <none>           <none>
    17  openstack       mariadb-server-0                                    0/1     Pending                 0          5d20h   <none>            <none>   <none>           <none>
    18  openstack       mariadb-server-1                                    0/1     Pending                 0          5d20h   <none>            <none>   <none>           <none>
    19  openstack       mariadb-server-2                                    0/1     Pending                 0          5d20h   <none>            <none>   <none>           <none>
ubuntu@device:~$
```

 ##### - Job=_ceph-rbd-pool_ & pod=_ceph-rbd-pool-_ is dependent on Ceph OSD:
``` 

ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$ kubectl -n ceph logs ceph-rbd-pool-thw56 -c ceph-rbd-pool
:
:
+ EXPECTED_OSDS=3
+ REQUIRED_PERCENT_OF_OSDS=75
+ '[' 0 -gt 3 ']'
+ MIN_OSDS=2
+ '[' 2 -lt 1 ']'
+ '[' '' ']'
+ '[' 3 -eq 0 ']'
+ '[' 3 -ge 2 ']'
+ '[' 0 -ge 2 ']'
Required number of OSDs (2) are NOT UP and IN status. Cluster shows OSD count=3, UP=0, IN=3
+ echo 'Required number of OSDs (2) are NOT UP and IN status. Cluster shows OSD count=3, UP=0, IN=3'
+ exit 1
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$ kubectl -n ceph get jobs | grep ceph-rbd-pool
ceph-rbd-pool                   0/1           9m23s      9m23s
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$ kubectl -n ceph get pods | grep osd
ceph-osd-default-83945928-4j9lw        0/2     Init:CrashLoopBackOff   33         148m
ceph-osd-default-83945928-fvshr        0/2     Init:CrashLoopBackOff   33         148m
ceph-osd-default-83945928-mbhkr        0/2     Init:CrashLoopBackOff   33         148m
ceph-osd-keyring-generator-r9pw8       0/1     Completed               0          5d20h
ubuntu@device:/opt/openstack-helm$
```

 ##### - Pods=_mariadb-server-_ pending for "_storageclass.storage.k8s.io "general" not found_"
```

    17  openstack       mariadb-server-0                                    0/1     Pending                 0          5d20h   <none>            <none>   <none>           <none>
    18  openstack       mariadb-server-1                                    0/1     Pending                 0          5d20h   <none>            <none>   <none>           <none>
    19  openstack       mariadb-server-2                                    0/1     Pending                 0          5d20h   <none>            <none>   <none>           <none>

ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$ kubectl -n openstack get pods|  grep mariadb-server-
mariadb-server-0                                    0/1     Pending     0          5d20h
mariadb-server-1                                    0/1     Pending     0          5d20h
mariadb-server-2                                    0/1     Pending     0          5d20h
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$ kubectl -n openstack describe pod mariadb-server-1
Name:           mariadb-server-1
Namespace:      openstack
Priority:       0
Node:           <none>
Labels:         application=mariadb
                component=server
                controller-revision-hash=mariadb-server-74cd64956
                release_group=mariadb
                statefulset.kubernetes.io/pod-name=mariadb-server-1
Annotations:    configmap-bin-hash: cb0bb95d4dcade83796f9f4f5bbca8723ada30eb65f55d5d9930fb907c0a0eea
                configmap-etc-hash: 05034f13b89a1ec3945984cc3209a9e1ef30dcb6f903cdd3783f37d0c2ddb991
                mariadb-dbadmin-password-hash: 1e6d76f39aff6238267d343440e15ac49c0922ee715c5767bb3b2ef5efbb5394
                mariadb-sst-password-hash: 1e6d76f39aff6238267d343440e15ac49c0922ee715c5767bb3b2ef5efbb5394
                openstackhelm.openstack.org/release_uuid:
Status:         Pending
IP:
IPs:            <none>
Controlled By:  StatefulSet/mariadb-server
Init Containers:
  init:
    Image:      quay.io/airshipit/kubernetes-entrypoint:v1.0.0
    Port:       <none>
    Host Port:  <none>
    Command:
      kubernetes-entrypoint
    Environment:
      POD_NAME:                    mariadb-server-1 (v1:metadata.name)
      NAMESPACE:                   openstack (v1:metadata.namespace)
      INTERFACE_NAME:              eth0
      PATH:                        /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/
      DEPENDENCY_SERVICE:
      DEPENDENCY_DAEMONSET:
      DEPENDENCY_CONTAINER:
      DEPENDENCY_POD_JSON:
      DEPENDENCY_CUSTOM_RESOURCE:
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from mariadb-mariadb-token-zbrgl (ro)
  mariadb-perms:
    Image:      docker.io/openstackhelm/mariadb:latest-ubuntu_xenial
    Port:       <none>
    Host Port:  <none>
    Command:
      chown
      -R
      mysql:mysql
      /var/lib/mysql
    Environment:  <none>
    Mounts:
      /tmp from pod-tmp (rw)
      /var/lib/mysql from mysql-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from mariadb-mariadb-token-zbrgl (ro)
Containers:
  mariadb:
    Image:       docker.io/openstackhelm/mariadb:latest-ubuntu_xenial
    Ports:       3306/TCP, 4567/TCP
    Host Ports:  0/TCP, 0/TCP
    Command:
      /tmp/start.py
    Readiness:  exec [/tmp/readiness.sh] delay=30s timeout=15s period=30s #success=1 #failure=3
    Environment:
      POD_NAMESPACE:           openstack (v1:metadata.namespace)
      MARIADB_REPLICAS:        3
      POD_NAME_PREFIX:         mariadb-server
      DISCOVERY_DOMAIN:        mariadb-discovery.openstack.svc.cluster.local
      DIRECT_SVC_NAME:         mariadb-server
      WSREP_PORT:              4567
      STATE_CONFIGMAP:         mariadb-mariadb-state
      MYSQL_DBADMIN_USERNAME:  root
      MYSQL_DBADMIN_PASSWORD:  <set to the key 'MYSQL_DBADMIN_PASSWORD' in secret 'mariadb-dbadmin-password'>  Optional: false
      MYSQL_DBSST_USERNAME:    sst
      MYSQL_DBSST_PASSWORD:    <set to the key 'MYSQL_DBSST_PASSWORD' in secret 'mariadb-dbsst-password'>  Optional: false
      MYSQL_DBAUDIT_USERNAME:  audit
      MYSQL_DBAUDIT_PASSWORD:  <set to the key 'MYSQL_DBAUDIT_PASSWORD' in secret 'mariadb-dbaudit-password'>  Optional: false
    Mounts:
      /etc/mysql/admin_user.cnf from mariadb-secrets (ro,path="admin_user.cnf")
      /etc/mysql/conf.d from mycnfd (rw)
      /etc/mysql/conf.d/00-base.cnf from mariadb-etc (ro,path="00-base.cnf")
      /etc/mysql/conf.d/99-force.cnf from mariadb-etc (ro,path="99-force.cnf")
      /etc/mysql/my.cnf from mariadb-etc (ro,path="my.cnf")
      /tmp from pod-tmp (rw)
      /tmp/readiness.sh from mariadb-bin (ro,path="readiness.sh")
      /tmp/start.py from mariadb-bin (ro,path="start.py")
      /tmp/stop.sh from mariadb-bin (ro,path="stop.sh")
      /var/lib/mysql from mysql-data (rw)
      /var/run/mysqld from var-run (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from mariadb-mariadb-token-zbrgl (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  mysql-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mysql-data-mariadb-server-1
    ReadOnly:   false
  pod-tmp:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  mycnfd:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  var-run:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  mariadb-bin:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      mariadb-bin
    Optional:  false
  mariadb-etc:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      mariadb-etc
    Optional:  false
  mariadb-secrets:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  mariadb-secrets
    Optional:    false
  mariadb-mariadb-token-zbrgl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  mariadb-mariadb-token-zbrgl
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  openstack-control-plane=enabled
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                   From               Message
  ----     ------            ----                  ----               -------
  Warning  FailedScheduling  4m55s (x190 over 4h)  default-scheduler  0/8 nodes are available: 8 pod has unbound immediate PersistentVolumeClaims.
ubuntu@device:/opt/openstack-helm$
ubuntu@device:/opt/openstack-helm$ kubectl -n openstack describe pvc mysql-data-mariadb-server-1
Name:          mysql-data-mariadb-server-1
Namespace:     openstack
StorageClass:  general
Status:        Pending
Volume:
Labels:        application=mariadb
               component=server
               release_group=mariadb
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Mounted By:    mariadb-server-1
Events:
  Type     Reason              Age                    From                         Message
  ----     ------              ----                   ----                         -------
  Warning  ProvisioningFailed  4m6s (x961 over 4h4m)  persistentvolume-controller  storageclass.storage.k8s.io "general" not found
ubuntu@device:/opt/openstack-helm$
```


 ##### - Checking Ceph deployment status, per [OSH Persistent-Storage guide](https://docs.openstack.org/openstack-helm/latest/troubleshooting/persistent-storage.html)
 ubuntu@device:/opt/openstack-helm$ ` MON_POD=$(kubectl get --no-headers pods -n=ceph -l="application=ceph,component=mon" | awk '{ print $1; exit }') ` <br/>
ubuntu@device:/opt/openstack-helm$ ` kubectl exec -n ceph ${MON_POD} -- ceph -s `
```
  cluster:
    id:     dfecae40-3104-4bda-9cbe-a0fe46d9fc6e
    health: HEALTH_WARN
            3 osds down
            1 host (3 osds) down
            1 root (3 osds) down
            Reduced data availability: 93 pgs inactive

  services:
    mon: 3 daemons, quorum rkew6,rkew5,rkew4 (age 4h)
    mgr: device(active, starting, since 1.6687s)
    osd: 3 osds: 0 up (since 3h), 3 in (since 5d)

  data:
    pools:   18 pools, 93 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     100.000% pgs unknown
             93 unknown

ubuntu@device:/opt/openstack-helm$
```