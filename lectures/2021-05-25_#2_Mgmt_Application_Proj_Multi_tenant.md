# Managing Application Projects in Multitenant Cloud (Multi Tenent Cloud 구성)  

1. 프로젝트별 사용가능한 리소스를 제한하여 구성할 수 있음. 

2. 클라우드 사용자 별로 접근 가능한 프로젝트를 정의할 수 있음. 

3. 프로젝트 리소스를 제한. 

* 최초 프로젝트 생성 시 Defaults template의 성능 정보를 사용하여 자원 제한 정책을 할당함. 

## 구분의 단위

아직 왜 구분을 해야하는지는 모르겠음. 단순히 지역별 Data center를 관리하기 위해 적용함. 데이터 참조하는 Storage 문제 때문에 어려움이 존재함. 

서로 다른 Region이라도 Keystone 설정 값 등은 같은 Storage로 보관되어 있는 경우가 많음. 그러므로 Storage Migration 시에 순단은 거의 반드시 발생하는 경우가 많음. 

1. Region : Keystone 즉, 계정 관리 체계의 단위. 

2. Domain : 사용자 계정, 프로젝트, 공유 자원의 집합.

3. Project : 자원과 역할(Role)이 매핑된 사용자의 집합, 사용자가 만든 Application이 배포되는 단위