# Purpose

* Hardware를 Code로 관리하여 프로젝트 단위별로 H/W를 베포하기 위해 만들어 졌음 

* 최소 인원으로 관리할 수 있도록 함.

# Character

* External과 Internal Database 의 정합성이 안 맞는 경우가 많음.

* Open stack과 VM Ware의 차이점은 Openstack은 직접 Resource를 조회하지 않고, Database를 통해서 조회함. 

* Database와의 Date sync는 각 서비스별 Event(Turn on/off)에 따라 이루어짐.

# History

* 최초에 NASA에서 Rockspace(No rockplace)가 만들었음. 

* Open Satck Platform으로 배포되고 Python, Oslo로 만들어졌고, 중국인 및 아시아인들이 대거 참여하여 Kilo로 만듦.


```
현재의 프로젝트 모습 : 각 영역의 구체적인 내역이 상세화 되어 있는 것을 볼 수 있음. 

H/W 
 |- nova : IMG, Project
 |- keystone : 사용자 + 프로젝트
```

# Setting

* Over Cloud : Rack 위의 서버들로, Open stack 서비스의 모음을 의미함. Undercloud의 관리 서버가 관리하는 서버의 그룹 의미. 

* Under Cloud : Over Cloud를 구성하도록 돕는 서비스들의 모음.(Utility, kolla, power로 구성되어 있음.)

# Architecture

TS(Trouble Shooting)이 어렵고, PS(Performance Setting)이 어렵다.

```
Openstack SW Hierarchy  
 |- (Level 5)
 |-- OSP
 |- (Level 4)
 |-- Oslo(Openstack Common Library) 
 |- (Level 3)
 |-- Python
 |-- DPDK(OVS기반의 서비스)
 |-- MQ(Messaging service)
 |- (Level 2) 
 |-- libvirtd : Nova가 VM을 생성할때 사용하는 Deamon, "/etc/libvirtd"에 존재, virsh list로 확인 가능
 |-- Linux Bridge : 최초 VM의 etho0와 tap을 연결하기 위한 서비스
 |-- OVS : Linux Bridge 하단에 중간 계층을 넣음.(Linux bridge는 종료되면 모든 서비스에 영향을 미침) 
 |- (Level 1) : 
 |-- OS(KVM(전가상화) or QEMU(반가상화))
 |-- Namespace : NIC를 분할하기 위해서임.
 |- (Level 0)
 |-- CPU(vt-x)
 |-- MEM(vt-d(SR-IoV, PCI-Pass))
```

```
Openstack HW Hierarcy
____________________________________________________________________________________________________________ 
|   User 영역
|   * VM (eth0) : SW이고 Tab device가 필요하다. 
------------------------------------------------------------------------------------------------------------
|   User 영역
|   * Tap driver : 논리 장비로 Kernel 영역의 Tab driver와 연동됨.
|   * Linux Bridge (veth(patch)), Namespace (NIC dividing, Namespace도 파일임.)
|   * OVS : Bridge로 Data를 원하는 장소로 전달되도록 설정해줌. 관리를 위해 사용함.
|   * DHCP(dnsmasq) / network4, network6 구성, 한 Project 데이터는 이 구역 사용하여 데이터를 교환함.
------------------------------------------------------------------------------------------------------------
|   Kernel 영역
|   * Tap driver : 논리장비로 User 영역의 Tab driver와 연동됨.
|   * iptables : (L2 (ebtables: MAC / arp-lables: ARP) / L3(NAT, Forward, Input), Conntrack(Data tracking)) 
|   * Namespace : vNIC를 생성하는 단위
|-----------------------------------------------------------------------------------------------------------
|   Device 영역
|   * Ethernet
|-----------------------------------------------------------------------------------------------------------
```

```
OS 구조 (x86 Ring 구조) 
------------
3(사용자) - VM(Process)으로부터 들어온 데이터
----------------------------
2(사용자) - 드라이버(Priviliaged user가 사용 가능)
-----------------------
1(커널) - 드라이버(priviliaged user가 사용 가능)
------------------
0(커널) - NIC
```

```
Openstack services
------------------
Identity service                        
Neutron Service ((OVN, OVS, iptables/nftables, LinuxBridge, Namespace device)                       
Compute service                                      
Image Storage service ((Glance:API, File Gateway(Up, Down), Cinder data(base/backup image))
Object Storage  (Swift: FS가 없음. 응용프로그램 및 메타정보 손상시 요단강...)                       
Telemetry service  (Cilo meter)           
Key stone  (계정 관리 서비스)               
Manila (NAS)           
Octavia (Loadbalancer)
------------------                                       
```

# Security

* 권한 이전에 인도인과 일할 때, Role을 지워버렸을 때가 많음. 인도인이 Roll off 하면 대부분 이런 상황이 발생함. 본인이 만든 코드를 지워버림.

```
User : Role의 모임
|- Role

Role : 프로젝트의 모임
|- Project 
```

* SEC Group : iptables, OVN (ingress, egress) / 기본적으로 Stateful하다. (Session 정보를 추적. : iptables + netfilter(OVN에서 제공) -> connect_track)

# Instance 생성 

1. Bootable Image 

* base image 와 extended image 그리고 extended image(Read only)와 Disk image 위치에 저장된 backing file(Wrtie)로 구성됨. 이때 각 VM이 로딩될때는 extended image와 Backing file이 참조되어 올라옴. 그래서 나중에 VM 올릴 때, base image가 삭제되면 오류가 발생함.

2. internal netowork(cloud-init)

3. security group(d, default = reject)

4. Flavor 를 통한 offering setting

4. 그외 cinder 등은 불필요

# Glossary 

* Admin : Domain의 이름으로 Bootstrap 역할을 함. 

* User : Domain의 이름으로 각 서비스의 운영자 역할.

* OSP Cluster : Project의 모음으로 kolla나 Director로 배포된다. 

* Project : 자원들에 대한 정리가 되는 것이고 server(instance/VM(libvirt, qemu/kvm), flavor(DB에만 존재하는 내용이고, DB, CPU/MEM/DISK Spec 명시됨), image(qcow3, standard), volume(lvm2(block + iscsi), NFS(block file + iscsi)), network(route(GW), network(subnet), port(IP), i/egress, namespcae(d)) 등이 만들어 진다. 서로 다른 Project는 격리되어 있다.

* Floating IP : S/D NAT 

* Fixed : Instance IP (Switch: DHCP, SDN: dhcpd, dnsmasq(d), IP Fixed)

* libvirt : 여러 hypervisor를 연결함. (QEMU, KVM, VMware에 연결), Docker 나 LXC(Linux container, Hardware를 직접 접근)

* LXC : Linux에서만든 Container이나 VM에 가깝게 동작기 때문에 커널을 그대로 가져와서 재시작이 이루어진다. OSC-I, Podman, Docker는 재시작이 이루어지지 않는다. 

# Site

* https://rol.redhat.com

# Instructor 

* 최
