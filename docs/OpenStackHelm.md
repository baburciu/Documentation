# Managing OpenStacK as Helm
 
## 0. Setup OSH:

 ### - First satisfy prerequisites for OSH on the KVMs (K8s cluster):
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

 ### - OSH uses Helm v2 with Tiller to deploy OpenStack services as containers. There are two parts to Helm: the Helm client (*helm*) and the Helm server (*Tiller*).

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

 ### - Clone the OpenStack-Helm Repos
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

 ### - Deployment
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