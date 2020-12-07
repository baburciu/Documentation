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
` curl -LO https://git.io/get_helm.sh `   
` chmod 700 get_helm.sh `
` ./get_helm.sh `

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
[boburciu@r220 OpenStackHelm_prereq]$ ` sudo git clone https://opendev.org/openstack/ openstack-helm.git /opt/openstack-helm `
```
Cloning into '/opt/openstack-helm'...
remote: Enumerating objects: 34989, done.
remote: Counting objects: 100% (34989/34989), done.
remote: Compressing objects: 100% (11792/11792), done.
remote: Total 34989 (delta 26581), reused 30308 (delta 22584)
Receiving objects: 100% (34989/34989), 7.36 MiB | 3.91 MiB/s, done.
Resolving deltas: 100% (26581/26581), done.
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ sudo ls -lt /opt/openstack-helm
total 80
drwxr-xr-x. 2 root root    56 Dec  7 21:59 zuul.d
drwxr-xr-x. 6 root root    85 Dec  7 21:59 tools
-rw-r--r--. 1 root root   848 Dec  7 21:59 tox.ini
-rw-r--r--. 1 root root   616 Dec  7 21:59 yamllint.conf
-rw-r--r--. 1 root root   583 Dec  7 21:59 yamllint-templates.conf
drwxr-xr-x. 2 root root    46 Dec  7 21:59 tests
drwxr-xr-x. 3 root root    81 Dec  7 21:59 tempest
drwxr-xr-x. 3 root root    81 Dec  7 21:59 senlin
-rw-r--r--. 1 root root   389 Dec  7 21:59 setup.cfg
-rwxr-xr-x. 1 root root   716 Dec  7 21:59 setup.py
drwxr-xr-x. 3 root root    18 Dec  7 21:59 releasenotes
drwxr-xr-x. 3 root root    98 Dec  7 21:59 rally
drwxr-xr-x. 4 root root   104 Dec  7 21:59 placement
drwxr-xr-x. 3 root root    81 Dec  7 21:59 panko
drwxr-xr-x. 3 root root    81 Dec  7 21:59 octavia
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 nova
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 neutron
drwxr-xr-x. 3 root root    81 Dec  7 21:59 mistral
drwxr-xr-x. 3 root root    81 Dec  7 21:59 magnum
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 keystone
drwxr-xr-x. 3 root root    81 Dec  7 21:59 ironic
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 horizon
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 heat
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 glance
drwxr-xr-x. 3 root root    42 Dec  7 21:59 doc
drwxr-xr-x. 3 root root    81 Dec  7 21:59 designate
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 cinder
drwxr-xr-x. 3 root root    81 Dec  7 21:59 ceilometer
-rw-r--r--. 1 root root   138 Dec  7 21:59 bindep.txt
drwxr-xr-x. 4 root root  4096 Dec  7 21:59 barbican
drwxr-xr-x. 3 root root    81 Dec  7 21:59 aodh
-rw-r--r--. 1 root root   659 Dec  7 21:59 CONTRIBUTING.rst
-rw-r--r--. 1 root root 11357 Dec  7 21:59 LICENSE
-rw-r--r--. 1 root root  1570 Dec  7 21:59 Makefile
-rw-r--r--. 1 root root  2142 Dec  7 21:59 README.rst
[boburciu@r220 OpenStackHelm_prereq]$ 
[boburciu@r220 OpenStackHelm_prereq]$ sudo ls -lt /opt/openstack-helm/tools/deployment/multinode/
total 112
-rwxr-xr-x. 1 root root 3054 Dec  7 21:59 900-tempest.sh
-rwxr-xr-x. 1 root root 1935 Dec  7 21:59 kube-node-subnet.sh
lrwxrwxrwx. 1 root root   22 Dec  7 21:59 070-memcached.sh -> ../common/memcached.sh
-rwxr-xr-x. 1 root root 1259 Dec  7 21:59 080-keystone.sh
-rwxr-xr-x. 1 root root 1167 Dec  7 21:59 085-horizon.sh
-rwxr-xr-x. 1 root root 2409 Dec  7 21:59 090-ceph-radosgateway.sh
-rwxr-xr-x. 1 root root 1891 Dec  7 21:59 100-glance.sh
-rwxr-xr-x. 1 root root 1427 Dec  7 21:59 110-cinder.sh
-rwxr-xr-x. 1 root root  954 Dec  7 21:59 120-openvswitch.sh
-rwxr-xr-x. 1 root root  973 Dec  7 21:59 130-libvirt.sh
-rwxr-xr-x. 1 root root 5218 Dec  7 21:59 140-compute-kit.sh
-rwxr-xr-x. 1 root root 1341 Dec  7 21:59 150-heat.sh
-rwxr-xr-x. 1 root root 1214 Dec  7 21:59 160-barbican.sh
-rwxr-xr-x. 1 root root  967 Dec  7 21:59 170-senlin.sh
-rwxr-xr-x. 1 root root 1217 Dec  7 21:59 180-mistral.sh
-rwxr-xr-x. 1 root root  970 Dec  7 21:59 190-magnum.sh
-rwxr-xr-x. 1 root root 1180 Dec  7 21:59 200-congress.sh
-rwxr-xr-x. 1 root root  895 Dec  7 21:59 210-postgresql.sh
-rwxr-xr-x. 1 root root 1002 Dec  7 21:59 220-gnocchi.sh
-rwxr-xr-x. 1 root root  886 Dec  7 21:59 230-mongodb.sh
-rwxr-xr-x. 1 root root  951 Dec  7 21:59 240-panko.sh
-rwxr-xr-x. 1 root root 1050 Dec  7 21:59 250-aodh.sh
-rwxr-xr-x. 1 root root 1054 Dec  7 21:59 260-ceilometer.sh
lrwxrwxrwx. 1 root root   25 Dec  7 21:59 010-setup-client.sh -> ../common/setup-client.sh
-rwxr-xr-x. 1 root root  674 Dec  7 21:59 020-ingress.sh
-rwxr-xr-x. 1 root root 2902 Dec  7 21:59 030-ceph.sh
-rwxr-xr-x. 1 root root 1507 Dec  7 21:59 040-ceph-ns-activate.sh
-rwxr-xr-x. 1 root root 1057 Dec  7 21:59 050-mariadb.sh
-rwxr-xr-x. 1 root root  978 Dec  7 21:59 060-rabbitmq.sh
[boburciu@r220 OpenStackHelm_prereq]$
[boburciu@r220 OpenStackHelm_prereq]$ cd /opt/openstack-helm
[boburciu@r220 openstack-helm]$
[boburciu@r220 openstack-helm]$
```

