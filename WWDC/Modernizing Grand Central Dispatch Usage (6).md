<h1><center> Modernizing Grand Central Dispatch Usage(6) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`, `GCD`
###### date: `2024-01-08T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 24:30부터 다시 시작이다. (5)에서는 QOS(Quality of Service)에 대해 학습했다.

## Granularity of Concurrency

네트워킹 서브시스템에서는 일반적으로 하나의 네트워크 연결이 아니라 여러 개의 연결이 있으며 이들은 모두 동일한 설정을 복제한다. 커널에서는 이러한 네트워크 연결을 모니터링해야 할 필요가 있다고 한다. 

GCD(Grand Central Dispatch)를 사용하여 dispatch source 및 dispatch queue와 같은 메커니즘을 통해 이를 수행한다. 

```
In a networking subsystem, you'll have to monitor some network connections in the kernel. And with GCD you'll do that with a dispatch source, and the dispatch queue like you just saw. But of course in any networking subsystem you usually not just have one network connection, you'll have many of them and they will all replicate the same setup.
```

### Event Monitoring Setup

![스크린샷 2024-01-08 20.21.52](https://hackmd.io/_uploads/HJo0qLKu6.png)

오른쪽 세 개의 connection에 초점을 맞추고 어떻게 실행되는지 보자.

**Event Handling on Many Indenpendent Queues**

![스크린샷 2024-01-08 20.22.51](https://hackmd.io/_uploads/SkBGoLtO6.png)

만약 첫 번째 연결이 트리거되면 방금 본 것과 동일한 일이 발생하며 해당 소스에 대한 Event Handler를 Target queue에 enqueue한다. 물론 다른 두 연결이 동시에 작동하면 여전히 복제가 되어 세 개의 큐가 Event Handler가 enqueue된 상태가 될 것이다.

```
If the first connection triggers, just like the same thing we just saw happens, we will enqueue the event handler for that source onto its target queue. Of course if the other two connections fire at the same time, they'll still replicate and you'll end up with three queues with an event handler enqueued.
```

![스크린샷 2024-01-08 20.26.17](https://hackmd.io/_uploads/BkS13Itup.png)

그리고 맨 아래에 세 개의 독립적인 serial queue가 있는데, 시스템에게 세 개의 독립적인 concurrency context를 제공해 달라고 요청한 것라고 한다.

만약 이러한 모든 것이 동시에 활성화된다면, 시스템은 순서대로 이벤트 핸들러를 실행할 세 개의 스레드를 생성해 주게 될 것이다.

![스크린샷 2024-01-08 20.29.47](https://hackmd.io/_uploads/rkDhhUFdT.png)

```
And because you have these three independent serial queues at the bottom, you've really asked the system to provide you with three independent concurrency contexts.

If all these become active at once, the system will oblige and create three threads for you to execute these event handlers.
```

이러한 이벤트 핸들러가 작고 네트워크에서 일부 데이터를 읽어서 Common data structure에 enqueue되는 것은 꽤 흔한 일이다. 

```
it is quite common for these event handlers to be small and only read some data from the network and enqueue it into a common data structure
```

위에서 말한 것처럼 connection이 3개만 있는 것이 아니라 subsystem에 여러 개의 네트워크 연결이 되어 있을 수도 있다. 이런 상황에서 context switch pattern이 나타날 수 있고, 과도한 context switch, 소량의 작업을 실행하는 경우 다른 스레드로 context switching을 다시 수행하는 것을 반복한다. 

 ![스크린샷 2024-01-08 20.32.52](https://hackmd.io/_uploads/SJJ_p8Y_a.png)

**어떻게 개선해야할까?**

![스크린샷 2024-01-08 20.40.00](https://hackmd.io/_uploads/Sy2zJPKOT.png)

위에서 언급한 단일 상호 배제 컨텍스트 아이디어를 적용할 수 있습니다. 단순히 맨 아래에 추가적인 직렬 큐를 배치하고 계층 구조를 형성함으로써, 이 모든 네트워크 연결에 대해 단일 상호 배제 컨텍스트를 얻을 수 있다.

이는 여러 개의 연결이 동시에 활성화되어 작업을 수행할 때 발생하는 상호 배제 문제를 해결하는 방법 중 하나일 수 있다. 상호 배제 컨텍스트는 동시에 하나의 연결만이 작업을 수행하도록 보장하는 메커니즘이다.

```
We can apply the single mutual exclusion context idea that we just talked about by simply putting in an additional serial queue at the bottom and forming a hierarchy, you can get a single mutual exclusion context for all of these network connections.
```

![스크린샷 2024-01-08 20.40.18](https://hackmd.io/_uploads/HJA7JvYO6.png)


그리고 동시에 실행되면 이전과 같은 일들이 발생하고 Event Handler는 Target Queue에 enqueue되지만 하단에 추가적인 직렬 queue가 있기 때문에 단일 스레드를 실행한다. 

![스크린샷 2024-01-08 20.41.38](https://hackmd.io/_uploads/H1A_yPKdp.png)

#### Too Much of a Good Thing

- Repeatedly waiting for exclusive access to contended resources
- Repeatedly switching between independent operations
- Repeatedly bouncing an operation between threads

### Avoid Unbounded Concurrency

> Repeatedly switching between independent operations

**Many queues becoming active at once**
- Independent per-client sources
- Independent per-object queues

**Many workitems submitted to global concurrent queue**
- If workitems block, more threads will be created
- May lead to thread explosion

Global concurrent queue가 작동하는 방식은 기존 스레드가 block될 때 더 많은 스레드를 생성하여 앱에서 지속적으로 좋은 수준의 concurrency를 제공하는 것이다. 

**그러나!** 해당 스레드가 다시 차단되면 **Thread explosion**이라는 상황이 발생할 수 있다.
-> **Building Responses and Efficient Apps with GCD**를 봐라.

28:26