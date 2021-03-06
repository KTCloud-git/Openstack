# Openstack Tenant 및 Identity 관리

</br>
<h2>목차</h2>

- Keystone은 무엇인가?
- Operation for cloud operator
- Keystone은 어떻게 작동할까요?
- Region / Domain / Project의 개념
- Keystone의 구성요소
- Reference : Openstack Official Document

</br>

---

## Keystone은 무엇인가?

Keystone은 Openstack's identify API 서비스를 개조해서 Identity, Resource, Assignment, Token, Catalog 서비스를 제공합니다.

Openstack's identify API 서비스는 authentication token을 생성하는 역할을 합니다. 

구체적으로 아래 서비스로 구성되어 있습니다.

Identify 서비스는 고객 계정 정보(소속, 이름, 아이디, 비밀번호)를 관리합니다.

Resource 서비스는 Project, domain의 정보를 관리합니다.

Assignment 서비스는 Role 정보와, Resource, Identify, Role 매핑정보를 관리합니다. 

Token 서비스는 토큰이 유효한지 확인하고, 유효하도록 처리해주는 역할을 합니다. 

Catalog 서비스 각서비스의 end point를 관리하는 역할을 합니다.

<br/>

아래는 각 서비스를 호출하는 소스코드의 예제입니다. REST 형식으로 End point를 호출하는 것이 다입니다. 

```python
class UserResource(ks_flask.ResourceBase):
    collection_key = 'users'
    member_key = 'user'
    get_member_from_driver = PROVIDERS.deferred_provider_lookup(
        api='identity_api', method='get_user')

    def get(self, user_id=None):
        """Get a user resource or list users.
        GET/HEAD /v3/users
        GET/HEAD /v3/users/{user_id}
        """
        ...

    def post(self):
        """Create a user.
        POST /v3/users
        """
        ...

class UserChangePasswordResource(ks_flask.ResourceBase):
    @ks_flask.unenforced_api
     def post(self, user_id):
         ...
```

위와 같이 각 서비스를 end point로 하여 요청을 분기처리하고, 해당 과업들을 수행합니다.

<br/>

## Operation for cloud operator

지난 2회차 수업은 운영에 대한 업무를 배웠습니다. 

Key stone 서비스 운영자로 해야하는 일이 있다면 "적합한 사용자에게" "적합한 token"이 부여되는지를 확인하는 것입니다. 

그럼 어떻게 "적합한 사용자"와 "적합한 token"을 확인하면 되는지 알아볼까요?

이전 강의에서 말씀드렸듯이 Keystone은 Role Based access control을 합니다. 

간단히, project, domain, system 범위 별로 해당 actor가 가지고 있는 Role에 따라 사용할 수 있는 api가 정해집니다. 

현재의 Rocky release 버전에서는 role은 admin, member, reader로 구분되어 있고, actor는 user와 group으로 구분되어 있습니다. 

|#|role|description|
|-|----|-----------|
|1|reader| 조회 |
|2|member| 조회, 생성 |
|3|admin| 조회, 생성, 삭제 |

아래는 system scope에 대한 role와 actor 정보를 보여주는 명령어입니다.

```console
$ openstack role assignment list --names --system all
+--------+------------------------+------------------------+---------+--------+--------+-----------+
| Role   | User                   | Group                  | Project | Domain | System | Inherited |
+--------+------------------------+------------------------+---------+--------+--------+-----------+
| admin  |                        | system-admins@Default  |         |        | all    | False     |
| admin  | admin@Default          |                        |         |        | all    | False     |
| admin  | operator@Default       |                        |         |        | all    | False     |
| reader |                        | system-support@Default |         |        | all    | False     |
| admin  | operator@Default       |                        |         |        | all    | False     |
| member | system-support@Default |                        |         |        | all    | False     |
+--------+------------------------+------------------------+---------+--------+--------+-----------+
```

아래는 system administrator에 대한 role과 actor 정보를 보여주는 명령어입니다.

```console
$ openstack role assignment list --names --system all --role admin
+-------+------------------+-----------------------+---------+--------+--------+-----------+
| Role  | User             | Group                 | Project | Domain | System | Inherited |
+-------+------------------+-----------------------+---------+--------+--------+-----------+
| admin |                  | system-admins@Default |         |        | all    | False     |
| admin | admin@Default    |                       |         |        | all    | False     |
| admin | operator@Default |                       |         |        | all    | False     |
+-------+------------------+-----------------------+---------+--------+--------+-----------+
```

아래는 system member와 reader에 대한 role과 actor 정보를 보여주는 명령어입니다.

