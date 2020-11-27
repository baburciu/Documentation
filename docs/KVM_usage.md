# Managing KVM machines in CentOS 7.2 
 
## 0. Setup notes:
 >  - virt-manager (a GUI tool), which is a API client of **libvirtd** service (same as `virsh` and `virt-install`) is not synced with `virsh <br/>
` virsh uri ` <br/>
 >  - If it returns qemu:///session, but you're using a qemu:///system connection in Virt-Manager, you found the cause <br/>
 >  - To change your connection URI and use qemu:///system as default is to edit your $HOME/.bahsrc and add <br/>
 ` export LIBVIRT_DEFAULT_URI="qemu:///system" `  <br/>
 >  - virt-manager will start with "QEMU/KVM user session" connection <br/>

## I. How to create KVM:
 >  - Commands for [KVM networking](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-managing_guest_virtual_machines_with_virsh-managing_virtual_networks)
 >  - Commands for [interface statistics](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-statlists)

[boburciu@r220 ~]$ ` sudo virt-install --name=ubuntu1804cmd --ram=1024 --vcpus=1 --cdrom=/home/boburciu/Desktop/ISOs/ubuntu-18.04.5-live-server-amd64.iso --os-type=linux --os-variant=ubuntu18.04 --network default --disk path=/BM_VMs/rkeM1cmd.qcow2,size=20 `<br/>
[boburciu@r220 ~]$ <br/>
[boburciu@r220 ~]$ ` sudo virsh list --all `<br/>
 Id    Name                           State<br/>
----------------------------------------------------<br/>
 1     DNSubuntu18.04                 running<br/>
 -     RancherOSm2                    shut off<br/>
 -     rke_master1                    shut off<br/>
 -     rke_master2                    shut off<br/>
 -     rke_worker1                    shut off<br/>
 -     rke_worker2                    shut off<br/>
 -     rke_worker3                    shut off<br/>
 -     rke_worker4                    shut off<br/>
 -     ubuntu20.04M1                  shut off<br/>
<br/>
[boburciu@r220 ~]$ ` sudo virsh domiflist DNSubuntu18.04 `<br/>
Interface  Type       Source     Model       MAC<br/>
-------------------------------------------------------<br/>
vnet0      network    default    virtio      52:54:00:69:8a:08<br/>
<br/>
[boburciu@r220 ~]$<br/>

```
[boburciu@r220 ~]$ # ` ansible rancheros -m command -a "hostnamectl status" --extra-vars "ansible_user=rancher ansible_password=rancher" -v `
Using /etc/ansible/ansible.cfg as config file
[WARNING]: No python interpreters found for host 192.168.122.155 (tried
['/usr/bin/python', 'python3.7', 'python3.6', 'python3.5', 'python2.7',
'python2.6', '/usr/libexec/platform-python', '/usr/bin/python3', 'python'])
192.168.122.155 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "module_stderr": "Shared connection to 192.168.122.155 closed.\r\n", 
    "module_stdout": "/bin/sh: /usr/bin/python: No such file or directory\r\n", 
    "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", 
    "rc": 127
}
[boburciu@r220 ~]$ 
```

## Further sections
