  # Project: Tenant_boburciu_SC-Robot-Automation-tries <=> C:\Users\bogdan.burciu\Desktop\svc_port_tuple_sameMGMT-Robot.yml

(pyenv-openstack) [prod] root@m1-foremansible:ansible# ` openstack project list | grep boburciu `
```
| 0a2d675d5f344f96acd5cf06fe05e65f | Tenant_boburciu_ROBOT_SC                                         |
| 1fc90fb281fe4bc8ae2aae25b36bc421 | Tenant_boburciu                                                  |
| 572fcf479c124e7a9d7db3d55711c8ca | Tenant_boburciu_proj_byRobot_Introspect                          |
| ba61416237c543f7ae745d1db0c07846 | Tenant_boburciu_SC-Robot-Automation-tries                        |
```
(pyenv-openstack) [prod] root@m1-foremansible:ansible # ` openstack server list --project=ba61416237c543f7ae745d1db0c07846 `
```
+--------------------------------------+---------------------------------+--------+------------------------------------------------------------------------------------------------------------+-------------------------------------+--------------+
| ID                                   | Name                            | Status | Networks                                                                                                   | Image                               | Flavor       |
+--------------------------------------+---------------------------------+--------+------------------------------------------------------------------------------------------------------------+-------------------------------------+--------------+
| f0adb7c7-7206-4086-afba-f4f6098a5f9d | SC-boburciu-Service-Instance-VM | ACTIVE | MGMT_vSRX_RobotFramework=200.200.200.76; SC-boburciu-Left-VN=100.12.12.4; SC-boburciu-Right-VN=100.34.34.4 | vsrx3_dpdk_testing_service-instance | GEN-FLV-VSRX |
| 324b95b3-96e1-45be-aec9-e68161bbf85d | SC-boburciu-Left-VM             | ACTIVE | SC-boburciu-Left-VN=100.12.12.3                                                                            | centos_testing                      | centos_flav  |
| b1ce64ad-fb4b-4655-8ceb-15d53b8cf8d5 | SC-boburciu-Right-VM            | ACTIVE | SC-boburciu-Right-VN=100.34.34.3                                                                           | centos_testing                      | centos_flav  |
+--------------------------------------+---------------------------------+--------+------------------------------------------------------------------------------------------------------------+-------------------------------------+--------------+
(pyenv-openstack) [prod] root@m1-foremansible:ansible#
```
(pyenv-openstack) [prod] root@m1-foremansible:ansible # ` openstack server show f0adb7c7-7206-4086-afba-f4f6098a5f9d `
```
+-------------------------------------+------------------------------------------------------------------------------------------------------------+
| Field                               | Value                                                                                                      |
+-------------------------------------+------------------------------------------------------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                                                                     |
| OS-EXT-AZ:availability_zone         | AZ-kernel4-manual-tests                                                                                    |
| OS-EXT-SRV-ATTR:host                | i1-hyp13-001.mgmt.oiaas                                                                                    |
| OS-EXT-SRV-ATTR:hypervisor_hostname | i1-hyp13-001.mgmt.oiaas                                                                                    |
:                                                                                                       |
| addresses                           | MGMT_vSRX_RobotFramework=200.200.200.76; SC-boburciu-Left-VN=100.12.12.4; SC-boburciu-Right-VN=100.34.34.4 |
:                                                                                    |
| flavor                              | GEN-FLV-VSRX (18fde8e6-ee53-460a-b8af-0f64f578b9b3)                                                        |
:                                                                   |
| image                               | vsrx3_dpdk_testing_service-instance (6cef0a5b-9990-4e2c-9fdf-c975288675f4)     
:
(pyenv-openstack) [prod] root@m1-foremansible:ansible#
```

  # connecting to vRouter/compute host of SC-boburciu-Left-VM w SC-boburciu-Left-VN=100.12.12.3   
```  
[prod] root@m1-foremansible:~ # ssh i1-hyp24-001.mgmt.oiaas
Last login: Mon Sep  7 08:39:00 2020 from 192.168.201.133
```
[prod] root@i1-hyp24-001:~ # ` mc_contrail vif --list | grep -C3 100.12.12.3 `
```
            TX port   packets:2899 errors:9 syscalls:2899

vif0/118    PMD: tap81b8b1a0-82 NH: 56
            Type:Virtual HWaddr:00:00:5e:00:01:00 IPaddr:100.12.12.3
            Vrf:112: Mcast Vrf:112 Flags:PL3L2DEr QOS:-1 Ref:24
            RX queue errors to lcore 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
            RX packets:0  bytes:0 errors:0
[prod] root@i1-hyp24-001:~#
[prod] root@i1-hyp24-001:~#
```

  # lookup routing table for destinations of SC-boburciu-Right-VN
