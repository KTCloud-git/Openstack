# 생각할 것들 

1. Workflow 

2. API Document

# Dashboard 접속 방법 

1. dashboard.overcloud.example.com 

2. cat ~/*-*-rc 

# profile 변경 

```bash
# Network 생성 
# L2
# --- Check point ---
# - Shared : Yes : 다른 Project에서 볼 수 있음.
# - External Network : Yes : 외부에서 볼 수 있음. 
openstack network create test-network

# Subnet 생성 
# Subnet Name : 원하는 내용
# Network Address : 원하는 내용 
# IP Version : IPv4 
# Subnet Defaults : Enable DHCP 
# Allocation Pools : 10.10.10.100,10.10.10.200
# DNS Name Server : 10.10.10.20

# Router 생성 
# L3
Dashboard를 통해 생성한다.

# Security Group 생성 
# L3에서 적용해 주는 것을 의미한다. 
Dashboard를 통해 Create Security Group를 한다. 방화벽 설정을 의미한다.

# Volume 생성 
Dashboard를 통해 볼륨을 생성한다.  
* 자주 나오는 질문 Create transfer를 통해 다른 Volume으로부터의 데이터를 받아올 수 있음. 

#volume type을 생성한다. 
openstack volume type create glusterfs 

#volume type을 확인한다.
openstack volume type list

#volume을 생성한다. 
openstack volume create 


openstack server add volume 

    #현재 마운트된 Volume을 확인할 수 있음. 
    lsblk 
    #포멧을 한다
    sudo mkfs.ext4 /dev/vdb 
    #mount를 한다.
    sudo mount /dev/vdb /mnt 
    #해당 위치로 간다. 
    cd /mnt 
    #unmount를 한다. 
    sudo umount /dev/vdb
```

# Key-pair 생성
# 재발급이 안되는 Private key pair 를 생성한다. 
이때 cirros os를 사용하는데 id는 cirros로 password는 gocubsgo 

# Image 생성 
Private, Shared, Community 세가지 유형으로 공유하는 대상을 정할 수 있음.

# Flavor 생성 
Default Flavor로 생성함. 


# image download
# image download는 Dashboard에서 제공되지 않음.
glance image-download test-volume-uuid --file test-volume.raw


## 임시 디스크 생성
## Swap 생성 

# VM 생성 

```

openstack group create --domain Default test-group