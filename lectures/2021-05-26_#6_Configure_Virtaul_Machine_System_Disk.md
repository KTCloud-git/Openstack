# Definition

이번 장에서는 Disk를 생성하기 위한 기능을 제공하는 모듈인 Cinder를 다룸. 단순히 모듈의 역할을 할 뿐임.

모든 VM의 Storage 들은 모두 휘발성임. 그래서 SWAP, Ephermal은 휘발성임. 하지만 Cinder를 통해 만들어진, Block Disk는 영구적으로 남음.

그러나 VM OS가 이미지가 아닌 Volume이라면 속도가 빠르고, 영구적으로 OS 내역이 남을 수 있음. 

Cinder는 1대 1로 매핑이됨. 단, manila project로 만들어진 NAS는 1 대 N의 관계를 가짐. 

# Service

Ceph 
: driver 이름. Software Driver

NFS
: driver 이름. Software Driver

EMC
: driver 이름. Software Driver

Gluster
: driver 이름. Software Driver

LVM
: driver 이름.

GFS
: driver 이름.

Glance 
: Image 저장용 / 단일 driver

Cinder
: 블록 저장용 / Multi driver

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
                 \_ NFS (Gluster) 
                 \_ GFS
                 \_ LVM
```

```
Backend(Driver, Device)는 모두 Type에 종속되어 있디.
VM - Ephermal 
   \_ block   - Cinder -- 
                       \_ Type - Driver - Device
                       \_ Type - Driver - Device
                       \_ Type - Driver - Device
```


``` Multi attach
+---------------------+     +--------------------------------+
|       Rack          |     |           Rack2                | 
| Node(VM1, VM2, VM3)-|--+--| +- Rack 1기반 이미지로 VM 생성 |
| Node(VM4, VM5, VM6)-|--+--| +- Rack 1기반 이미지로 VM 생성 |
| Cinder Node  -------|--+--|-+                              |
+---------------------+     +--------------------------------+
```

Block storage의 사례
```
    Cinder service -  Glusterfs
                   \_ ceph 
                   \_ nfs
                   \_ GFS+DLM

    block infra(multi block management) + Filesystem Type(Multi Access Journaling(Journal(1인이 한개의 경로로)+1Meta 정보))
    GFS는 Filesystem을 Journal과 meta로 구성해 놓음. 
    * ZFS, JFS, btrfs, xfs(multi locking:journal은 1개지만 multi로 filesystem 구현 가능) 제공함. 
```

Snapshot 
```
    OS -> Glance -> Snapshot -> Full -> (증분) -> (증분)
    OS -> Glance -> Image -> Cinder -> (저장)

```

Backup(Migrate / Snapshots)
```
    Nova --+
           |
        +--+
        |
     libvirt ----- VM

virsh snapshot-create-as --domain v1 overlay1 --diskspec vda, file=/var/lib/libvirt/images/overlay1.qcow2 

libvirt, qemu version을 확인하면 snapshot 방법을 확인할 수 있다. 

```
# Management 

1. Root(OS) Disk Management 
   vda0(Virtual Disk A~Z (Partition Number)) : virtio(반가상화 드라이버)를 사용함. 
   sda0(Scsi Disk A~Z (Partition Number)) : "virtio + scsi"를 사용함. windows/linux 물리장비에 붙을 때의 sda 를 사용함.   

2. Ephemeral Disk Management 
   cloud-init(Package)를 "/mnt"로 mount. HDD/SDD 및 OS DISK와 함께 넣는다. 

3. SWAP Disk Management 
   file 형태나 partition, disk를 사용한다. 
   Memory에서 사용 빈도가 낮은 것은 paging이 시작되고, paging된 정보는 Disk로 삽입됨.
   왠만하면 권장하지 않음. 왜냐하면 사용량이 낮은 것이 Swap에 넣어지면, 나중에 다른 VM이 사용할 때, computing의 대부분의 자원이 swap 에 대하여 memory로 로딩하는데 사용됨. 

4. Permanent Disk Management 

# Character 

1. Ephermeral Disk 
    특징 : Local에서 사용되기 때문에, 외부 성능에 종속성이 없다. 

2. Permanent Disk
    Ceph를 영구 스토리지로 사용함.
    Cinder도 영구 스토리지 관리를 위해 사용함./Glance도 쓸수 있는데, 하나의 Backend를 사용함. 
    1:1 / 1:N / 1:N(공유형태의 File Storage)
    DB나 Filesystem을 사용함. 

# Commands

```bash
#cinder container 진입
podman으로 확인하고, 
podman exec -it container name bash ...

#cinder 확인 
/etc/cinder 

#Driver 확인 
volume_backend_name=tripleo_ceph
grep -EV '^$|^#' cinder.conf 
```
# Question 

1. backing file 도 휘발성인가.  

2. 왜 Openstack을 Microservice Arhictecture가 아니라고 하는가. 

Microservice architecutre는 function별(Model, View, Controller)로 나눈 것, Modulur architecture는 각 module(OS 시스템 자체/물리적 서비스 자체)의 집합임.

