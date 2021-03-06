---
title: "Domain에 CNAME, MX Record를 같이 등록하면서 겪은 이슈"
date: 2021-04-14T18:15:10+09:00
description: ""
tags: [
	"dns",
	"cname",
	"mx",
	"domain",
	"rfc974",
	"rfc2181",
	"도메인"
]
categories: []
disableComments: false
  
---

기존에 사용하고 있던 도메인에 Record를 추가하면서 겪었던 이슈에 대해 기록해보려고한다.

최소한 아래 5가지에 대한 개념은 알고 있어야하기에 간단히 짚고 넘어가보자.

1. **Domain**  
   인터넷에 연결되어있는 장치(컴퓨터, 서버, 스마트폰 등)를 식별할 수 있는 주소인 IP를 사람이 이해하고 기억하기 어렵기때문에 IP에 이름을 부여하여 쉽게 사용할 수 있도록하였는데 이때 부여한 이름이 도메인이다.
   e.g) google.co.kr, naver.com

2. **DNS**  
   Domain Name System으로 전화번호부라고 생각하면된다. 도메인과 IP의 쌍(pair)을 가지고있어 서로 변환하는 역할을 한다.

   도메인-IP에 대한 쌍을 하나의 Record(Record)라 한다.

   - Forward Zone (도메인 -> IP)
   - Reverse Zone (IP -> 도메인)

3. **CNAME Record**  
   Canonical Name의 약자로 도메인을 또 다른 도메인 주소로 매핑 시키는 형태이다.

4. **MX Record**  
   Mail Exchanger Record로 메일이 수신될 위치를 결정하는 Record다. MX Record에는 우선순위와 도메인을 설정할 수 있다.

5. **nslookup**  

   네트워크 관리 명령 줄 인터페이스 도구로서 많은 컴퓨터 운영 체제에서 사용 가능하며, 도메인 네임을 얻거나 IP 주소 매핑 또는 다른 특정한 DNS Record를 도메인 네임 시스템에 질의할 때 사용한다.

   

------



이제 본론으로 들어가서 문제를 겪었던 상황을 A.co.kr, B.co.kr 도메인으로 예로들어 적어보려한다.

- **주어진 상황**

  - A.co.kr, B.co.kr 모두 AWS에서 route53에서 co.kr에대한 도메인이 구매가 불가능하여, 다른 곳에서 구매한 도메인임.
  - A.co.kr의 DNS에는 MX Record가 세팅되어 사용 중임.
  - B.co.kr의 DNS에는 CNAME Record에  AWS ELB DNS name이 세팅되어 사용 중임.  
    ![dns_cname_mx_Record_route_list](https://hkyeong0703.github.io/posts/images/dns_cname_mx_record_route_list.png)

- **하고자 했던 것**
  
  - A.co.kr로 접속시 B.co.kr로 접속 했을 때와 동일한 페이지로 연결되도록하고 싶었음.
  
- **적용했던 방법**

  - A.co.kr DNS에 CNAME Record로 B.co.kr 등록

- **벌어진 상황**

  - A.co.kr로 접속시 B.co.kr로 접속 했을 때와 동일한 페이지를 볼 수 있었음. (원하던 결과 O)

  - 그러나 A.co.kr 메일 계정으로 발송한 메일 중 일부가 B.co.kr의 메일 계정으로 발송되는 문제 발생. (원하던 결과 X)

문제 상황을 파악하기위해 nslookup을 이용해 확인을 해보았다.

```shell
nslookup -querytype=MX A.co.kr

None-authoritative answer:
A.co.kr canonical name = B.co.kr
B.co.kr mail exchanger = 1 mail9.example.com
B.co.kr mail exchanger = 10 mail7.example.com
B.co.kr mail exchanger = 5 mail8.example.com
```

```shell
nslookup -querytype=MX A.co.kr

None-authoritative answer:
A.co.kr mail exchanger = 1 mail.example.com
A.co.kr mail exchanger = 10 mail3.example.com
A.co.kr mail exchanger = 5 mail2.example.com
```

의도했던대로 A.co.kr에 등록된 메일서버로 조회가 될 때도 있었지만, 간헐적으로 CNAME Record가 먼저 조회되어 B.co.kr에 MX Record로 등록된 메일서버가 조회 되고있었다🤯🤯🤯🤯



구글링을 시작했다...**CNAME Record를 사용할 때의 주의 사항**과 **CNAME Record와 MX Record를 같이 사용했을 때**의 문제점들이 발견되었다.

- **CNAME을 사용할 때 주의 사항** 

  - DNS 네임스페이스의 최상위 노드에 대한 CNAME Record를 생성할 수 없다.
  - 하위 도메인에 대한 CNAME Record를 생성하면, 그 하위 도메인에 대해서는 다른 Record를 생성할 수 없다.
  - 참고
    - [CNAME Record 형식](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/ResourceRecordTypes.html#CNAMEFormat)
    - [RFC 2181, Clarifications to the DNS ](https://tools.ietf.org/html/rfc2181)Specification, section 10.1

- **CNAME Record와 MX Record를 같이 사용했을 때의 문제점** 

  - RFC 문서에 MX Record와 CNAME 연동에대해 주의 또는 제한을 둔 글이 없기때문에 명확하게 안 된다고 말할 수 없지만, 불필요한 쿼리들이 발생되는 트래픽을 사유로 오랜 기간 잘못된 사용으로 인식되어 왔고, 대부분 관련 문서에서 사용하지말기를 권고하고있음.

    - 참고
      - [Powered by DNS 6.3 MX와 CNAME 연동](http://wiki.kldp.org/KoreanDoc/html/PoweredByDNS-KLDP/mx-and-cname.html)
      - [rfc974](https://tools.ietf.org/html/rfc974)
    - [RFC 2181, Clarifications to the DNS ](https://tools.ietf.org/html/rfc2181)Specification, section 10.3
  
  
  

A.co.kr에 MX Record로 등록된 메일 서버를 계속 이용해야했기에 CNAME Record를 사용할 수 없는 것이었다...😭

<br />

------



이제 해결 방안을 찾아보았다. 3가지정도가 있었다.

1. **네임 서버를 옮겨 AWS Route53 Hosted Zones을 이용하여 AWS ELB를 ARecord alias로 등록**  
   참고: [Creating a public hosted zone](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingHostedZone.html)  
   [Making Route 53 the DNS service for a domain that's in use](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html)
2. AWS ELB에 NLB를 사용하여 고정 IP를 부여하여 A Record로 등록  
   참고: [Create a Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-network-load-balancer.html)
3. **AWS global accelerator 사용**  
   참고: [AWS Global Accelerator](https://aws.amazon.com/ko/blogs/korea/new-aws-global-accelerator-for-availability-and-performance/)

주로 AWS를 사용하고있기에 앞으로 유지보수를 위해 1번을 택해 해결하였다.





Route53에 Record를 옮기는 과정에서 MX Record를 등록한 상태에서 CNAME Record를 등록해보았더니 아래 사진과 같은 에러가 노출되고 등록이 불가했다... ![dns_cname_mx_Record_error](https://hkyeong0703.github.io/posts/images/dns_cname_mx_record_error.png)  

모든 서비스에서 등록이 불가하도록 처리가되었다면 삽질을 안했어도 됐을텐데...  
앞으로 이 기회를 통해 CNAME Record 사용을 최대한 자제할 것 같다.





