## Critical Section(임계 영역)이란?

**두 개 이상의 스레드가 동시에 접근하면 문제가 발생할 수 있는 코드 영역**

> 즉, **공유 자원에 접근하는 코드 영역**
> 

```java
class Counter {
    private int count = 0;

    public void increment() {
        count++; // 이 한 줄이 임계 영역!
    }
}
```

## Race Condition(경쟁 상태)이란?

**여러 개의 스레드 또는 프로세스가 동시에 공유 자원에 접근할 때, 실행 순서에 따라 결과가 달라질 수 있는 상황   
(적절한 동기화가 이루어지지 않은 상황)**

> 즉, 여러 스레드/프로세스가 임계 영역에 동시에 접근하여 공유 자원에 쓰기 작업을 수행하여 예상치 못한 버그가 발생할 수 있는 상황
> 

- 프로세스 간에도 경쟁 상태가 발생할 수 있는가?
    
    가능하다. 비록 **OS가 프로세스의 주소 공간을 분리해서 메모리 침범은 막아주지만**, 공유 자원(공유 메모리, 파일, 소켓, DB, IPC 등)을 통해 간접적으로는 **서로 영향을 줄 수 있기 때문**
    

## Mutual Exclusion(상호 배제) 이란?

임계 영역에 한번에 단 하나의 스레드/프로세스만 접근할 수 있도록 보장하는 것 (여러 실행 흐름이 동시에 공유 자원에 접근하지 못하도록 하는 제어)

- 동기화와 상호 배제의 차이는?
    
     **Mutual Exclusion ⊂ Synchronization**
    
    - Mutual Exclusion은 “한 번에 하나만 접근”이라는 **접근 제어 중심 개념**
    - Synchronization은 더 넓게 **실행 순서, 가시성, 동시성 전체를 다루는 개념**
  
    
    | **구분** | **설명** | **관계** |
    | --- | --- | --- |
    | **동기화 (Synchronization)** | 여러 실행 흐름(스레드, 프로세스 등)의 **실행 순서를 제어**하여 **정합성을 보장**하는 기법 전체 | 상위 개념 |
    | **상호 배제 (Mutual Exclusion)** | 여러 실행 흐름이 **동시에 공유 자원에 접근하지 못하게** 하는 제어 방식 | 동기화의 한 종류 |
    
    동기화는 상호 배제 말고도 volatile (Java에서 메모리 가시성 보장), Thread.join() (다른 쓰레드 종료 후 진행), Thread.sleep() (단순 대기), memory barrier (JVM 내부, OS, CPU 수준의 순서 제어) 방식으로 가능
    
    ```java
    volatile boolean flag = false;
    
    Thread A:
        data = 100;
        flag = true; // write to flag happens-after write to data
    
    Thread B:
        if (flag) {
            print(data); // data == 100 보장됨
        }
    ```
    

## 상황 별 동기화 도구

| **목적** | **대표 도구** | **설명** |
| --- | --- | --- |
| 동시에 접근 못하게 막기 | synchronized, Lock | 상호 배제 |
| 작업 순서 보장 | volatile, join, memory barrier | 가시성 및 happens-before |
| 조건 만족 시까지 대기 | wait/notify, Condition, Latch | 조건 동기화 |
| 일정 수량만 허용 | Semaphore | 자원 제한 |
| 비동기 간 통신 | BlockingQueue, Future, Kafka | 메시지 기반 동기화 |

## Dead Lock이란?

서로가 상대방이 자원을 내놓길 기다리면서 무기한 연기 상태(교착 상태)에 걸리는 것

## SpinLock
<img width="901" alt="스크린샷 2025-05-01 오후 8 37 22" src="https://github.com/user-attachments/assets/484f61eb-a402-4627-9f3a-066a0e6d5530" />

