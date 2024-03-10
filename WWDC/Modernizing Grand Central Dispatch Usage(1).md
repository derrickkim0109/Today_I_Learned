<h1><center> Modernizing Grand Central Dispatch Usage(1) </center></h1>

###### tags: `💻 WWDC 스터디`
###### date: `2023-06-13T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> [Blog](https://wlaxhrl.tistory.com/91)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요

`Application`에서 최상의 성능을 위해 `Grand Central Dispatch`를 활용하는 방법을 보여주기 위한 WWDC이다.

GCD는 `Application` 코드를 동적으로 확장할 수 있도록 설계되어 있습니다.
싱글 코어 Apple Watch에서 미니 코어 Mac까지 사용자가 어떤 종류의 하드웨어를 실행하고 있는지에 대해 너무 많이 걱정할 필요가 없습니다. 

**그러나 코드의 확장성과 효율성에 영향을 줄 수 있는 문제가 있다**

- 디스패치 비동기와 같은 GCD API를 사용해서 큐를 생성하고 작업을 시스템에 디스패치 할 수 있다.
    - 우리가 Grand Central Dispatch라고 부르는 동시성 기술에 대한 인터페이스 중 일부에 불과


## Efficiency Through Observation
> Going off core during an operation reduces efficiency 

코드가 작업을 완료하기 전에 코어에서 벗어나면 해당 코어가 구축한 기록을 더 이상 활용하지 못할 수 있다.

![](https://hackmd.io/_uploads/Byj5MoSv2.png)


## Parallelism and concurrency

병렬성과 동시성을 가장 잘 표현할 수 있는 방법에 대해 논의 해보자.!

Parallelism - 여러 코어가 필요하고 동시에 모두 사용하려는 것

Concurrency - 동시에 실행할 어플리케이션의 독립 구성 요소를 구성하는 방법에 관한 것
    - 단일 코어 시스템에서도 수행가능하다.
> Composition of independently executed tasks


### Parallelism
> Simultaneous execution of closely related computations

예시) 앱을 만들고 거대한 이미지를 처리한다고 상상해 보자. 

