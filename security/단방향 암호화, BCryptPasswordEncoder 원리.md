## 단방향 암호화

**비밀번호를 해시로 변환하고, 원래 값을 복호화할 수 없도록 설계된 암호화 알고리즘**

> “단방향 암호화”라고 부르는 대부분의 알고리즘은 **해시 함수** 기반
> 

| **항목** | **단방향 암호화 (해시)** | **양방향 암호화** |
| --- | --- | --- |
| 목적 | 비밀번호 저장, 무결성 확인 | 데이터 암·복호화 (예: 카드번호) |
| 대표 알고리즘 | bcrypt, PBKDF2, Argon2, SHA | AES, RSA |
| 복호화 가능 여부 | ❌ 불가능 | ✅ 가능 |
| 예시 | 로그인 시 비밀번호 검증 | API 통신 시 데이터 암호화 |

## **무차별 공격(Brute-force attack)의 정의**

**암호화된 비밀번호(해시값)를 보고, 원래의 평문 비밀번호를 추측하려는 시도**

> 즉, **“해시 → 평문”을 역으로 찾는 행위
복호화가 불가능한 단방향 해시이기 때문에**, 가능한 모든 후보를 넣어보는 방식으로만 가능
> 

## BCryptPasswordEncoder 핵심 원리

기존 SHA-256 암호화 방식이 빠르고 간단하지만 GPU로 brute force 공격에 취약하다는 단점이 있음

Bcrypt는 다음과 같은 특징으로 이를 보완

- Salt 자동 포함

> 입력된 평문에 대해 **매번 새로운 salt(무작위 문자열)** 를 자동으로 생성해서 추가
동일한 비밀번호여도 해시 결과가 매번 다르기 때문에   
> 자주 사용될것 같은 비밀번호를 미리 해싱해둔 Rainbow Table이 무용지물이 되고 모든 해시값을 해당 salt로 다시 계산해야 함
> 
- cost factor

> 해시 연산을 의도적으로 느리게 만들어 brute force 공격 방지
cost 값을 높이면 내부 반복 횟수가 증가함 (기본: 10) → 2^cost 번의 연산이 수행됨
해시 1회 연산 시간이 조금만 늘어나도 공격자 입장에선 수억 개의 비밀번호를 해싱해야 하므로 큰 영향
> 
- **Blowfish 기반 key stretching**

> 내부적으로 **Blowfish 암호 알고리즘을 변형**해서 사용, 단순 해시보다 더 복잡한 연산 과정을 통해 보안성 향상
> 

## BcryptPasswordEncoder 사용 흐름

```java
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String hash = encoder.encode("myPassword");
```

`BCryptPasswordEncoder` 는 내부적으로 랜덤 salt을 생성하고 평문 + salt를 해싱

salt, cost, hash가 합쳐진 문자열로 저장됨

```java
$2a$10$KbQURflL3CNdEf.FF9RO7epO.kZ/OmY1hDPcdobLQhRMxoH0iRW.a
```

| **파트** | **설명** |
| --- | --- |
| $2a$ | bcrypt 알고리즘 버전 |
| 10 | cost factor (2^10 = 1024번 반복 연산) |
| KbQU...RO7e | **salt** (22자 base64 인코딩) |
| pO.kZ/... | **해시 결과** (암호화된 비밀번호) |

이후 비밀번호를 검증할 땐

```java
encoder.matches("입력된 평문", "저장된 해시")
```

matches()는 저장된 해시에서 salt와 cost를 추출해서 동일한 방식으로 평문을 재해시 → **두 해시를 비교**

- **서버나 공격자나 둘 다 평문 → 해시 연산은 동일하게 해야 하는데, 왜 salt나 cost가 공격자에게는 더 큰 부담이 되는가?**
    
    cost와 salt는 결국 “해시 1번의 비용을 높이는 전략”이지만, 정상 사용자는 이를 1번만 하면 되는걸 공격자는 수천만 번 반복해야 하기 때문에 비용이 기하급수적으로 늘어남
