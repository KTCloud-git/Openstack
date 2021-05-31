# Architecture

+-----------Virtual Machine-----------+
|    "ip a"로 Floating IP 확인 가능   |
|------------QEMU Agent---------------|
                 |
+--------------Switch-----------------+
|             OVN / OVS               |
+-------------------------------------+
                 |
+--------------Switch-----------------+
|           OVS/Floating IP           |
+-------------------------------------+


# Resources 

Provider network : provider라는 것 자체가 external을 의미함. L2 단계에서 외부와 통신하는 Network를 의미함. 


# Configuration 

router:external : External

# Reference 

* libvirt.org/kbase/live_full_disk_backup.html
* www.ovh.com/blog/create-and-use-openstack-snapshots/
