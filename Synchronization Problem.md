<h1><center> 멀티 프로세스 & 멀티 스레드 환경에서 동기화 문제 해결방법 </center></h1>

###### tags: `💻 TIL`
###### date: `2024-02-05T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Nossi.DEV](https://www.nossi.dev/02a22f24-4b97-4a93-8aa3-951f11c94187)

## 임계영역(Critical Section)

![image](https://hackmd.io/_uploads/BkoSLRTc6.png)

둘 이상의 프로세스/스레드가 동시에 동일한 자원에 접근하도록 하는 프로그램 코드 부분을 의미한다. 

중요한 특징은, 한 프로세스/스레드가 자신의 임계구역에서 수행하는 동안에는 다른 프로세스/스레드들은 그들의 임계구역에 들어갈 수 없어야 한다는 것이다. 즉, 임계구역 내의 코드는 원자적(atomically)으로 실행되야 한다. 

원자적으로 실행되기 위해 각각의 프로세스/스레드는 자신의 임계구역으로 진입하려는 진입 허가를 요청해야 한다. 이 부분을 `Entry Section`이라고 하고, 진입이 허가되면 임계구역을 실행할 수 있다. 

임계 구역이 끝나고 나면 `Exit Section`으로 퇴출되게 된다. 임계구역의 원자성을 보장하여 프로세스/스레드들이 동기화되도록 할 수 있다. 

## 동기화 문제

> 동기화 문제는 여러 개의 실행 단위 간에 데이터나 자원에 대한 접근이 겹치면서 발생할 수 있는 문제를 말한다. 

**예시)**

![image](https://hackmd.io/_uploads/HJkxHppqa.png)

`count++`를 CPU입장에서 분해해보면 3개의 atomic operations로 나뉘게 된다. 

1. `count` 변수의 값을 가져온다.
2. `count` 변수의 값을 1 증가시킨다.
3. 변경된 `count` 값을 저장한다. 

CPU는 atomic operation을 연산하게 된다. 따라서 `count++`를 하기 위해 3번의 연산을 한다. 

시분할 시스템으로 작동하는 멀티 프로세스/멀티 스레드 시스템에서, 두 개의 스레드가 동일한 데이터 `count`에 접근한 상황을 가정해보자. 

thread1에서 `count++`를 하고, thread2에서도 `count++`를 한다면 그 실행 결과가 접근이 발생한 순서에 따라 달라질 수 있다. 이를 `Race Condition`이라고 한다. 

즉, 둘 이상의 스레드가 동일한 자원에 접근해 조작할 경우, 그 결괏값이 접근이 발생한 순서에 따라 달라지는 Race Condition에 의해 동기화 문제가 발생할 수 있다. Race Condition으로부터 동기화 문제를 보호하기 위해 한 순서에 하나의 프로세스/스레드만 해당 자원에 접근하고 조작할 수 있도록 보장해야 한다. 

Swift에서는 공유 자원의 타입을 Thread Safe하게 만들면 된다. 

**NOTE**

> 시분할 시스템: 컴퓨터 시스템의 동작 방식 중 하나로, 다수의 작업이 공존하면서 각 작업에 일정 시간씩 번갈아 가며 CPU를 할당하는 시스템을 말한다. 

## 멀티 프로세스 & 멀티 스레드 환경에서 동기화 문제 해결방법

### 뮤텍스(Mutex)

동기화 방법 중 하나로 Mutual Exclusion의 약어이다. 공유자원에 접근할 수 있는 프로세스/스레드의 수를 1개로 제한한다. 임계영역을 보호하고, Race Condition을 방지하기 위해 Mutex Lock을 사용한다. 

즉, 프로세스/스레드는 임계구역에 들어가기 전에 반드시 Lock을 획득해야 하고, 임계구역을 빠져나올 때, 반환해야 한다. 

**NSLock**

```swift 
import Foundation

// 뮤텍스 락 생성
let myLock = NSLock()

// 임계 구역에 들어가기 전에 락 획득
myLock.lock()

// 임계 구역에서의 작업 수행
// ...

// 임계 구역을 빠져나올 때 락 반환
myLock.unlock()

```

**DispatchQueue**

```swift 
import Foundation

let myQueue = DispatchQueue(label: "com.example.myQueue")

// 임계 구역에 들어가기 전에 DispatchQueue를 사용하여 동기화
myQueue.sync {
    // 임계 구역에서의 작업 수행
    // ...
}

```

Busy waiting은 다른 프로세스/스레드가 사용할 수 있는 CPU를 낭비한다는 단점이 있다.

![image](https://hackmd.io/_uploads/ByNPPCa5T.png)

### Semaphore

Mutex와 가장 큰 차이점은 공유 자원에 접근할 수 있는 프로세스/스레드의 개수가 2개 이상이 될 수 있다는 점이다. 

세마포어 변수 S에 동시에 접근 가능한 프로세스/스레드의 개수를 저장한다. S가 0보다 크면 임계구역으로 들어갈 수 있고, 임계구역에 들어가면 S값을 1 감소시킨다. S값이 0이 되면 다른 프로세스/스레드가 임계구역으로 접근할 수 없다. 임계구역에서의 작업이 끝나고 임계영역에서 exit하면서 S값을 1증가 시킨다.

![image](https://hackmd.io/_uploads/S1MW_A6cT.png)

세마포어 값이 0,1만 가질 수 있는 경우를 Binary Semaphore라고 하는데, 이는 Mutex와 거의 유사하게 작동한다.

```swift 

import Foundation

// 세마포어 생성, 초기값은 동시에 접근 가능한 프로세스/스레드의 개수를 나타냄
let semaphore = DispatchSemaphore(value: 1)

// 임계 구역에 들어가기 위해 세마포어를 기다림
semaphore.wait()

// 임계 구역에서의 작업 수행
print("Entering critical section")

// 임계 구역을 빠져나오면서 세마포어 값을 1 증가시킴
semaphore.signal()

print("Exiting critical section")

```

