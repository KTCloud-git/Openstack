# Openstack Tenent 및 Identity 관리

</br>
<h2>목차</h2>

- Keystone은 무엇인가?
- Region / Domain / Project의 개념
</br>

---

</br>
<h2> Keystone은 무엇인가?</h2>

</br>

Keystone은 Openstack의 인증을 담당하고 있습니다.  
어떤 서비스를 설치하더라도 가장 먼저 설치가 되는 가장 중요한 서비스이며, Keystone이 없으면 그 어떤 서비스도 이용할 수 없습니다.

</br>

따라서, 이렇게 생각을 해볼 수 있습니다.

</br>

Openstack이 집이라면 Keystone은 **현관문 키와 같은 기능**을 하는 것이 아닐까?  

</br>

Openstack이라는 집은 물리서버로 구성되어 있고, **집 안에는 Compute, Image, Network, Storage와 같은 자원들이 있다고 가정해봅시다.**

</br>

<img src="./img/house.png" width="400" height="400"> 

</br>

이러한 자원들이 우리가 평소에 사용하는 가전제품이라고 한다면  
Openstack이라는 집에 들어가야만 사용할 수 있습니다.

</br>

즉, 내돈내산 가전제품들을 집 주인과 손님만 사용할 수 있습니다.  
이렇듯 **Openstack이 제공하는 서비스들도 동일하게 아무나 사용할 수 없습니다.**

</br>

그렇기 때문에 집 주인과 손님이라는 인증을 받아야만 해당 가전제품(자원)들을 사용할 수 있는데,  
Keystone이 해당 부분을 담당하고 있다고 생각하면 이해가 편합니다.

</br>

**결과적으로 Keystone은 사용자 인증을 통하여 물리 서버내의 자원을 사용할 수 있도록 관리합니다.**

</br>
</br>
</br>

**<h3>Logical Architecture로 보는 Keystone</h3>**

</br>

이번에는 논리적인 구조를 통해 Keystone에 대해 한번 더 알아보겠습니다.  
* Keystone은 **서비스 및 관리자 API, Token Backend, Catalog Backend, Policy Backend, Identity Backend**로 구성이 됩니다.

</br>

<img src="./img/logical architecture.png" width="600" height="300"> 

</br>

+ **Token Backend** : 사용자 별 Token 관리
+ **Catalog Backend** : Openstack 모든 서비스의 End-Point URL 관리
+ **Policy Backend** : Tenent, 사용자 계정 및 Role 등을 관리
+ **Identity Backend** : 전체적인 사용자 인증 관리

</br>
</br>

<h3>Openstack에서 Keystone의 위치</h3>

</br>

이번에는 Openstack에서 Keystone의 위치를 알아볼까 합니다.

</br>

<img src="./img/Keystone location.png" width="800" height="400"> 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjEzNDMxNjQ0MSwyMTI2NzE1MTIyLDE2Mj
M2ODU3NjIsNjk5MTA2MzQzLDE0Mzg0ODEzNzIsMTY1NDczNDY2
OSwtMTY1NjU0MTY3OSw2NDk3MDMwNiwyODQ1MzM3NTUsMzQyMz
gzMjEwLDEyMTM3NTE0NDQsMTUwMTc5MDgzMF19
-->