![](https://hackmd.io/_uploads/HJXZciBP2.png)


그리고 이러한 이미지를 더 빠르게 처리할 수 있도록 Mac Pro의 많은 코어를 활용할 수 있길 원한다면 내가 할 일은 그 이미지를 덩어리로 나누는 것이다.
그리고 각 코어가 해당 덩어리들을 병렬로 처리하도록 한다.

![](https://hackmd.io/_uploads/B17Z5irPh.png)

 코어가 이미지의 다른 부분들을 동시에 작업하기 때문에 속도가 빨라진다.
 
 **이걸 어떻게 구현해야 할까?**
 
 우선, 시스템 프레임워크를 활용할 수 있는지 여부를 고려해야 한다. 
 
 ![](https://hackmd.io/_uploads/BJHaqorwn.png)

### Parallelism with GCD

 예를 들어 Accelerate에는 고급 이미지 알고리즘의 병렬 실행을 기본적으로 지원한다.
 
 메탈, 코어의 이미지는 강력한 GPU를 활용할 수 있다.
 
 이것을 직접 구현하기로 결정했다면? 
 GCD를 사용하라.
 - 이 패턴을 쉽게 표현할 수 있는 tool을 제공한다.

이때 사용해야 하는 API는 GCD에서 병렬을 표현하는 `concurrentPerform`이다.

이렇게 하면 프레임워크가 모든 코어에서 병렬 계산을 수행하려 한다는 것을 알기 때문에 병렬 사례를 최적화 할 수 있다.

![](https://hackmd.io/_uploads/SJ1ijjSwn.png)

`concurrentPerform`은 시스템의 모든 코어에서 계산을 자동으로 부하의 정도를 균형 잡아 주는 병렬 for 루프이다.

```swift 
DispatchQueue.concurrentPerform(1000) { i in /* iteration i */ }
                                       
                                       
dispatch_apply(DISPATCH_APPLY_AUTO, 1000 ^(size_t i) { /* iteration i */ })
```

- `1000`은 시스템 전체에서 블록이 병렬로 호출되는 횟수이다.



![](https://hackmd.io/_uploads/SJ19niBP2.png)
- three-core system

```swift 
DispatchQueue.concurrentPerform(3) { i in /* iteration i */ }

```

> 여기에서는 3개의 블록이 3개의 코어에서 병렬로 실행되는 이상적인 경우를 볼 수 있다.

![](https://hackmd.io/_uploads/H1tGTsrP3.png)

**만약에 UI 렌더링을 위해 세 번째 코어를 잠시 사용하면 어떻게 될까?**

Load Balancer는 UI 렌더링을 위해 세 번째 블록을 첫 번째 코어로 옮겨야 한다.

![](https://hackmd.io/_uploads/HJjtaiBw2.png)

> we get a bubble of idel CPU(?)

이 시간을 활용하여 더 많은 병렬 작업을 수행할 수 있었습니다. 그래서 작업이 대신 더 오래 걸림.

**어떻게 해결할 수 있을까?**

반복 횟수를 늘리고 로드 밸런서에 더 많은 유연성을 제공하라.?

![](https://hackmd.io/_uploads/BkF-AjrD3.png)

```swift 
DispatchQueue.concurrentPerform(6) { i in /* iteration i */ }

```

세 번째 코어에 또 다른 버블이 있음
![](https://hackmd.io/_uploads/S10LCirDh.png)

이 시간도 활용할 수 있다.

![](https://hackmd.io/_uploads/HyVKCiHPn.png)

```swift 
DispatchQueue.concurrentPerform(11) { i in /* iteration i */ }

```

> Load Balancer가 시스템의 격차를 메우고 시스템의 사용 가능한 리소스를 최대한 활용할 수 있는 유연성을 갖도록 충분히 큰 반복 횟수를 사용할 수 있다.

> Load Balancer의 오버헤드와 벙렬 for 루프의 각 블록이 수행하는 유용한 작업 사이의 균형을 유지해야 한다.


- 모든 CPU를 항상 사용할 수 있는 것은 아님.
    - 시스템에는 동시에 실행되는 많은 작업이 있음
    - 또한, 모든 작업자 스레드가 동일하게 진행되는 것은 아님


**Summary**
> 병렬 문제가 있는 경우 사용 가능한 시스템 프레임워크를 사용해야 한다.
- conccurenPerform을 사용해서 내부의 자동 로드 밸런싱을 활용해야 한다.


### Concurrency

간단한 뉴스 앱을 작성하고 있다고 가정해보자. 
어떻게 구성할까요?
앱을 구성하는 독립적인 하위 시스템으로 나누는 것부터 시작한다.

뉴스 앱을 독립된 하위 시스템으로 나누는 방법을 생각하면 UI를 렌더링하는 UI 컴포넌트들이 있을 수 있다. 이것이 기본 스레드이다(?).

![](https://hackmd.io/_uploads/SJ0Qgnrw2.png)


![](https://hackmd.io/_uploads/rJa-ZhHvh.png)

현재 데이터베이스가 앱의 메인스레드에서 실행되기 때문에 OS는 즉시 CPU를 UI 스레드에서 작업하도록 전환할 수 있으며 데이터베이스 스레드가 완료될 때까지 기다릴 필요 없이 사용자에게 즉시 응답할 수 있다.

- Main Thread의 이점

사용자 인터페이스가 응답을 완료하면 CPU는 다시 데이터베이스 스레드로 전환한 다음 네트워킹 작업도 완료할 수 있다.

-> 따라서 이와 같은 동시성을 활용하면 반응형 앱을 구축할 수 있다.
![](https://hackmd.io/_uploads/rkSQ7TSv3.png)

> Main Thread는 앱의 다른 부분이 완료될 때까지 기다릴 필요 없이 항상 사용자 작업에 응답할 수 있다. 


### Context switching

이제 CPU가 어떻게 생겼는지 살펴보자.

![](https://hackmd.io/_uploads/ryUO7aSv2.png)

- 흰색 선은 하위 시스템 간의 Context Switching을 보여주는 것

> `Context Switching`은 앱을 구성하는 서로 다른 하위 시스템 또는 스레드 간에 CPU가 전환하는 경우를 말한다.

**Instrument System Trace**
> 앱이 어떻게 보이는지 시각화하고 싶다면 CPU와 스레드가 앱에서 실행 중일 때 사용하면 된다.

시스템 추척을 좀 더 자세히 알고 싶으면
**System Trace In-Depth** 강연을 참고하라.

따라서 Context Switching 개념의 동시성의 파워가 나오는 곳이다.

**Context Switching 발생할 수 있는 시기**

![](https://hackmd.io/_uploads/B19pNprPh.png)


스레드가 현재 작업을 완료하거나 리소스를 얻기 위해 대기 중인 경우에도 발생할 수 있다.

![](https://hackmd.io/_uploads/rJTgBprD2.png)

![](https://hackmd.io/_uploads/B1AbHpSv3.png)

**Context Switching 원인**
![](https://hackmd.io/_uploads/rJkVBpHP2.png)

너무 많이 반복하면 비용이 합산되기 시작한다.

1. Contented Resource에 대한 배타적인 접근

- 이런 일이 발생하는 주된 경우
    - 잠금장치가 있을 때, 여러 스레드가 모두 잠금장치를 얻을려고 시도하는 경우

![](https://hackmd.io/_uploads/SJ7hSaHP2.png)

그렇다면 이것이 앱에서 발생하고 있다는 것을 어떻게 알 수 있을까? 

-> System Trace

![](https://hackmd.io/_uploads/S11kL6Hw3.png)

- 시각화한 이미지

이는 매우 짧은 시간 동안 실행되는 많은 스레드가 있고 모두 약간의 cascade에서 서로에게 전달되고 있음을 보여주는 것이다.

**파란색 트랙** - 스레드가 CPU에 있을 떄를 나타낸다.

**빨간색 트랙** - 시스템 호출을 할 때 나타난다.

첫 번째 라인은 대부분의 시간에서 뮤텍스가 사용 가능해지기를 기다리고 있음을 보여주는 것이다(?)

![](https://hackmd.io/_uploads/S1V58TrDn.png)


### Lock Contention
> Fair locks

![](https://hackmd.io/_uploads/SkrkDTBD2.png)

- 잠금 상태와 이를 소유한 스레드를 표시하는 새 잠금 트랙을 추가했다.
- 이 경우 파란색 스레드가 잠금을 소유하고 녹색 스레드가 대기한다.

![](https://hackmd.io/_uploads/S1jVPpHDh.png)

- 파란색 스레드가 잠금 해제되면 해당 잠금의 소유권은 다음 줄에 있는 녹색 스레드로 이전된다.
- 그러나 파란색 스레드가 돌아서서 다시 잠금을 잡으면 잠금이 녹색 스레드용으로 예약되어 있기 때문에 잠금을 할 수 없다.

![](https://hackmd.io/_uploads/r1Z2D6BPh.png)

- 이제 다른 작업을 수행해야 하기 때문에 컨텍스트 전환을 강제합니다.

![](https://hackmd.io/_uploads/ryoaw6Hwh.png)

- 녹색 스레드로 전환이 되면 CPU가 잠금을 완료하고 반복할 수 있다.


잠금을 기다리고 있는 모든 스레드가 리소스를 획득할 기회를 얻으려하지만 다른 방식으로 작동하는 잠금이 있는 경우(?)애는 어떻게 됩니까?

### unfair lock이 무엇을 하는지 살펴보자.

![](https://hackmd.io/_uploads/BkaVuaBvn.png)

- 이번에는 파란색 스레드가 잠금 해제될 때 잠금이 예약되지 않는다

![](https://hackmd.io/_uploads/SJ0UO6BD3.png)

- 잠금장치의 소유권을 가지게 된다.

![](https://hackmd.io/_uploads/SJ4tOpHv3.png)

- Blue는 잠금 장치를 다시 가질 수 있고 Context Switching을 강제하지 않고 즉시 다시 획득하여 CPU에 머무를 수 있다.

-> 녹색 스레드가 잠금 장치를 얻는 기회를 어렵게 만들 수 있지만, 파란색 스레드가 잠금 장치를 다시 획득하기 위해 필요한 Context Switching을 줄일 수 있다.

![](https://hackmd.io/_uploads/r1ebt6rw2.png)

![](https://hackmd.io/_uploads/ryqmY6HD2.png)


## low-level primitive

**Single Owner**

![](https://hackmd.io/_uploads/S1zc2THv2.png)

**No Owner**

![](https://hackmd.io/_uploads/H1O6npBDh.png)

- 런타임에 어떤 스레드가 동기화 primitive를 호출할 지모르기 때문에 Single Owner의 기능이 없다.

**Multi Owner**

![](https://hackmd.io/_uploads/Bk2kT6HPh.png)

- 여러 소유자가 있는 기본 요소는 Sinle onwer가 없기 때문에 오늘날의 시스템은 이를 활용하지 않음.


> Primitive를 선택할 때 서로 다른 우선 순위의 스레드가 상호 작용하는지 여부를 고려하라.

우선순위가 낮은 백그라운드 스레드, 우선순위가 높은 UI 스레드가 있다고 가정하자.

-> 우선 순위가 낮은 백그라운드 스레드에서 대기하여 UI 스레드가 지연되지 않도록 하는 Onwer가 있는 Primitive를 활용할 수 있다.

![](https://hackmd.io/_uploads/H1hRp6Svn.png)


# Using GCD for Concurrency

- GCD 기초는 아래 WWDC를 참고

![](https://hackmd.io/_uploads/H1xmApBDh.png)


## Serial Dispatch Queue

> GCD의 기본적인 동기화 Primitive


- 상호 배제
- FIFO 순서를 제공
- Concurrent atomic enqueue
    - 동시 원자적 인큐 작업은 여러 스레드가 큐에 동시에 작업할 때
- Single dequeuer
    - 시스템이 큐에서 비동기 작업을 실행할 때

![](https://hackmd.io/_uploads/Skg7ckABvh.png)

```swift 
let queue = DispatchQueue(label: "com.exaple.queue")
queue.async { /* 1 */ }
queue.async { /* 2 */ }
queue.sync { /* 3 */ }
```

- DispatchQueue 생성자를 호출하여 직열 대기열을 만들고 있는 상태이며, 사용하지 않은 한 응용프로그램에서 비활성 상태인 메모리를 제공한다. 

![](https://hackmd.io/_uploads/Byc7gAHPh.png)

위의 두 작업들은 비동기이기 때문에 첫 번째 스레드가 결국 `queue.sync`를 호출할 수 잇다. 이것이 Queue와 synchronous가 상호 작용하는 방식이다.

- ordered primitive
- 스레드가 자신의 차례가 될 때까지 기다릴 수 있도록 placeholder를 대기열에 넣는 것이다.

![](https://hackmd.io/_uploads/S1xgVCBvn.png)

## GCD Event Monitoring Primitive 

```swift 
let source = DispatchSource.makeReadSource(fileDescription: fd, queue: queue)
source.setEventHandler { read(fd) }
source.setCancelHandler { close(fd) }
source.activate()
```
This target queue is also where you might put other work that should be serialized with this operation, such as processing the data that was read.

- 이 target 큐는 setEventHandler와 같이 이 작업으로 직렬화해야 하는 다른 작업을 배치할 수도 있는 곳입니다.

- 그런 다음 소스가 무효화 패턴(invalidation pattern)을 구현하는 방법인 setCancelHandler를 설정한다.

![](https://hackmd.io/_uploads/Bk2orCrvh.png)

![](https://hackmd.io/_uploads/H1iaHAHwh.png)

- 타겟 큐(Q1, Q2)에 연결되어 있는 두 개의 소스(S1, S2)
- 그리고 맨 아래에 상호 배제 큐인 EQ, 또 다른 직렬 큐를 추가하여 작은 트리를 형성할 수 있다.


![](https://hackmd.io/_uploads/rJHPU0BP2.png)


```swift 
let Q1 = DispatchQueue(label: "Q1", 
                       target: EQ)
let Q2 = DispatchQueue(label: "Q2", 
                       target: EQ)
```

이 방법은 단순히 Optional Target Argument를 디스패치 큐 생성자에게 전달하는 것이다. 따라서, 이 전체 트리에 대해 공유된 단일 상호 배제 컨텍스트(Shared Single Mutual Exclusion Context)를 제공한다.

![](https://hackmd.io/_uploads/By8gvArvn.png)