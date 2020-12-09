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

 #### -  Doc is incomplete, running _/opt/openstack-helm/tools/deployment/multinode/010-setup-client.sh_ script will not assemble chart and you need to:
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
ubuntu@device:/$  ` cd /opt/openstack-helm-infra `
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
ubuntu@device:/opt/openstack-helm-infra$ ` cd /opt/openstack-helm `
ubuntu@device:/opt/openstack-helm$ `` sudo OSH_DEPLOY_MULTINODE=True ./tools/deployment/component/common/ingress.sh `
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

 ##### -  Transfer the Ansible inventory file from _rkem1_ in order to run Ansible ad-hoc from CentOS host, since all K8s nodes are SSH reachable from this host:
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

 ##### -  Problem:
```
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ ansible all -i ./multinode-inventory_modifiedBurciu.yaml -m shell -a "/opt/openstack-helm/tools/deployment/common/setup-ceph-loopback-device.sh --ceph-osd-data /dev/loop0 --ceph-osd-dbwal /dev/loop1" -vv
ansible 2.9.15
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/boburciu/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Oct 14 2020, 14:45:30) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
Using /etc/ansible/ansible.cfg as config file
META: ran handlers
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew5 should use /usr/bin/python3, but is using /usr/bin/python for
backward compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform
python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more
information. This feature will be removed in version 2.12. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
rkew5 | FAILED | rc=127 >>
/bin/sh: 1: /opt/openstack-helm/tools/deployment/common/setup-ceph-loopback-device.sh: not foundnon-zero return code
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew6 should use /usr/bin/python3, but is using /usr/bin/python for
backward compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform
python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more
information. This feature will be removed in version 2.12. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
rkew6 | FAILED | rc=127 >>
/bin/sh: 1: /opt/openstack-helm/tools/deployment/common/setup-ceph-loopback-device.sh: not foundnon-zero return code
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host rkew4 should use /usr/bin/python3, but is using /usr/bin/python for
backward compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform
python for this host. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more
information. This feature will be removed in version 2.12. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
rkew4 | FAILED | rc=127 >>
/bin/sh: 1: /opt/openstack-helm/tools/deployment/common/setup-ceph-loopback-device.sh: not foundnon-zero return code
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ cat ./setup-ceph-loopback-device.sh                                                     #!/bin/bash
function setup_loopback_devices() {
  osd_data_device="$1"
  osd_wal_db_device="$2"
  namespace=${CEPH_NAMESPACE}
  sudo mkdir -p /var/lib/openstack-helm/$namespace
  sudo truncate -s 10G /var/lib/openstack-helm/$namespace/ceph-osd-data-loopbackfile.img
  sudo truncate -s 8G /var/lib/openstack-helm/$namespace/ceph-osd-db-wal-loopbackfile.img
  sudo losetup $osd_data_device /var/lib/openstack-helm/$namespace/ceph-osd-data-loopbackfile.img
  sudo losetup $osd_wal_db_device /var/lib/openstack-helm/$namespace/ceph-osd-db-wal-loopbackfile.img
  #lets verify the devices
  sudo losetup -a
}

while [[ "$#" > 0 ]]; do case $1 in
  -d|--ceph-osd-data) OSD_DATA_DEVICE="$2"; shift;shift;;
  -w|--ceph-osd-dbwal) OSD_DB_WAL_DEVICE="$2";shift;shift;;
  -v|--verbose) VERBOSE=1;shift;;
  *) echo "Unknown parameter passed: $1"; shift;;
esac; done

# verify params
if [ -z "$OSD_DATA_DEVICE" ]; then
  OSD_DATA_DEVICE=/dev/loop0
  echo "Ceph osd data device is not set so using ${OSD_DATA_DEVICE}"
else
  ceph_osd_disk_name=`basename "$OSD_DATA_DEVICE"`
  if losetup -a|grep $ceph_osd_disk_name; then
     echo "Ceph osd data device is already in use, please double check and correct the device name"
     exit 1
  fi
fi

if [ -z "$OSD_DB_WAL_DEVICE" ]; then
  OSD_DB_WAL_DEVICE=/dev/loop1
  echo "Ceph osd db/wal device is not set so using ${OSD_DB_WAL_DEVICE}"
else
  ceph_dbwal_disk_name=`basename "$OSD_DB_WAL_DEVICE"`
  if losetup -a|grep $ceph_dbwal_disk_name; then
     echo "Ceph osd dbwal device is already in use, please double check and correct the device name"
     exit 1
  fi
fi

: "${CEPH_NAMESPACE:="ceph"}"
# setup loopback devices for ceph osds
setup_loopback_devices $OSD_DATA_DEVICE $OSD_DB_WAL_DEVICE
[boburciu@r220 OpenStackHelm_prereq]$
```