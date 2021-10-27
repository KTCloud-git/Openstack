# 명령어 모음

## libvirt

* brew install libvirt

## Environment setting

* brew install ipmitool

* pip install virtualbmc

## Under cloud 와 Over cloud 의 관계 

* Under cloud에서 "노드, 구성 요소, 시나리오의 성능 업데이트", "보안 수정과 같은 마이너 릴리스 업데이트 수행", "Red hat OpenStack Platform에 대한 주요 버전 업그레이드 자동 수행" 

* director를 만들어 해결이 안되니, contianer를 사용하여 rolling update등이 원활하게 되게 되었음. 

## 장단점 

* Under Cloud와 Over Cloud 간의 데이터 불일치. 

* Under Cloud와 Over Cloud 간의 데이터를 교환할 때, Python을 사용하게 되는데, Python의 Context switching 등의 비용이 높은 단점이 있음. nP

## Glossary 

* pacemaker : HA를 만들기 위한 방법.

* network : overcloud 에도 서비스별로 management network와 eth0 network로 구분이 됨.

* provisioning : OS 영역에 대한 처리를 해주는 부분.

* kolla : OS 영역의 윗단, AP 부분에 대한 처리를 해주는 부분.

## Commands

1. https://github.com/machyve/xhyve

2. openstack server list 

## architecture

+-----+
| amq | 
+-----+
| sys | -> systemctl, podman, libvirtd (virsh ...)
+-----+
| net | -> ovs, ovn (user space) / bridge, ip, ss (kernel space)
+-----+


