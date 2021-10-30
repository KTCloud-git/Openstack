# 실습 환경 구성

## 관련 

1. distroWatch.com 을 통해서 업데이트 주기를 확인할 수 있음.

## 기본 Infra 설정

1. VM은 4 core X 8G giga 3대 만들고, Disk는 100Giga 수준으로 만듭니다. 

2. "compute0 / compute1 / controller0" 역할을 수행합니다. 

   3개의 VM을 만들고 각각 포트 포워딩 해줍니다.

3. private subnet 을 만들어 NIC를 추가합니다. 이때 Linux ubuntu 18.04를 기준으로 

   "/etc/network/interfaces.d/50-cloud-init.cfg"에 아래 내용을 추가한 후에

   eth0:
        dhcp4: false
        addresses: [IP Address/CIDR]

   netplan apply를 수행하도록 합니다.

   *통상 NetworkManager라는 유틸리티를 쓰는 것으로 알고 있습니다.

4. ssh key를 생성

4-1. 
'''
ssh-keygen -t rsa -N'' -f ~/.ssh/id_rsa 
sshpass -p{비밀번호} ssh-copy-id ubuntu@<IP>
'''

4-2. ssh Key를 이용하여 접속하게 하려면 아래 설정을 해야함. 
'''
cd /etc/ssh/sshd_config
*/Pub로 검색해서 Yes
systemctl restart sshd
'''

4-3. sudo 명령어 수행시 password를 요청하면 아래 설정을 해주어야 함.
'''
/etc/sudoers.d/nopasswd
ubuntu ALL=NOPASSWD: ALL
'''


5. controller0에 22번으로 접속 후 아래 명령어를 각각 수행합니다.

   git clone https://github.com/openstack/kolla
   
   git clone https://github.com/openstack/kolla-ansible


(
  
 아래 내용에는 Openstack Kolla Ansible 자료를 참고하였습니다.

 https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html 

 해당 내용을 보시고 어려운 부분을 참고하시면 됩니다.

)


* 어느 정도 진행을 하면 아래 multinode, all-in-one 에 대하여 나오는 데, 각각 다음과 같은 뜻이 있음.

    * multinode : 여러 VM(Baremetal)에 Kolla를 이용해 Openstack을 설치할 경우 사용함. 
   
    * all-in-one : 하나의 VM(Baremetal)에 Kolla를 이용해 Openstack을 설치할 경우 사용함.



* network 구성과 관련된 궁금점.
'''
vim /kolla-ansible/etc/kolla/globals.yml 

multiple management network : 서비스 구성을 위한 network.

neutron external network : 외부로의 인터페이스를 위한 network
'''

* 각 compute 서버에서 python을 재설치 해야함. 
'''
apt install --reinstall python3-pkg-resources python3-setuptools
apt install python-setuptools 
'''

* 서버 초기화
kolla-ansible -i ./multinode bootstrap-servers 

