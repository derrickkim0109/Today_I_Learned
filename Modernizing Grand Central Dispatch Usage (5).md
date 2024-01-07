<h1><center> Modernizing Grand Central Dispatch Usage(5) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`, `GCD`
###### date: `2024-01-06T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 19:37부터 다시 시작이다. (4)에서는 Using GCD for Concurrency에 대해 학습했다.

## Dispatch Source


```swift 

let source = DispatchSource.makeReadSource(fileDescriptor: fd, queue: queue) 

source.setEventHandler { read(fd) }
source.setCancelHandler { close(fd) }
source.activate() 

```

> GCD의 Event monitoring primitive

`makeReadSource` 생성자를 통해 `fileDescriptor`를 모니터링하도록 설정하는 것이다.

- Event handler executes on target queue

- Invalidation pattern with explicit cancellation
- initial setup followed by activate

source의 target 큐에 큐를 전달한다. 여기에서 source의 event handler를 실행한다. 그리고 fileDescriptor로 코드를 읽는다. 

또한, 이 target queue는 읽은 데이터를 처리하고 직렬화하는 다른 작업을 넣을 수도 있다.

그런 다음 source가 cancel hander를 설정하여 invalidation pattern을 구현한다. 

마지막으로 모든 것이 설정되면 source를 호출하고, monitoring을 활성화한다. 

Source 실제로 OS 전체에서 사용자가 지정한 target queue에 이벤트를 전달하는 객체라기 보다 일반적인 패턴의 인스턴스일 뿐이라는 것이다.

- 예: XPC 연결

```
So, it's worth nothing that sources arer really just an instance of a more general pattern throughout the OS, where you have objects that deliver events to you on a target queue that you specify.

```

### Target Queue Hierarchy

- Serial queues and sources can form a tree
- Shared single mutual exclusion context
- Independent individual queue order

![스크린샷 2024-01-07 12.30.24](https://hackmd.io/_uploads/ByjR99P_a.png)

위에는 S1, S2와 Q1, Q2는 두 개의 소스와 큐입니다. 

![스크린샷 2024-01-07 12.33.58](https://hackmd.io/_uploads/r17njcDup.png)

```swift

let Q1 = DispatchQueue(label: "Q1", target: EQ)
let Q2 = DispatchQueue(label: "Q2", target: EQ)

```

이 두 개에 다른 직렬 큐를 추가하고 하단에 exclusion queue인 EQ를 추가함으로써 작은 트리를 형성합니다.

```
So, here we have two sources with their associated target queue, S1, S2 and the queue is Q1 and Q2. And we can form a little tree out of this situation by adding yet another serial queue to the mix, by adding a mutual exclusion queue, EQ, at the bottom.

```

이를 수행하는 방법은 단순히 optional target의 argument를 dispatchQueue 생성자에 전달하는 것이다. 

- 한 번에 하나의 soruce or queue를 실행할 수 잇다.

그러나, queue 1, queue2는 독립적인 개별 queue order가 유지된다.   

무슨 말인지 아래에 설명하겠다.

![스크린샷 2024-01-07 12.38.43](https://hackmd.io/_uploads/rkJR39P_T.png)

이미지에는 queue에 특정 순서로 item을 가지는 queue1과 queue2가 있다. 


![스크린샷 2024-01-07 12.47.55](https://hackmd.io/_uploads/r1Dgyjwd6.png)

우리가 가지고 있는 추가적인 직렬 큐를 통해 단일 작업 스레드가 q1, q2의 작업을 수행한다. 이 큐는 EQ에서 작동하게 되는데 이 과정에서 exclusion의 특성이 적용되어 한 번에 오직 하나의 작업만 수행되게 된다.

- 두 큐의 item들은 큐에서 가지고 있던 개별 순서를 유지하면서 교차 실행될 수 있다.

```
And because we have this extra serial queue at the bottom, and this executes, they will execute in EQ and there will be a single worker thread executing these items giving you that mutual exclusion property, only one item executing at one time.
```

## Quality of Service 

- Abstract notion of priority
- Provides explicit classification of your work
- Affects various execution properties

우선순위에 대한 추상적인 개념..?

QOS와 priority라는 용어를 비슷한 개념으로 사용한다. 

![스크린샷 2024-01-07 13.06.44](https://hackmd.io/_uploads/ByA87jDd6.png)

시스템에는 총 4개의 service class가 있다.
> 위에서부터 아래로 갈수록 낮은 우선순위를 가진다.

### QoS and Target Queue Hierarchy

![스크린샷 2024-01-07 13.08.12](https://hackmd.io/_uploads/SkIhmiDd6.png)

S2는 사용자 인터페이스와 관련이 있다고 가정해보자.
이벤트가 트리거되자마자 UI를 업데이트해야 하는 이벤트가 있는지 모니터링할 수 있다. 

![스크린샷 2024-01-07 13.08.52](https://hackmd.io/_uploads/HkRAXsPda.png)

또 다른 사례는, 실행 흐름을 제공하기 위해 EQ에 레이블을 배치하여 이 트리의 어떤 item도 이 수준 아래에서 실행할 수 없도록 하는 것이다.


![스크린샷 2024-01-07 13.15.27](https://hackmd.io/_uploads/By9wHjvOa.png)

S1이 실행되면서 자체적으로 quality of service(QoS)에 관한 정보를 가지고 있지 않은 경우, 해당 트리(flow)를 사용하게 된다.

```
And now if anything else in this tree fires, for instance source 1, we will be using this flow for the tree if it doesn't have its own quality of service associated.
```

![스크린샷 2024-01-07 13.16.24](https://hackmd.io/_uploads/HyXjHoPu6.png)

- Source의 실행은 커널에서 실행되는 비동기이다.
- source handler의 실행을 위해 EQ에 넣는다.

user space에서 비동기의 quality of service는 일반적으로 queue.async를 호출한 스레드에서 결정된다. 


![스크린샷 2024-01-07 13.19.24](https://hackmd.io/_uploads/r1KUIiv_p.png)

IN의 item을 큐에 넣고 EQ로 실행하는 User initiated thread가 있다.

![스크린샷 2024-01-07 13.19.09](https://hackmd.io/_uploads/Hy_HUivO6.png)


이 상황에서 매우 높은 우선순위를 갖는 UI와 관련된 이벤트를 발생시키는 S2를 실행하는 Event Handler가 있다고 가정하자. 

이 때, 이벤트 핸들러를 실행하고 그 핸들러를 이벤트 큐 (EQ)에 추가하면 어떻게 될까?

![스크린샷 2024-01-07 13.23.30](https://hackmd.io/_uploads/HkhHDiwOp.png)

![스크린샷 2024-01-07 13.24.13](https://hackmd.io/_uploads/HJjuPiDOp.png)

priority inversion 상황이 발생했다. 

![스크린샷 2024-01-07 13.25.29](https://hackmd.io/_uploads/rJLTPjDda.png)

하지만, system이 현재 queue에 추가된 것들 중 가장 높은 우선순위를 가지는 worker thread를 불러와서 이 문제를 해결한다.

```
And now, maybe we have the source 2 that fires with this very high priority UI relevant event that executes its event handler, and enqueues its event handler into EQ.
```

24:30