[prod] root@i1-hyp24-001:~ # ` mc_contrail rt --dump 112 --family inet | head `
```
Flags: L=Label Valid, P=Proxy ARP, T=Trap ARP, F=Flood ARP
vRouter inet4 routing table 0/112/unicast
Destination           PPL        Flags        Label         Nexthop    Stitched MAC(Index)
0.1.0.0/16              0                       -              0        -
:
[prod] root@i1-hyp24-001:~#
```
[prod] root@i1-hyp24-001:~ # ` mc_contrail rt --dump 112 --family inet | grep "100.34.34.1/32" `
100.34.34.1/32          0                       -              0:        -
[prod] root@i1-hyp24-001:~ # ` mc_contrail rt --dump 112 --family inet | grep "100.34.34.2/32" `
100.34.34.2/32          0                       -              0:        -
[prod] root@i1-hyp24-001:~ # ` mc_contrail rt --dump 112 --family inet | grep "100.34.34.3/32" `
100.34.34.3/32         32           LP         65             29:        -
[prod] root@i1-hyp24-001:~ #

  # we observed the MPLS label for forwarding from VRF:112 to destination VRF of 100.34.34.3/32 is Label:65 and the Nexthop:29
[prod] root@i1-hyp24-001:~ # ` mc_contrail rt --dump 112 --family inet | grep "100.34.34.4/32" `
100.34.34.4/32         32           LP         65             29:        -
[prod] root@i1-hyp24-001:~#
[prod] root@i1-hyp24-001:~#
[prod] root@i1-hyp24-001:~ # ` mc_contrail  nh --list | grep "Id:0 " `
```
Id:0          Type:Drop           Fmly: AF_INET  Rid:0  Ref_cnt:1018899    Vrf:0
You have mail in /var/spool/mail/root
```
[prod] root@i1-hyp24-001:~ # ` mc_contrail  nh --list | grep "Id:29 " `
```
Id:29         Type:Tunnel         Fmly: AF_INET  Rid:0  Ref_cnt:23         Vrf:0
[prod] root@i1-hyp24-001:~# 
```

  # in order to know how the packet will be forwarded we have to check the NextHop used for this route
[prod] root@i1-hyp24-001:~ # ` mc_contrail nh --get 29 `
```
Id:29         Type:Tunnel         Fmly: AF_INET  Rid:0  Ref_cnt:23         Vrf:0
              Flags:Valid, MPLSoUDP, Etree Root,
              Oif:0: Len:14 Data:14 02 ec 87 b6 51 48 df 37 29 6e e5 08 00
|\              **Sip:192.168.25.17 Dip:192.168.25.15**
[prod] root@i1-hyp24-001:~#
```
  # in the Oif field of above output we have the interface where the packet will be sent to the other compute node
  # to get the details about this interface:
