# Definition

이번 장에서는 Disk를 생성하기 위한 기능을 제공하는 모듈인 Cinder를 다룸. 단순히 모듈의 역할을 할 뿐임.

# Service

Ceph 
: driver 이름. Software Driver

NFS
: driver 이름. Software Driver

EMC
: driver 이름. Software Driver

Gluster
: driver 이름. Software Driver

# Architecture

```
Nova Compute (VM(Process로 qemu_kvm이 구동시킴)-Operating System) - Cinder Service - Software Driver - Block disk
                                                 
```

```
Glance Service  
              \_ (flavor(root disk), image info) -> Nova Service (Base Iamge(root disk+ Image info)) -(각 Base Image를 가지고 Boot up KSM) -> VM
                                                                                                                                           \_> VM
                                                                                                                                             ... 

```

Block storage의 사례
```
    Nova service -  VM
                 \_ NFS 
                 \_ GFS
                 \_ LVM
```

# Question 

1. backing file 도 휘발성인가.  

2. 왜 Openstack을 Microservice Arhictecture가 아니라고 하는가. 

Microservice architecutre는 function별(Model, View, Controller)로 나눈 것, Modulur architecture는 각 module(OS 시스템 자체/물리적 서비스 자체)의 집합임. 