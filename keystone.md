# Preface dd

* Key stone : 건물의 한 부분과 다른 한부분을 잇는 돌조각을 의미함.

![Key stone is the final piece placed during construction and locks all the stones into position](/img/keystone.png)
# Definition

* Keystone
  
  각 서비스로 접속 가능한 경로 정의하고 Token을 생성하는 역할을 수행함.
  
  접속 가능할 때 인증을 돕는 주는 서비스
  
  즉, 하나의 서비스가 요청을 하면 중간 전달자로서 역할을 심어서 다른 서비스로 전달하는 역할을 함. 그러므로 인증 자체는 다른 서비스가 수행함. 

# Architecture
## 인증의 구조 

1. 특정 member가 어느 "USER- PROJECT -ROLE"과 매핑되는지 Policy.json 파일에 정의함. 

2. tokne을 통해 policy.json에 어떤 memeber로 정의되어 있는지 확인

3. 해당 member에 해당하는 role을 확인하여, 접속 가능한 project를 확인

```bash
          USER  -   PROJECT
              \    /
    API(Rest)  ROLE
        \         \
        token      Policy.json 
            \       /
              member
```

## 그럼 Open stack key stone은 

1. API가 어느 서비스를 향해 가는지 확인 (경로 설정)

2. 위 "인증의 구조"에서 token을 만들어 주는 역할 수행 (Token 생성)

# Openstack에서의 Command

```
student@workstation ~(admin) # openstack catalog list

service name / type / endpoint 
   Nova  / service / 123.3.4.5
```

# Glossary 

## 기본 용어

* catalog : 각서비스 접속 위치별 endpoint의 묶음. 이며 각 서비스의 Access point임. Nova, Cinder, Glance의 endpoint 내의 endpoint를 알려준다.

* endpoint : Internal/Public/ADM의 URL을 의미. 

* user : Keystone을 사용하는 서비스나 유저를 의미. service(Nova, Cinder, Keystone), member, Admin. policy.json에 정의가 되어 있지 않으면 member와 role이 매핑 되는 특성을 지님.

* role : user와 project를 연결하는 단위. 어느 유저가 어느 프로젝트에 접속 가능한지를 정의.

* token : user가 로그인을 하면 endpoint에 접근할 수 있는지 없는지를 판단하는 기준.

## 격리의 단위

* project : admin, service로 구성되어 있음. Object(객체)를 격리 및 분리하기 위한 목적으로 만들어짐. 

* domain : project의 집합. 기본값 : default

* region : domain의 집합. 기본값 RegionOne


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUwMDgxMTMzOV19
-->