[prod] root@i1-hyp24-001:~ # ` vif --get 0 `
```
Vrouter Interface Table

Flags: P=Policy, X=Cross Connect, S=Service Chain, Mr=Receive Mirror
       Mt=Transmit Mirror, Tc=Transmit Checksum Offload, L3=Layer 3, L2=Layer 2
       D=DHCP, Vp=Vhost Physical, Pr=Promiscuous, Vnt=Native Vlan Tagged
       Mnp=No MAC Proxy, Dpdk=DPDK PMD Interface, Rfl=Receive Filtering Offload, Mon=Interface is Monitored
       Uuf=Unknown Unicast Flood, Vof=VLAN insert/strip offload, Df=Drop New Flows, L=MAC Learning Enabled
       Proxy=MAC Requests Proxied Always, Er=Etree Root, Mn=Mirror without Vlan Tag, Ig=Igmp Trap Enabled

vif0/0      PCI: 0000:00:00.0 (Speed 20000, Duplex 0) NH: 4
|\           Type:Physical HWaddr:48:df:37:29:6e:e5 IPaddr:0.0.0.0
            Vrf:0 Mcast Vrf:65535 Flags:TcL3L2VpEr QOS:-1 Ref:57
            RX device packets:822870175  bytes:696739008832 errors:0
            RX port   packets:411156719 errors:0
            RX queue  packets:54877 errors:0
            RX queue errors to lcore 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
            Fabric Interface: eth_bond_br-mesh  Status: UP  Driver: net_bonding
            Slave Interface(0): 0000:37:00.1  Status: UP  Driver: net_ixgbe
            Slave Interface(1): 0000:5c:00.1  Status: UP  Driver: net_ixgbe
            RX packets:411156719  bytes:348334986125 errors:0
            TX packets:382626959  bytes:63377612486 errors:0
            Drops:1799208
            TX port   packets:384081779 errors:7
            TX device packets:768737409  bytes:124343627522 errors:0

[prod] root@i1-hyp24-001:~#
[prod] root@i1-hyp24-001:~#
[prod] root@i1-hyp24-001:~ # ip addr sh dev vhost0
20: vhost0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc pfifo_fast state UNKNOWN group default qlen 1000
|\    link/ether 48:df:37:29:6e:e5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.25.17/24: brd 192.168.25.255 scope global vhost0
       valid_lft forever preferred_lft forever
    inet6 fe80::4adf:37ff:fe29:6ee5/64 scope link
       valid_lft forever preferred_lft forever
[prod] root@i1-hyp24-001:~ # ip addr sh dev br-mgmt
10: br-mgmt: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc noqueue state UP group default qlen 1000
    link/ether 48:df:37:29:6e:e4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.22.20/24: brd 192.168.22.255 scope global br-mgmt
       valid_lft forever preferred_lft forever
    inet6 fe80::4adf:37ff:fe29:6ee4/64 scope link
       valid_lft forever preferred_lft forever
[prod] root@i1-hyp24-001:~#
```

  # the br-mgmt is the EXEC-MGMT network address of the nova-compute pod, placed on a EXEC zone server added as K8s worker, 
  # using host-networking:     
(pyenv-openstack) [prod] root@m1-foremansible:ansible # ` kubectl -n openstack get pods -o wide | grep 192.168.22.20 `
```
envoy-i1-hyp24-001-egress-c5f858c5d-dt494                    1/1     Running     0          59d     192.168.22.20    i1-hyp24-001.mgmt.oiaas   <none>           <none>
libvirt-dpdk-libvirt-km7tc                                   1/1     Running     0          59d     192.168.22.20    i1-hyp24-001.mgmt.oiaas   <none>           <none>
|\ nova-compute-dpdk-compute-wt427                              1/1     Running     0          59d     192.168.22.20    i1-hyp24-001.mgmt.oiaas   <none>           <none>
(pyenv-openstack) [prod] root@m1-foremansible:ansible#
```

  # connecting to vRouter/compute host of SC-boburciu-Service-Instance-VM, a vSRX VNF:
```  
[prod] root@m1-foremansible:~ # ssh i1-hyp13-001.mgmt.oiaas
Last login: Wed Sep 16 13:13:54 2020 from 192.168.201.133
[prod] root@i1-hyp13-001:~#
[prod] root@i1-hyp13-001:~ # ip addr sh dev vhost0
1868: vhost0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 14:02:ec:87:b6:51 brd ff:ff:ff:ff:ff:ff
    inet 192.168.25.15/24: brd 192.168.25.255 scope global vhost0
       valid_lft forever preferred_lft forever
    inet6 fe80::1602:ecff:fe87:b651/64 scope link
       valid_lft forever preferred_lft forever
[prod] root@i1-hyp13-001:~#
```

 # we know the MPLS label used by the egress vRouter was Label:65, so we can check the action entry on current vRouter:
[prod] root@i1-hyp13-001:~ # ` mc_contrail mpls --get 65 `
```
MPLS Input Label Map

   Label    NextHop
-------------------
      65       110:
[prod] root@i1-hyp13-001:~#
```
[prod] root@i1-hyp13-001:~ # ` mc_contrail nh --get 110 `
```
Id:110        Type:Encap          Fmly: AF_INET  Rid:0  Ref_cnt:13         Vrf:12
              Flags:Valid, Policy, Etree Root,
              EncapFmly:0806 Oif:9: Len:14
              Encap Data: 02 f5 71 51 39 f1 00 00 5e 00 01 00 08 00

```

  # we get the outgoing interface Oif:9 for this Nexthop
