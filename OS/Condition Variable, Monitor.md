## Condition Variable

**특정 조건이 만족될 때까지 스레드의 실행을 일시 정지시켜 대기할 수 있는 공간을 제공**

> 단독으로는 사용할 수 없고, **보통 mutex와 함께 Monitor라는 이름으로 사용**
> 

| **구성 요소** | **설명** |
| --- | --- |
| **대기 큐 (Wait Queue)** | wait() 호출 시 스레드가 등록됨→ 스레드는 sleep 상태로 전환됨 |
| **signal()/broadcast()** | 대기 큐에서 하나 또는 모든 스레드를 꺼내 깨움 |
| **mutex 연동** | wait(cond, mutex) 형태로 항상 락과 함께 사용→ wait() 시 mutex는 반납, 깨어난 후 재획득 |

특징

- 내부적으로 대기 큐를 갖고있어, wait이 호출된 스레드들을 여기서 대기시키고, signal이 호출되면 대기 큐에서 대기중이던 스레드를 깨움
- 세마포어는 내부적으로 **정수값을 기억하기 때문에** signal()이 누적될 수 있지만, 조건 변수는 내부적으로 “상태”를 저장하지 않기 때문에 signal 호출 당시 **대기 중인 스레드가 없으면 아무 효과 없음**

```cpp
// 구조 (POSIX 스타일)
pthread_mutex_t lock;
pthread_cond_t cond;

pthread_mutex_lock(&lock);

while (!condition) {
    pthread_cond_wait(&cond, &lock);  // 조건 만족할 때까지 대기 + mutex 반환
}

// 조건이 만족된 후 작업 수행
작업 수행...

pthread_mutex_unlock(&lock);
```

`pthread_cond_wait` 는 

- **조건 변수 cond에 대해 기다리는 동안 mutex를 잠시 반납**하고,
- 깨워질 때 다시 mutex를 **자동으로 재획득**해서 실행을 이어감

헷갈리지 말아야 할 것은 **조건 변수는 상태(조건)를 저장하지 않고, 단지 조건이 만족될 때까지 기다리는 대기 큐 역할일 뿐임**

> 논리적 조건 그 자체는 while문 안의 변수(위 코드에선 `condition`) 가 갖는것임
따라서 이것(**조건의 상태)은 개발자가 따로 만들어서 관리**해야 함 (보통 boolean, int, queue 상태 등)
> 

항상 위와 같은 구조로 while문으로 조건을 재확인하도록 해야하고, mutex를 통해 이 조건을 보호해야 함

- while문 들어가기 전 mutex를 잡는 이유는?
    
    조건이 만족되어 pthread_cond_wait를 호출하려고 하기 직전 context switching이 일어나면 다른 스레드가 이 조건을 변경할 수 있으므로
    
- if문이 아닌 while문으로 조건 변수를 감싸야 하는 이유는?
    
    스레드가 깨어났다고 해서 조건이 항상 만족된 것은 아니기 때문 → 조건을 다시 확인하는 while 루프가 필요   
    시스템 내부적인 이유(커널 이벤트, 인터럽트 등) **때문에 signal 없이도 조건이 만족할때 까지 대기해야 하는 스레드가 예상치 않게 일어날 수 있기 때문에 (spurious wakeup) 항상 다시 한번 조건을 확인하도록 while문을 사용해야 함**
    
    > 여기서 말하는 시스템 내부적인 이유는 **예측 불가능한 외부 이벤트를 말함   
    예를 들어 인터럽트(I/O 이벤트, 타이머 인터럽트)는 프로그램의 실행 흐름과 무관하게 언제든지 발생할 수 있음**
    > 
    
    또한, broadcast로 모든 스레드를 깨우는 경우 선착순 하나만 조건을 만족하고, 나머지는 다시 wait해야 함
    

signal()은 **“조건이 만족되었다”는 뜻이 아니라**, **“조건이 만족됐을 수도 있으니 다시 확인해 봐”라는 신호일 뿐이라고 생각하고,** 무조건 **while (!condition)로 감싸서 재확인하는 것**이 안전하고 정석적인 방법

## Monitor (`synchronized`)

위 코드와 같이 뮤텍스와 조건 변수를 같이 사용하는 구조를 모니터라고 함

(위 코드 구조 자체가 모니터의 내부 구현 형태)

자바에서는 위 코드와 같이 개발자가 직접 뮤텍스를 다루면 실수할 가능성이 크기 때문에, 이를 더 안전하고 편리하게 사용할 수 있도록 `synchronized` 라는 키워드로 추상화하여 제공

> 자바에서는 모든 객체의 조상인 `Object` 클래스가 모니터를 내장하고 있음 (wait, notify, notifyAll 메서드 보유)
> 

```java
synchronized (obj) {
    while (!condition) {
        obj.wait();      // condition variable처럼 동작
    }
    // 조건 만족
    ...
    obj.notify();        // 다른 스레드 깨우기
}
```

- 여기서 obj는 모니터 역할을 하고
- wait(), notify()는 내부적으로 **조건 변수와 유사한 역할**
- synchronized 블록이 **뮤텍스 역할을 하는 monitor lock**을 자동으로 처리

| **개념** | **구성** | **자바에서 대응** |
| --- | --- | --- |
| **Monitor** | Mutex + Condition Variable | synchronized + wait()/notify() |
| **Mutex** | 임계 영역 보호 | synchronized 블록 |
| **Condition Variable** | 조건 만족까지 대기 | Object.wait(), notify() 등 |

POSIX 스타일에서 뮤텍스 획득을 대기하는 큐와 조건 변수 내부의 조건이 충족될 때 까지 대기하는 큐가 있는 것과 동일하게, 모니터에 entry queue(뮤텍스를 위한 큐)와 wait queue(조건 변수를 위한 큐)가 존재

만약 POSIX 스타일 처럼 뮤텍스와 조건 변수를 명시적으로 사용하고 싶다면, 자바에선 `ReentrantLock`과 `Condition` 을 사용하여 `synchronized`  보다 세밀하게 직접 구현할 수 있다

```java
Lock lock = new ReentrantLock();
Condition cond = lock.newCondition();

lock.lock();
try {
    while (!condition) {
        cond.await();
    }
    ...
    cond.signal();
} finally {
    lock.unlock();
}
```

## 생산자 - 소비자 문제

생산자(Producer): 데이터를 생성해서 **공유 버퍼에 넣음**

소비자(Consumer): 공유 버퍼에서 데이터를 꺼내서 처리하는 구조

문제점

1. 공유 자원이므로 생산자와 소비자가 동시에 버퍼를 수정하려 하면 경쟁 상태가 발생할 수 있음

> mutex를 통해 해결
> 
2. 공유 버퍼의 사이즈가 한정되어 있으므로 생산자는 버퍼가 가득 찼는데 계속 넣으려 하면 안되고, 소비자는 버퍼가 비었는데 계속 꺼내려 하면 안됨

> 세마포어 혹은 조건 변수로 해결
> 

세마포어 기반 구현

![producer-consumer](https://github.com/user-attachments/assets/cb0b7462-51a8-47e7-a139-74304c713e90)


slots는 버퍼의 빈 공간의 사이즈를 의미, slots가 0이면 버퍼가 가득 찼다는 의미이므로 생산자 대기

items는 버퍼에 담긴 아이템의 개수를 의미, items가 0이면 버퍼가 비었다는 의미이므로 소비자 대기

조건 변수 기반 구현

```java
while (buffer.isFull()) {
    notFull.await();  // 생산자는 버퍼가 찰 때까지 대기
}
buffer.put(item);
notEmpty.signal();     // 소비자에게 알림
```
