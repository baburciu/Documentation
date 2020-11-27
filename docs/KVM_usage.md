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

## Further sections
