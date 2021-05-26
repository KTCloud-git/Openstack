# 서비스

Glance

# Image

Glance 
: Image를 관리하는 서비스로 RADOS Block Device를 사용하여 데이터를 저장함. API 및 Gateway 역할을 수행함. 

QCOW2
: Image 형식 이름을 의미하여 Ceph가 사용하려면, Ceph를 이용한다. 단, Ceph를 이용하지 않을 경우 가상 Disk를 만든다. 


## Image 사용과정

1. Ceph에 Image 저장됨. 이때 Ceph에 Image pool이 생성됨.

2. Image는 VMS(Virtaul Machine Service, 곧 instance로 읽힌다.)로 전환되어, Instance 생성시 해당 VMS를 참조하게 됨. 

## Image 속성 

1. Read, Write 등의 속성 변경이 가능함. 

2. Image가 Image 이름등을 저장하게 하기 위해 자원을 사용할 수 있도록 하는 서비스가 별도로 존재함.


# Flavor

Flavor 
: Hardware spec 뿐만아니라 Flavor를 통해 만들어진 Instance가 사용가능한 용량, 그리고 Project별 접속 등의 권한을 설정.
: IOPS Spec도 정의함.
: Network는 tc(* network를 setting 하는 process)를 사용함. (cgroup(*process별 접근 가능한 자원의 용량을 정의함)에 정의되어 있음.)

```bash
openstack image create \
--disk-format qcow2 \
--file ~/Downloads/osp-small.qcow2 \
rhel8-dbsmall
```

# Network

Project별 Network를 구성한다.

## Architecture

### Make instance 
```
User - Neutron API - Neutron Server (Agents(ml2), Agent(ml3), Agent(Linux Bridge)) -- OVN
                                                                                   \_ OVS
```

# Instance 

Project별 VM을 구성하는 방법을 배운다. 

## Requirements

0. Compute : Nova 서비스가 구동되고 있는 서버를 의미함. 

0. flavor(vCPU, Memory, Disk)를 Compute가 XML로 변경하여 libvertd로 전달을 하면, VM으로 올라옴. 

1. Image 

2. Network : Network + Subnet + SEC Group(Security Group)

## Architecture

### Delete instance 

명령어는 Message queue에 적재된다. 

```
User - Nova API - Nova server - MQ(Queueing Commands)
```

### 사용자 제공

```
Case 1. Windows 및 Linux 서버의 경우 : Instance - VNC(Virtual Network Computing) - User

Case 2. Linux 서버의 경우 : Instance - Serial - User 
```



