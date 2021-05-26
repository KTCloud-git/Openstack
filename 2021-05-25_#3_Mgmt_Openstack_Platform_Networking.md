

# Managing Red Hat OpenStack Platform Networking 

0. Netron 

* OVN과 OVS를 관리하는 역할을 수행함. 즉, Routing 및 Switching 역할을 수행함. 기존에 iptables, Namespace가 Kernel에서 수행하던 것을 User 영역에서 수행

* SDN(소프트웨어 정의 네트워킹) 제공. ml2, ml3 제공. SDN은 Kernel에서 제공하는 기능들을 User 영역에서 제공한다고 생각하면 됨. 

* 기본 구조

```
network(namespace)
\_ subnet(route, DHCP) 
\_ dnsmasq + namespace + tab
\_ security group (iptables) 
    \_ OVN(DHCP, Routing)
    \_ OVN(security group(ACL))

PROJECT
    \_ network 
        \_ VLAN (SDN) - VLAN은 여러 Swith의 역할을 나눈다. 즉, L2영역에 여러 Switch를 구축해야하는 것을 VLAN으로 분할한다.
            \_ Subnet 
                \_ DHCP 
                    \_ Route 

        위 일련의 분할 및 Network 체계 구성은 OVN이 담당 
```

1. Openstack HW Network Hierarcy 

```
Openstack HW Hierarcy

* 아래 내용은 표준 구조로, TAP 영역에 OVS를 넣어서 네트워크를 구성할 수 있음. 

* SKT를 TAP을 이용하여 Tenency ip network를 구성하고 있고 버라이즌의 경우는 TAB 이하의 OVS INT-BR을 이용하고 있음.
______________________________________________________________________________________________
| User  
|         VM(Virtual Machine)       VM(Virtual Machine)
|                |                          |
|               TAP                        TAP
|                | <- Linux Bridge          | <- Linux Bridge
-----------------|--------------------------|--------------------------------------------------
| Kernel         |                          |
|               TAP <- iptables            TAP <- iptables                Name Space  
|                |                          |                                 |  
|________________|__________________________|_______________________vNIC(virtual NIC/ 'S/D NAT')
| Hardware        \                         /                               /    \   
|                 OVS INT-BR(Internal Bridge)---OVS EXT-BR(External Birdge)      External Network(Internet)
|
|_______________________________________________________________________________________________
```

2. Openstack HW Network Hierarcy(Advanced)

```
Openstack HW Hierarcy

* OVN은 Namespace과 Linux Bridge, iptables의 역활을 모두 처리함. 그러므로 매우 단순한 구조를 가짐. 

* Linux Bridge : netlink
___________________________________________________________________________________________
| User  
|         VM(Virtual Machine)                VM(Virtual Machine)
|                |                                  |
|               TAP  -----------  OVN ------------ TAP-------------------OVN---------------
|                | <- Linux Bridge  \_iptables      | <- Linux Bridge      \_ vNIC
-----------------|----------------------------------|------------------------ / -- \ ------
| Kernel         |                                  |                        /      \
|                |                                  |                       /        \
|                |                                  |                      /          \          
|________________|__________________________________|_____________________/____________\____
| Hardware        \                                /                     /              \       
|                    OVS INT-BR(Internal Bridge)   ---  OVS EXT-BR(External Birdge)    External Network(Internet)
|
|___________________________________________________________________________________________
```

3. Openstack SW Network Hierarcy(Basic)

```
Openstack HW Hierarcy

* OVN은 Namespace과 Linux Bridge, iptables의 역활을 모두 처리함. 그러므로 매우 단순한 구조를 가짐. 
___________________________________________________________________________________________
| User  
|         VM(Virtual Machine)                VM(Virtual Machine)
|                |                                  |
|               TAP  -----------  OVN ------------ TAP-------------------OVN---------------
|                | <- Linux Bridge  \_iptables      | <- Linux Bridge      \_ vNIC
-----------------|----------------------------------|------------------------ / -- \ ------
| Kernel         |                                  |                        /      \
|                |                                  |                       /        \
|                |                                  |                      /          \          
|________________|__________________________________|_____________________/____________\____
| Hardware        \                                /                     /              \       
|                    Provider(Datacenter Netowork) ----------------------       External Network(Internet)
|                           (물리 Switch)
|___________________________________________________________________________________________
```

4. NameSpace vs IPtables vs OVN 

* Namespace는 각 서비스들의 연관성을 깨는 역할을 하는 것임. virtaul Device를 만들어 주는 것임.

* NameSpace는 "vNIC(S/D NAT), DNS Mask, Network subnet"을 만드는 역할을 한다. 

* iptables는 커널 영역에서 Routing table(L3) 및 L2 MAC, SEC Group(Security Group)을 만드는 역할을 한다. 반면 OVN은 User 영역에서 Routing table, L2 MAC(L3, L2), Security Group(SEC) 역할을 한다. 

* iptables는 각 compute node 별(server별)로 할당되어 관리되므로, 안정성은, 통합되어 DB가 관리되는 OSN보다 낮음. 물론, 갱신이 안되는 문제가 있음.

* Linux는 Bootloader 에서 이어지는 process 진행에서, 일부 process가 죽으면 동작하지 않음. 

* Kernel -> user -> bash -> httpd -> mysql 에서 bash가 죽으면 다 운영 안됨. 

* 위 구조에서 아래 구조로 변화하게 되었음. 각 구조별로 격리를 Namespace로 해주었음. 

* kernel -> user --> bash ($home)
                 \ 
                  -> httpd (/var/libhttpd)

* 따라서 Network에 대하여도 아래와 같은 구성으로 안정성을 확보하고 있음. 

* kernel -> mem -> vNIC -> cpu
                     \ 
                      \_ 가상장치 - NIC 


5. OVN(Open Virtual Network) 

* OVN은 Linux Bridge, DNSMASK, Namespace의 역할을 수행함.

* https://github.com/ovn-org/ovn/blob/