[boburciu@r220 openstack-helm]$ ` sudo mkdir /opt/openstack-helm-infra `
[sudo] password for boburciu:
[boburciu@r220 openstack-helm]$
[boburciu@r220 openstack-helm]$ ` sudo git clone https://opendev.org/openstack/openstack-helm-infra.git /opt/openstack-helm-infra `
```
Cloning into '/opt/openstack-helm-infra'...
remote: Enumerating objects: 24032, done.
remote: Counting objects: 100% (24032/24032), done.
remote: Compressing objects: 100% (10092/10092), done.
remote: Total 24032 (delta 17238), reused 19357 (delta 13186)
Receiving objects: 100% (24032/24032), 4.35 MiB | 1.02 MiB/s, done.
Resolving deltas: 100% (17238/17238), done.
[boburciu@r220 openstack-helm]$
[boburciu@r220 openstack-helm]$ ls -lt /opt
total 8
drwxr-xr-x. 64 root root 4096 Dec  7 22:30 openstack-helm-infra
drwxr-xr-x. 28 root root 4096 Dec  7 21:59 openstack-helm
drwxr-xr-x.  2 root root    6 Oct 30  2018 rh
[boburciu@r220 openstack-helm]$
```

 ### - Deployment
[boburciu@r220 openstack-helm]$ ` sudo ./tools/deployment/multinode/010-setup-client.sh `
```
:
Successfully installed Babel-2.6.0 PyYAML-3.13 appdirs-1.4.3 certifi-2018.11.29 cffi-1.12.2 chardet-3.0.4 cliff-2.14.1 cmd2-0.8.9 cryptography-2.8 debtcollector-1.21.0 decorator-4.3.2 dogpile.cache-0.7.1 idna-2.8 iso8601-0.1.12 jmespath-0.9.4 jsonpatch-1.23 jsonpointer-2.0 jsonschema-2.6.0 keystoneauth1-3.13.2 msgpack-0.6.1 munch-2.3.2 netaddr-0.7.19 netifaces-0.10.9 openstacksdk-0.27.1 os-service-types-1.6.0 osc-lib-1.12.1 oslo.config-6.8.2 oslo.i18n-3.23.1 oslo.serialization-2.28.2 oslo.utils-3.40.7 pbr-5.1.3 prettytable-0.7.2 pyOpenSSL-19.1.0 pycparser-2.19 pyparsing-2.3.1 pyperclip-1.7.0 python-cinderclient-4.2.2 python-glanceclient-2.16.0 python-heatclient-1.17.1 python-keystoneclient-3.19.1 python-novaclient-13.0.2 python-openstackclient-3.18.1 python-swiftclient-3.7.1 pytz-2018.9 requests-2.21.0 requestsexceptions-1.4.0 rfc3986-1.2.0 simplejson-3.16.0 six-1.15.0 stevedore-1.30.1 urllib3-1.24.1 warlock-1.3.0 wcwidth-0.1.7 wrapt-1.11.1
+ sudo -H mkdir -p /etc/openstack
++ id -un
+ sudo -H chown -R root: /etc/openstack
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
[boburciu@r220 openstack-helm]$
```