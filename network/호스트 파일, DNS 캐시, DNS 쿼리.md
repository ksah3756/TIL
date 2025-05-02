브라우저에 url 입력 시 일어나는 일들

```
브라우저 입력
↓
[1] 호스트 파일 확인
↓
[2] DNS 캐시 확인
↓
[3] DNS 서버로 쿼리 보내기
↓
IP 응답 받음
↓
서버 접속
```

## URL 구조

```
https://www.example.com:443/products/list?q=phone&page=2#section3
```

| **구성** | **값** |
| --- | --- |
| scheme | https |
| host | www.example.com |
| port | 443 (생략 가능) |
| path | /products/list |
| query | q=phone&page=2 |
| fragment | section3 (HTML 문서 내 앵커) |
- url과 uri의 차이는?
    
    URL ⊂ URI    
    URI 는 추상적/물리적 리소스를 식별하기 위한 식별자(Identifier)로써 정의에 초점이 맞춰져있다면,
    
    URL 은 인터넷환경에서 접근가능한 문자열 형태의 표현 Locators(지시자)에 초점

## 호스트 파일

OS에 설치된 로컬 파일로, 도메인 명 → IP 주소를 수동으로 매핑하는 파일

```
파일 위치
	•	Windows: C:\Windows\System32\drivers\etc\hosts
	•	Linux/Mac: /etc/hosts
```

가장 먼저 참조되기 때문에, 여기서 도메인이 악성 사이트의 IP로 매핑되면 큰 보안 사고가 일어날 수 있음

## DNS 캐시


**최근에 질의한 도메인 이름과 IP 매핑을 저장**해두는 임시 저장소

> 도메인 이름을 매번 DNS 서버에 질의하는 건 느리기 때문에, 일단 한 번 질의하고 나면, 그 결과를 메모리에 저장해두고 재사용하는 구조
> 

- 브라우저 자체 캐시 (ex: 크롬)
- OS 수준 캐시 (Windows DNS Client, Mac mDNSResponder 등)
- 로컬 DNS 서버 (회사 내부 DNS 등)

TTL(Time To Live)이 지나면 캐시가 자동으로 만료되고 재요청

## DNS 쿼리 과정


도메인이 매우 많으므로, 계층화되어 DNS 서버가 분리

![](https://velog.velcdn.com/images%2Fchun_gil%2Fpost%2F1fb63765-8762-4f93-a172-2d8f9fca0b54%2F210111_03.jpg)

![](https://velog.velcdn.com/images%2Fchun_gil%2Fpost%2Fdcd7f9e7-9bfa-42dc-9646-5ac89ba34683%2F210111_03_2.jpg)

Root DNS 서버는 전세계에 13개 존재
**ISP(ex. KT, SKB, LG U+ 등)가 제공하는 기본 DNS 서버 = 대부분 로컬 DNS 서버 역할**

설정을 통해 따로 public DNS 서버(ex. google 8.8.8.8 등)을 로컬 DNS 서버로 사용할 수 있음 (혹은 사내 DNS 서버)

> 출처: [https://velog.io/@chun_gil/DNS-서버-정의와-종류](https://velog.io/@chun_gil/DNS-%EC%84%9C%EB%B2%84-%EC%A0%95%EC%9D%98%EC%99%80-%EC%A2%85%EB%A5%98)
> 

| **질의 방식** | **설명** |
| --- | --- |
| Recursive Query (재귀 질의) | “IP를 직접 알려줘!” → 클라이언트는 **최종 IP를 받을 때까지** DNS 서버가 알아서 모든 과정을 처리하게 한다. |
| Iterative Query (반복 질의) | “내가 갈 수 있는 곳을 알려줘.” → 클라이언트가 중간 서버 정보를 받아서 다음 질의를 직접 이어서 보내야 한다. |

대부분 Recursive Query 방식으로 브라우저나 OS는 “난 몰라, IP를 완성해서 가져와”라고 DNS 리졸버에 맡긴다.