[prod] root@i1-hyp13-001:~ # ` mc_contrail vif --get 9 `
```
Vrouter Interface Table

Flags: P=Policy, X=Cross Connect, S=Service Chain, Mr=Receive Mirror
       Mt=Transmit Mirror, Tc=Transmit Checksum Offload, L3=Layer 3, L2=Layer 2
       D=DHCP, Vp=Vhost Physical, Pr=Promiscuous, Vnt=Native Vlan Tagged
       Mnp=No MAC Proxy, Dpdk=DPDK PMD Interface, Rfl=Receive Filtering Offload, Mon=Interface is Monitored
       Uuf=Unknown Unicast Flood, Vof=VLAN insert/strip offload, Df=Drop New Flows, L=MAC Learning Enabled
       Proxy=MAC Requests Proxied Always, Er=Etree Root, Mn=Mirror without Vlan Tag, Ig=Igmp Trap Enabled

vif0/9      OS: tapf5715139-f1 NH: 110
            Type:Virtual HWaddr:00:00:5e:00:01:00 IPaddr:100.12.12.4:
            Vrf:12 Mcast Vrf:12 Flags:PL3L2DEr QOS:-1 Ref:6
            RX packets:1073  bytes:47116 errors:0
            TX packets:2450  bytes:104037 errors:0
            ISID: 0 Bmac: 02:f5:71:51:39:f1
            Drops:6

You have mail in /var/spool/mail/root
[prod] root@i1-hyp13-001:~#
```

  # the packet sourced from SC-boburciu-Left-VN=100.12.12.0/24, VRF:112 will get sent to 100.12.12.4 in VRF:12.


  # Contents of svc_port_tuple_sameMGMT-Robot.yml for reproducibility:
