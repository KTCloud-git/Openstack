# define

* Keystone : 각 서비스가 접속 가능한 경로를 정의하고, 접속 가능할 때 인증하는 체계를 제공해 주는 서비스를 의미함. 중간에서 경로 설정 및 token(식별자) 생성 역할 밖에 안함. 역할을 심어서 보냄. 


# Architecture

```
          USER  -   PROJECT
              \    /
    API(Rest)  ROLE
        \         \
        token      Policy.json 
            \       /
              member
```

# 관련 커멘드

```
student@workstation ~(admin) # openstack catalog list

service name / type / endpoint 
   Nova  / service / 123.3.4.5
```

# Glossary 

## 기본 용어

* keystone : 

* catalog : 각서비스 접속 위치별 endpoint의 묶음. 이며 각 서비스의 Access point임. Nova, Cinder, Glance의 endpoint 내의 endpoint를 알려준다.

* endpoint : Internal/Public/ADM의 URL을 의미함. 

* user : service(Nova, Cinder, Keystone), member, Admin. policy.json에 정의가 되어 잇지 않으면 모두 member와 role이 매핑이 됨. 

* role : user와 project를 연결하는 단위 

* token : user가 로그인을 하면 endpoint에 접근할 수 있는지 없는지를 판단함. (기한을 정함.)

## 격리의 단위

* project : admin, service로 구성되어 있음. Object(객체)를 격리 및 분리하기 위한 목적으로 만들어짐. 

* domain : project는 domain으로 구분이 된다. 기본값 : default

* region : 기본값 RegionOne