```console
$ openstack role assignment list --names --system all --role member --role reader
+--------+------------------------+------------------------+---------+--------+--------+-----------+
| Role   | User                   | Group                  | Project | Domain | System | Inherited |
+--------+------------------------+------------------------+---------+--------+--------+-----------+
| reader |                        | system-support@Default |         |        | all    | False     |
| admin  | operator@Default       |                        |         |        | all    | False     |
| member | system-support@Default |                        |         |        | all    | False     |
+--------+------------------------+------------------------+---------+--------+--------+-----------+
```

위와 같이 조직별로 원하는 actor에 따라서, openstack에서는 persona라고 하는데요. 

부여된 role과 scope가 다르게 됩니다.

key stone 서비스 자체에 대한 관리는 [OpenStack 공식 문서](https://docs.openstack.org/keystone/latest/cli/index.html) 를 참조하세요.

<br/><br/>

## Keystone은 어떻게 작동할까요

Keystone이 어떻게 작동되는지 알아 보겠습니다. 

Keystone은 **Token Backend, Catalog Backend, Policy Backend, Identity Backend**로 구성되어 있습니다.

Token으로 사용자를 식별해서 해당 Token으로 접근 가능한 End-point를 확인하고, 해당 End-point에서 무엇을 할 수 있을지 확인합니다.

<br/>

<div style="text-align:center">
  
<img src="./img/logical architecture.png" width="600" height="300"> 

<br/>

+ **Identity Backend** : 사용자 및 그룹에 대한 인증을 관리

+ **Token Backend** : 사용자의 임시 토큰(Openstack 서비스에 접근하기 위한 신분 증명 데이터)을 관리

+ **Catalog Backend** : 모든 Openstack 서비스에 대한 End-Point URL 관리(Openstack client에서 Openstack Service 접근을 위한 정보)

+ **Policy Backend** : 사용자 및 권한 등에 대한 Role을 관리

</br>

인증의 실체 아래와 같이 데이터가 전달되었다고 했을 때, 

해당 사용자가 프로젝트에 적합한 사용자임을 인증받게 되어, token을 발급받게 됩니다.

```json
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "id": "0ca8f6",
                    "password": "secretsecret"
                }
            }
        },
        "scope": {
            "project": {
                "id": "263fd9"
            }
        }
    }
}

```

문제 : 이전 수업을 토대로 아래와 같은 요청 전문은 유효할까요?

참고로 "auth"의 "identity" > "password" > "user" > "id" 부분이 바뀐것을 알 수 있습니다.

```json
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "domain": {
                        "name": "acme"
                    }
                    "name": "userA",
                    "password": "secretsecret"
                }
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "id": "1789d1"
                },
                "name": "project-x"
            }
        }
    }
}
```

정답, 유효합니다. 왜냐하면 Domain에서 user name은 unique하기 때문입니다. 

<br/>

> 참고 : Keystone 동작 절차 ([참고](https://galid1.tistory.com/207))
<br/>

1. **User가 Keystone으로부터 범위 비지정(Unscoped) Token 획득**

   - User는 Nova 서비스를 통해 VM을 생성하는 권한을 획득하기 위해 Keystone에 접근

   - Keystone은 User에게 신분 증명을 요구

   - User는 ID, PW를 제출하여 신분을 증명

   - Keystone은 인증에 성공한 User에게 **Unscoped Token을 발급**  

      (현재 Token에는 User가 사용가능한 서비스들에 대한 범위가 포함되어 있지 않음. 따라서 다른 서비스들에 접근이 불가능)

</br></br>

<div style="text-align:center;">
  <img src="./img/step1.png" width="300" height="300">
</div>
</br>
</br>

2. **Keystone이 User가 접속 가능한 프로젝트 조사**

   - 신분 증명을 완료하여 Unscoped Token을 받은 User는 해당 Token을 통해 자신이 접속 가능한 프로젝트를 요청

   - Keystone은 User에게 할당된 Role에 따라 프로젝트의 종류와 수행 가능한 End-Point 목록을 작성  

      (현재 단계에서는 User에게 End-Point 목록이 전달되지 않은 상태)

</br></br>

<div style="text-align:center;">
  <img src="./img/step2.png" width="300" height="300">
</div>

</br>
</br>

3. **User가 Keystone으로 부터 범위 지정 토큰(Scoped Token) 획득**

   - **Keystone은 User가 접속 가능한 프로젝트 목록과 User가 사용을 원하는 프로젝트를 결정**

   - 위의 정보(프로젝트와 Role)가 포함된 Token을 User에게 보냄
</br>
</br>

<div style="text-align:center;">
  <img src="./img/step3.png" width="300" height="300">
</div>
</br>
</br>

4. **User가 서비스에게 수행 요청**

   - 범위가 지정된 Token을 받은 User는 Token 내부의 End-Point를 확인하고 서비스에게 수행을 요청
</br>

<div style="text-align:center;">
  <img src="./img/step4.png" width="500" height="400">
</div>

<br/>
<br/>

5. **요청을 받은 서비스가 Keystone에게 Token 검증 의뢰**
   - 서비스는 User가 요청과 함께 제출한 Token의 메타데이터 정보가 유효한지 검증하기 위해 Keystone에게 의뢰
   - Keystone은 서비스로부터 받은(User가 서비스에게 제출한) 메타데이터와 Policy Backend에 저장된 메타데이터와 비교(RBAC)
<br/>

<div style="text-align:center;">
<img src="./img/step5.png" width="500" height="400">
</div>
<br/><br/>

6. **검증이 완료된 Token을 서비스에게 제공**
   - User가 요청한 서비스에 접근이 가능한지 Token의 메타데이터 검증을 마친 Keystone은 그 결과를 서비스에게 제공
<br/><br/>

<div style="text-align:center;">
  <img src="./img/step6.png" width="500" height="400">
</div>
<br/>
<br/>

7. **서비스가 User의 요청 실행**
    - 서비스는 Keystone으로부터 Token 검증을 완료한 후 인스턴스를 실행하거나 볼륨을 생성하는 등의 요청을 수행
<br/>
<br/>

8. **작업에 대한 응답**
    - 서비스는 User의 요청에 대한 수행 결과를 응답
</br>

<div style="text-align:center;">
  <img src="./img/step7.png" width="500" height="400">
</div>
</br>
</br>


## Keystone의 구성요소 ([참고](https://docs.openstack.org/keystone/latest/getting-started/architecture.html))

<br/>

+ Authentication
   + 사용자를 인증하는 절차로 특정 값을 통해 Keystone이 이를 검증
   + ID, PW가 사용되며 Keystone은 인증 완료 시 **인증(임시) 토큰**을 발행
</br></br>

+ End-Point
  + 서비스를 이용하기 위한 네트워크 주소로 URL(IP, PortNum)을 사용
  + End-Point 유형
    - admin : 운영자들에게만 접근이 허용
    - internal : 내부 서비스간에만 접근이 허용
    - public : 인터넷에서 인식이 가능하며, 사용자들이 인터넷을 통해 자신의 클라우드 환경을 제어할 때 사용
</br></br>

+ Role
   + 사용자가 어떤 동작을 수행하도록 허용하는 집합
   + 사용자가 가지는 역할은 사용자에게 발행된 **인증 Token**에서 찾을 수 있음
</br></br>

+ Tenant
   + 자원에 대한 User(Role)의 집합
</br></br>

+ Token
   + RBAD의 신분을 증명하기 위해 사용되는 데이터
   + 어떤 자원에 접근이 가능한지 지정되어 있음. (Authorization)
</br></br>

+ User
   + 사람 또는 Openstack 서비스를 이용하는 서비스(nova, neutron, cinder 등)를 의미
   + User는 특정 프로젝트에 할당할 수 있으며, 중복을 허용하지 않음
</br></br>

## Region / Domain / Project의 개념

> ### Region ([참고](https://docs.openstack.org/python-openstackclient/rocky/cli/command-objects/region.html))
</br>

Region은 다른 Region과 동일한 Keystone의 관리를 받습니다.

<div style="text-align:center;">
  <img src="./img/region.png" width="550" height="350">
</div>
</br></br>

> ### Domain([참고](https://docs.openstack.org/mitaka/ko_KR/install-guide-rdo/keystone-users.html))
</br>

Region의 하위 집합입니다.

Domain은 Project, User, User들의 집합인 그룹으로 되어 있습니다.

<div style="text-align:center;">
  <img src="./img/Domain.png" width="750" height="400">
</div>
</br>
</br>

현재 Keystone에서는 Default라는 이름의 기본 Domain을 제공합니다.

Domain 관리자는 Domain에서 Project 및 User, Group을 만들고 Domain의 User 및 Group에 Role을 할당할 수 있습니다.

하나의 인증 체계(Keystone)을 공유하므로 다른 Region/Domain/Project 자원에 접근이 가능
</br></br>

> #### Project(Tenant)
</br>

Project(Tenant)는 Identity v2까지 Tenant라는 용어로 불렸지만, 

개념적 표현보다는 직관적인 표현으로 표기하기 위해 Identity v3부터 Project라는 용어로 변경되었습니다.
</br>

Project란 Openstack 서비스 리소스(Server, Image 등 - Project 및 Domain에 데이터 제공)를 그룹화하고 격리(기본 소유권 단위)합니다.

Project는 Domain에서 고유해야 합니다.  

User 또는 User Group을 Role 할당 개념으로 사용하여, Project 리소스에 대한 접근 가능 권한을 부여할 수 있습니다.  

권한이 부여된 User는 특정 Role에 따라 접근 유형 및 기능이 결정됩니다.

</br>
</br>

## Reference : Openstack Official Document
</br>

### Token ([상세보기](https://docs.openstack.org/keystone/latest/admin/tokens-overview.html))

위에서 Keystone 동작 절차에 대해 설명할 때 Unscoped Token, 범위가 지정된 Token 개념이 나왔었는데, 해당 부분을 다루어 보겠습니다.

### 권한 부여 범위

Token은 Role 할당 정보를 전달하는데 사용됩니다.  

User는 Project, Domain 또는 전체 시스템에 걸쳐 여러 Role을 가지고 있는 것은 자주 있는 일입니다.  

이러한 범위를 **권한 부여 범위**라고 합니다.

여기서 Token은 단일 작업 범위(ex. Project, Domain 또는 시스템)를 가지기 때문에, 

만약 Project로 범위가 지정된 Token은 다른 Project에서 다른 작업을 수행하기 위해 재사용할 수 없습니다.

따라서, 각각의 권한 부여 범위는 특정 Openstack 서비스에서 특정 유형의 작업에서만 유용하며 상호 호환되지 않습니다.

#### 범위가 지정되지 않는 Token(Unscoped Tokens)

Unscoped Token은 **서비스 카탈로그, Role 또는 권한 부여 범위(ex. Token 내의 Project, Domain 또는 시스템 속성)가 포함되지 않습니다.**  
주요 사용 사례로는 원본 자격 증명을 반복적으로 제시하지 않고 나중에 Keystone에 Identity를 증명(범위가 지정된 Token 생성)하는 것이 있습니다. Unscoped Token을 받기 위해서는 아래와 같은 조건이 만족되어야 합니다.

- 인증 요청에 권한 부여 범위를 지정하지 않아야 합니다.
(ex. --os-project-name 또는 같은 인수가 있는 명령 줄에서 --os-domain-id)

- Identity에는 Role 할당 및 권한 부여와 관련된 "기본 프로젝트"가 없어야 합니다.

#### 프로젝트 범위 Token(Project-Scoped Token)

Project는 Volume이나 Instance와 같은 리소스의 컨테이너라고 생각하시면 됩니다. 프로젝트 범위 Token은 클라우드의 특정 테넌시(Tenancy)에서 운영할 수 있는 권한을 나타내며, 컴퓨팅 리소스를 가동하거나 블록 스토리지를 분할하는 작업 등에 유용합니다. 추가적으로, **프로젝트 범위 Token은 서비스 카탈로그, Role 및 Project의 정보가 포함됩니다.**  
따라서, **User는 리소스를 사용하기 위해 Project에 대한 Role 할당이 필요합니다.**

#### 도메인 범위 Token(Domain-Scoped Token)

도메인 범위 Token은 Domain의 컨텐츠 또는 Domain 자체에서 작동할 수 있는 권한을 나타냅니다. 즉, Domain에 대한 권한이 있는 User는 Domain 내에서 작업을 관리할 수 있습니다. 예를 들어, Domain 관리자는 해당 Domain 내에 새로운 User 및 Project를 생성할 수 있습니다. 추가적으로, **도메인 범위 Token은 서비스 카탈로그, Role 및 Domain에 대한 정보가 포함됩니다.**  
따라서, **User와 Project를 관리해야하는 사람들은 일반적으로 도메인에 대한 권한이 필요합니다.**
</br></br>

### 토큰 공급자(Token Providers)

#### UUID Token

UUID는 현재 기본 Token 제공자이며 32바이트의 길이로 이루어져 있고 백엔드에서 유지되어야 합니다.

Client는 Identity 서비스에 UUID Token을 전달해야 유효성을 확인할 수 있습니다.
</br></br>

#### Fernet Token

Fernet Token 형식은 Openstack Kilo 릴리즈에서 도입되었습니다.

다른 Token 유형과는 달리 Fernet Token은 백엔드에서 유지될 필요가 없습니다.

AES256암호화는 Token에 저장된 정보를 보호하는데 사용되며, 무결성은 서명으로 확인됩니다.

Identity 서비스만 Fernet Token을 암호화하고 해독하는데 사용되는 Key에 접근할 수 있어야하고, UUID Token과 마찬가지로 Fernet Token은 유효성을 검사하기 위해 Identity 서비스로 전달되어야 합니다.

#### PKI 및 PKIZ Token

PKI Token은 인증 Context와 서비스 카탈로그를 포함하는 문서(서명이 되어 있음)라고 생각하면 됩니다.  

Openstack 배포 크기에 따라 Token이 매우 길어질 수 있습니다.

Identity 서비스는 PKI Token을 만들고 유효성을 검사하기 위해 Public Key, Private Key 쌍과 인증서를 사용합니다.

PKIZ Token은 기본적으로 PKI Token과 유사하나, PKIZ Token은 PKI의 크기 문제를 완화하기 위해 압축되어 제공된다는 점이 다릅니다.