```   
heat_template_version: 2015-04-30

parameters:
  left_vn:
    type: string
    description: Name of left network to be created
    default: 'SC-boburciu-Left-VN'
  right_vn:
    type: string
    description: Name of right network to be created
    default: 'SC-boburciu-Right-VN'    
  left_vn_fqdn:
    type: string
    description: FQ Name of the left network
    default: 'default-domain:Tenant_boburciu_SC-Robot-Automation-tries:SC-boburciu-Left-VN'    
  right_vn_fqdn:
    type: string
    description: FQ Name of the right network
    default: 'default-domain:Tenant_boburciu_SC-Robot-Automation-tries:SC-boburciu-Right-VN'    
  
  domain:
    type: string
    description: Name of the Domain
    default: 'default-domain'
  service_template_name:
    type: string
    label: Service template fq name
    description: Service template for port tuple
    default: 'SC-boburciu-Service-Template'    
  service_template_properties_version:
    type: string
    description: Indicates service version
    default: 2
  service_template_properties_service_mode:
    type: string
    description: service mode
    default: in-network
  service_template_properties_service_type:
    type: string
    description: service type
    default: firewall
  service_template_properties_interface_type_service_interface_type_1:
    type: string
    description: service_interface_type for the ServiceTemplate
    default: management
  service_template_properties_interface_type_service_interface_type_2:
    type: string
    description: service_interface_type for the ServiceTemplate
    default: left
  service_template_properties_interface_type_service_interface_type_3:
    type: string
    description: service_interface_type for the ServiceTemplate
    default: right
  service_template_properties_ordered_interfaces:
    type: string
    description: Indicates service interfaces are ordered
    default: true    

  service_instance_name:
    type: string
    label: Service instance name
    description: Service instance for port tuple
    default: 'SC-boburciu-Service-Instance' 
  service_instance_fq_name:
    type: string
    label: Service instance fq name
    description: Service instance FQDN for port tuple
    default: 'default-domain:Tenant_boburciu_SC-Robot-Automation-tries:SC-boburciu-Service-Instance' 
  
  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_2:
    type: string
    description: subnet prefix for left network
    default: '100.12.12.0'        
  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len_2:
    type: string
    description: subnet prefix len for left network
    default: 24       
  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_3:
    type: string
    description: subnet prefix for right network
    default: '100.34.34.0'        
  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len_3:
    type: string
    description: subnet prefix len for right network
    default: 24
  network_ipam_refs_data_ipam_subnets_addr_from_start_true:
    type: boolean
    description: Address allocation from start of the pool
    default: true    
  
  svm_name: 
    type: string
    description: Name of the SVM
    default: 'SC-boburciu-Service-Instance-VM'        
  svm_flavor: 
    type: string
    description: Flavor of the SVM
    default: GEN-FLV-VSRX    
  svm_image_name: 
    type: string
    description: Name of the SVM image
    default: 	vsrx3_dpdk_testing_service-instance     
    
  flavor: 
    type: string
    description: Flavor of the end VMs
    default: centos_flav
  image: 
    type: string
    description: Name of the end VM image
    default: centos_testing       
  left_vm_name:
    type: string
    description: Name of the left VM
    default: 'SC-boburciu-Left-VM'    
  right_vm_name: 
    type: string
    description: Name of the right VM
    default: 'SC-boburciu-Right-VM'        
  pt_name:
    type: string
    description: Name of the port tuple
    default: 'SC-boburciu-Service-Instance-VM-port-tuple'     
 
  policy_name:
    type: string
    description: Name of the Policy
    default: 'SC-boburciu-Net-Policy' 
  policy_fq_name:
    type: string
    description: FQDN of the Policy
    default: 'default-domain:Tenant_boburciu_SC-Robot-Automation-tries:SC-boburciu-Net-Policy'    
  simple_action:
    type: string
    description: Pass or Deny; lowercase
    default: 'pass'       
  protocol:
    type: string
    description: Protocol
    default: 'any'    
  src_port_end:
    type: number
    description: End of the Source Port Range
    default: 4095       
  src_port_start:
    type: number
    description: Start of the Source Port Range
    default: 0          
  direction:
    type: string
    description: Direction of the Policy
    default: '<>'           
  dst_port_end:
    type: number
    description: End of the Destination Port Range
    default: 4095    
  dst_port_start:
    type: number
    description: Start of the Destination Port Range
    default: 0   

resources:
  template_NetworkIpam_2:
    type: OS::ContrailV2::NetworkIpam
    properties:
      name: { get_param: left_vn }

  template_NetworkIpam_3:
    type: OS::ContrailV2::NetworkIpam
    properties:
      name: { get_param: right_vn }

  template_VirtualNetwork_2:
    type: OS::ContrailV2::VirtualNetwork
    depends_on: [ template_NetworkIpam_2, template_NetworkPolicy ]
    properties:
      name: { get_param: left_vn }
      network_ipam_refs: [{ get_resource: template_NetworkIpam_2 }]
      network_ipam_refs_data:
        [{
          network_ipam_refs_data_ipam_subnets:
            [{
              network_ipam_refs_data_ipam_subnets_subnet:
                {
                  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix: { get_param: network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_2 },
                  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len: { get_param: network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len_2 },
                },
               network_ipam_refs_data_ipam_subnets_addr_from_start: { get_param: network_ipam_refs_data_ipam_subnets_addr_from_start_true },
 	     }]
	 }]
      network_policy_refs: [{ get_param: policy_fq_name }]
      network_policy_refs_data:
        [{
          network_policy_refs_data_sequence:
            {
              network_policy_refs_data_sequence_major: 0,
              network_policy_refs_data_sequence_minor: 0,
            },
        }]
        
  template_VirtualNetwork_3:
    type: OS::ContrailV2::VirtualNetwork
    depends_on: [ template_NetworkIpam_3, template_NetworkPolicy ]
    properties:
      name: { get_param: right_vn }
      network_ipam_refs: [{ get_resource: template_NetworkIpam_3 }]
      network_ipam_refs_data:
        [{
          network_ipam_refs_data_ipam_subnets:
            [{
              network_ipam_refs_data_ipam_subnets_subnet:
                {
                  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix: { get_param: network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_3 },
                  network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len: { get_param: network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len_3 },
                },
	      network_ipam_refs_data_ipam_subnets_addr_from_start: { get_param: network_ipam_refs_data_ipam_subnets_addr_from_start_true },
            }]
         }]
      network_policy_refs: [{ get_param: policy_fq_name }]
      network_policy_refs_data:
        [{
          network_policy_refs_data_sequence:
            {
              network_policy_refs_data_sequence_major: 0,
              network_policy_refs_data_sequence_minor: 0,
            },
        }]

  template_ServiceTemplate:
    type: OS::ContrailV2::ServiceTemplate
    properties:
      name: { get_param: service_template_name }
      service_template_properties:
        {
          service_template_properties_version: { get_param: service_template_properties_version },
          service_template_properties_service_mode: { get_param: service_template_properties_service_mode },
          service_template_properties_service_type: { get_param: service_template_properties_service_type },
          service_template_properties_interface_type:
            [
              {
              service_template_properties_interface_type_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_1 },
              },
              {
              service_template_properties_interface_type_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_2 },
              },
              {
              service_template_properties_interface_type_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_3 },
              }
            ],
        }
      domain: { get_param: domain }

  template_ServiceInstance:
    type: OS::ContrailV2::ServiceInstance
    depends_on: [ template_ServiceTemplate ]
    properties:
      name: { get_param: service_instance_name }
      service_instance_properties:
        {
          service_instance_properties_interface_list:
            [
              {
                service_instance_properties_interface_list_virtual_network: 'default-domain:admin:MGMT_vSRX_RobotFramework'
              },
              {
              service_instance_properties_interface_list_virtual_network:
                {
                   list_join: [':', { get_attr: [ template_VirtualNetwork_2, fq_name ] } ]
                },
              },
              {
              service_instance_properties_interface_list_virtual_network:
                {
                   list_join: [':', { get_attr: [ template_VirtualNetwork_3, fq_name ] } ]
                },
              }
            ],
        }
      service_template_refs: [{ get_resource: template_ServiceTemplate }]

  template_PortTuple:
    type: OS::ContrailV2::PortTuple
    depends_on: [ template_ServiceInstance ]
    properties:
      name: { get_param: pt_name }
      service_instance: { list_join: [':', { get_attr: [ template_ServiceInstance, fq_name ] } ] }

  template_VirtualMachineInterface_1:
    type: OS::ContrailV2::VirtualMachineInterface
    depends_on: [ template_PortTuple ]
    properties:
      virtual_machine_interface_properties:
        {
          virtual_machine_interface_properties_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_1 },
        }
      port_tuple_refs: [{ get_resource: template_PortTuple }]
      virtual_network_refs: [ 'default-domain:admin:MGMT_vSRX_RobotFramework' ]
  
  template_VirtualMachineInterface_2:
    type: OS::ContrailV2::VirtualMachineInterface
    depends_on: [ template_PortTuple ]
    properties:
      virtual_machine_interface_properties:
        {
          virtual_machine_interface_properties_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_2 },
        }
      port_tuple_refs: [{ get_resource: template_PortTuple }]
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_2, fq_name ] } ] }]

  template_VirtualMachineInterface_3:
    type: OS::ContrailV2::VirtualMachineInterface
    depends_on: [ template_PortTuple ]
    properties:
      virtual_machine_interface_properties:
        {
          virtual_machine_interface_properties_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_3 },
        }
      port_tuple_refs: [{ get_resource: template_PortTuple }]
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_3, fq_name ] } ] }]

  template_InstanceIp_1:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_1 ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_1 }]
      virtual_network_refs: [ 'default-domain:admin:MGMT_vSRX_RobotFramework' ]

  template_InstanceIp_2:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_2, template_VirtualNetwork_2 ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_2 }]
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_2, fq_name ] } ] }]

  template_InstanceIp_3:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_3, template_VirtualNetwork_3 ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_3 }]
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_3, fq_name ] } ] }]

  instance:
    type: OS::Nova::Server
    depends_on: [ template_InstanceIp_1, template_InstanceIp_2, template_InstanceIp_3 ]
    properties:
      name: {get_param: svm_name }
      image: { get_param:  svm_image_name }
      flavor: { get_param: svm_flavor }
      networks:
        - port: { get_resource: template_VirtualMachineInterface_1 }
        - port: { get_resource: template_VirtualMachineInterface_2 }
        - port: { get_resource: template_VirtualMachineInterface_3 }

  template_InstanceIp_1_SC-Left-VM:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_1_SC-Left-VM ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_1_SC-Left-VM }]
      virtual_network_refs: [ 'default-domain:admin:MGMT_vSRX_RobotFramework' ]
      #instance_ip_address: '200.200.200.81'      

  template_VirtualMachineInterface_1_SC-Left-VM:
    type: OS::ContrailV2::VirtualMachineInterface
    properties:
      name: 'SC-boburciu-MGMT-VMI-SC-Left-VM'
      virtual_machine_interface_properties:
        {
          virtual_machine_interface_properties_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_1 },
        }
      virtual_network_refs: [ 'default-domain:admin:MGMT_vSRX_RobotFramework' ]    

  template_InstanceIp_2_SC-Left-VM:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_2_SC-Left-VM, template_VirtualNetwork_2 ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_2_SC-Left-VM }]
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_2, fq_name ] } ] }]

  template_VirtualMachineInterface_2_SC-Left-VM:
    type: OS::ContrailV2::VirtualMachineInterface
    properties:
      name: { get_param: left_vm_name }
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_2, fq_name ] } ] }]
  
  template_leftVM:
    type: OS::Nova::Server
    depends_on: [ template_InstanceIp_1_SC-Left-VM, template_InstanceIp_2_SC-Left-VM ]
    properties:
      name: {get_param: left_vm_name }
      image: { get_param:  image }
      flavor: { get_param: flavor }
      networks:
        # - port: { get_resource: template_VirtualMachineInterface_1_SC-Left-VM }      
        - port: { get_resource: template_VirtualMachineInterface_2_SC-Left-VM }

  template_InstanceIp_1_SC-Right-VM:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_1_SC-Right-VM ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_1_SC-Right-VM }]
      virtual_network_refs: [ 'default-domain:admin:MGMT_vSRX_RobotFramework' ]
      #instance_ip_address: '200.200.200.82'      

  template_VirtualMachineInterface_1_SC-Right-VM:
    type: OS::ContrailV2::VirtualMachineInterface
    properties:
      name: 'SC-boburciu-MGMT-VMI-SC-Right-VM'
      virtual_machine_interface_properties:
        {
          virtual_machine_interface_properties_service_interface_type: { get_param: service_template_properties_interface_type_service_interface_type_1 },
        }
      virtual_network_refs: [ 'default-domain:admin:MGMT_vSRX_RobotFramework' ]    

  template_InstanceIp_3_SC-Right-VM:
    type: OS::ContrailV2::InstanceIp
    depends_on: [ template_VirtualMachineInterface_3_SC-Right-VM, template_VirtualNetwork_3 ]
    properties:
      #config_root: "config-root"
      virtual_machine_interface_refs: [{ get_resource: template_VirtualMachineInterface_3_SC-Right-VM }]
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_3, fq_name ] } ] }]

  template_VirtualMachineInterface_3_SC-Right-VM:
    type: OS::ContrailV2::VirtualMachineInterface
    properties:
      name: { get_param: right_vm_name }
      virtual_network_refs: [{ list_join: [':', { get_attr: [ template_VirtualNetwork_3, fq_name ] } ] }]
  
  template_rightVM:
    type: OS::Nova::Server
    depends_on: [ template_InstanceIp_1_SC-Right-VM, template_InstanceIp_3_SC-Right-VM ]
    properties:
      name: {get_param: right_vm_name }
      image: { get_param:  image }
      flavor: { get_param: flavor }
      networks:
        # - port: { get_resource: template_VirtualMachineInterface_1_SC-Left-VM }      
        - port: { get_resource: template_VirtualMachineInterface_3_SC-Right-VM }

  template_NetworkPolicy:
    type: OS::ContrailV2::NetworkPolicy
    properties:
      name: { get_param: policy_name }
      network_policy_entries: { network_policy_entries_policy_rule: [{ 
		network_policy_entries_policy_rule_direction: { get_param: direction },
		network_policy_entries_policy_rule_protocol: { get_param: protocol },
		network_policy_entries_policy_rule_src_ports: [{
			network_policy_entries_policy_rule_src_ports_start_port: { get_param: src_port_start },
			network_policy_entries_policy_rule_src_ports_end_port: { get_param: src_port_end }
								}],
 		network_policy_entries_policy_rule_dst_ports: [{
			network_policy_entries_policy_rule_dst_ports_start_port: { get_param: dst_port_start },
			network_policy_entries_policy_rule_dst_ports_end_port: { get_param: dst_port_end }
								}],
		network_policy_entries_policy_rule_dst_addresses: [{
			network_policy_entries_policy_rule_dst_addresses_virtual_network: { get_param: right_vn_fqdn }
								}],
		network_policy_entries_policy_rule_src_addresses: [{
			network_policy_entries_policy_rule_src_addresses_virtual_network: { get_param: left_vn_fqdn }
								}],
		network_policy_entries_policy_rule_action_list: { 
			network_policy_entries_policy_rule_action_list_simple_action: { get_param: simple_action },
			network_policy_entries_policy_rule_action_list_apply_service: 
									[{ get_param: service_instance_fq_name }]
								},
									}]
	} 
```