> 출처: [유튜브 쉬운코딩](https://www.youtube.com/watch?v=gTkvX2Awj6g)
> 

`TestAndSet` 메소드에서 일단 기존의 lock 값을 oldLock에 저장하고, 기존 lock 값과 관계 없이 일단 lock 값을 1로 바꿈. 

그리고 기존 lock값인 oldLock을 리턴하기 때문에 만약 기존 lock값이 0이었다면 while 문을 빠져나와 임계영역에 진입할 수 있는거고, 기존 lock 값이 1(이미 누가 락을 획득한 상태)이면 계속 while문에서 락을 획득할 때 까지 spin

이때 이 `TestAndSet`메소드는 cpu의 도움을 받아 원자적으로 처리되기 때문에 공유 변수인 lock 값을 동시에 읽어들이는 동시성 문제는 걱정할 필요 없음

단점

- 자신의 time slice동안 락이 해제될 때 까지 계속 CPU 자원을 점유한 채 무의미한 스핀을 하므로 CPU 자원을 효율적으로 사용하지 못함 (busy waiting)

싱글코어 상황에선 반드시 선점형 스케줄러여야 함(그렇지 않으면 스핀락을 돌고 있는 스레드가 CPU 자원을 획득하면 이를 놔주지 않고 무한 루프에 빠짐)

> 멀티 프로세서 상황에선 락 보유 스레드와 락 대기 스레드가 각각 다른 프로세서에서 실행될 수 있으므로 임계 영역이 짧다면 잠깐의 스핀 후 바로 락을 획득하는 것이 context switching 비용보다 나을 수 있음
> 


## Mutex

앞서 확인한 스핀락의 비효율적인 CPU 사용을 개선하고자, 스핀락과 같이 락을 획득하지 못하면 busy waiting을 하는게 아니라 현재 스레드를 OS가 관리하는 대기 큐에 넣고 sleep 상태로 전환되어 CPU 자원을 양보

락 보유 스레드가 `pthread_mutex_unlock()` 호출 시 커널이 대기 큐에 대기중인 스레드 하나를 깨워 다시 락 획득 기회를 준다

특징

- 상태가 오직 lock/unlock만 있으므로 Boolean 변수를 활용하여 lock, unlock 명령어를 제공한다

•  Lock(m) : mutex m이 unlock 상태면 lock하고 바로 리턴, locked 상태면 unlock될 때까지 대기

•  Unlock(m) : mutex m을 unlock상태로 변경 (mutex가 locked상태일때만 호출할 수 있음, unlock 상태일때 호출 시 error 리턴)


## Semaphore

뮤텍스와 다르게 양수인 정수를 상태로 가진다. 따라서 하나의 스레드만 접근을 허용하는 뮤텍스와 달리 상태값이 음수가 되기 전까지는 임계 영역에 접근이 가능하다 (여러 스레드의 접근을 허용한다)

•  **P(S) (wait/down)** : S를 1 감소시키고 리턴하는 함수, S는 non-negative 변수이므로 S가 0이면 V를 통해 S가 양수가 될 때 까지 기다린 후 1 감소시킴

•  **V(S) (signal/up)** : S를 1 증가시키고 리턴하는 함수, 만약 P 함수로 대기하고 있는 스레드들이 있다면 이 중 한 스레드를 깨움 -> 구현에 따라 다를 수 있지만 보통 wait queue의 첫번째 스레드를 깨움   


## Mutex와 Binary Semaphore는 같은 것인가?

비슷한 기능을 하지만, 차이점이 있다

- Mutex는 락을 보유한 스레드만 unlock이 가능하지만, 이진 세마포어는 누구나 signal을 호출하여 해제할 수 있다

**이진 세마포어**는 일반적으로 **동기화 이벤트 전달**에 사용됨

- 예: 한 스레드가 어떤 조건을 만족했음을 다른 스레드에게 알려주는 목적

> 즉, **“진입 제어”보다는 “동작 순서 제어”에 초점**
>     

**뮤텍스**는 단순하고 명확하게 **임계 영역 보호**에만 집중

| **개념** | **비유** |
| --- | --- |
| **Binary Semaphore** | 신호등 (지금 들어가도 되는지 단순히 알려주는 역할) |
| **Mutex** | 문 잠금 장치 (누가 잠갔는지 아는 상태, 주인만 열 수 있음) |
