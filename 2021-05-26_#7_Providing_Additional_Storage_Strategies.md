# Definition 

Object Storage를 사용할 수 있도록 돕는 Swift, Ceph RGW, Amazon S3에 대하여 배운다. 
최초 개발자들이 배포 소스를 빠르게 공유하기 위해서 만들었음. 

# Character 

* Data 일관성 : 강력한 일관성은 최신 데이터를 제공하지만 느린 속도를 보인다. 

* Data 복제

* Data 암호화 : Swift(TLS를 통한 암호화 통신), Ceph(Barbican을 통한 암호화 통신), Amazone S3(HTTPS + TLS)

* Data 만료 : 만료 날짜를 명시하고, 특정 시간이 지나면 사라짐. 

* Data 한계 : 단일 파일의 크기는 5GiB로 제한이 있음. 

