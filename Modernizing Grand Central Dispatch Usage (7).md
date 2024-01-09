<h1><center> Modernizing Grand Central Dispatch Usage(7) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`, `GCD`
###### date: `2024-01-09T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 28:26부터 다시 시작이다. (6)에서는 Granularity of Concurrency에 대해 학습했다.


## One Queue per Subsystem

![스크린샷 2024-01-09 20.20.31](https://hackmd.io/_uploads/S1FW3i5da.png)

뉴스 앱에는 이미 사용자 인터페이스를 위한 메인 큐가 하나 있다. 또한 네트워킹 및 데이터베이스 서브시스템 각각에 대해 하나의 시리얼 큐를 선택할 수 있다.

```
So, here back in our news application, we already have one queue for the user interface, the main queue and we could choose one serial queue for the networking and one serial queue for the database subsystem in addition.
```

그러나, 우리가 배운 것은 subsystem당 하나의 queue 계층구조를 사용하는 것이다.

### One Queue Hierarchy per Subsystem

![스크린샷 2024-01-09 20.24.02](https://hackmd.io/_uploads/rkaRhoqda.png)

이는 서브시스템에 대한 mutual exclusion context를 제공하여 시스템의 나머지 큐 이벤트 하위 구조를 그대로 두고, 큐 계층 구조의 하단에 깔려있는 해당 네트워크 큐 또는 데이터베이스 큐를 집중적으로 다룰 수 있다.

```
This gives you a mutual exclusion context for the subsystem, where you can leave the rest of the queue event substructure of your system alone and just target that network queue or database queue that underlies the bottom of your queue hierarchies.
```

-> 복잡한 앱에 비해 단순한 패턴일 수 있다..?

## Good Granularity of Concurrency

- Fixed number of serial queue hierarchies

중요한 것은 앱에 고정된 수의 serial queue 계층구조를 갖는 것이라고 한다.


복잡한 서브시스템의 경우, 추가적인 큐 계층 구조를 갖는 것이 의미가 있을 수 있다. 예를 들어, 느린 작업이나 큰 작업 item을 위한 보조 큐를 설정하여 기본 큐, 즉 primary 큐가 외부에서 들어오는 요청에 대한 서브시스템 응답성을 유지할 수 있도록 할 수 있다.

```
So, it may make sense to have additional queue hierarchies for a complicated subsystem, say a secondary one for slower work, or larger work items, so that the first one, the primary one, can keep the subsystem responsive to requests coming in from outside.

```

이 맥락에서 생각해야 할 또 다른 중요한 요소는 해당 서브시스템에 제출되는 작업의 세분성(? <- granularity)입니다.

```
Another thing that's important to think about in this context is the granularity of the work submitted to those subsystems.

```

서브시스템 간에 이동할 때 상당히 큰 work item을 사용하길 원한다고 할때, CPU가 서브시스템을 충분한 시간 동안 실행하여 효율적인 성능 상태에 도달할 수 있는 그림을 얻고자 한다고 하면?


```
You want to use fairly large work items when you move between subsystems to get a picture like what we saw earlier in the session, where the CPU is able to execute your subsystem for long enough to reach an efficient performance state.
```

- Coarse workitem granularity between hierachies
- Finer workitem graunlarity inside a hierachy

![스크린샷 2024-01-09 20.36.16](https://hackmd.io/_uploads/SJa2k25dT.png)


## Using GCD for Concurrency

- Organize queues and sources into serial queue hierarchies
- Use a fixed number of serial queue hierarchies
- Size your workitems appropriately

# Introducing Unified Queue Identity

### Mutual Exclusion Context

> Deep dive

![스크린샷 2024-01-09 20.40.10](https://hackmd.io/_uploads/S1Hje29u6.png)


### Unified Queue Identity

> Asynchronous workitems

![스크린샷 2024-01-09 20.42.07](https://hackmd.io/_uploads/B1jfb3c_a.png)

```
let EQ = DispatchQueue(label: "com.example.exclusion-context")

EQ.async { ... }
```


![스크린샷 2024-01-09 20.46.05](https://hackmd.io/_uploads/S1YbG35Op.png)

이번 릴리스에서 변경한 부분은 카운터 객체를 생성하는 것인데, 이 객체는 큐에 강하게 연결되어 있으며 정확히는 커널에서 큐를 대표하는 데 사용된다고 한다.

```
In thise release, we change that and what we do is that we create our counter object, the Unified Queue Identity that is strongly tied to your queue and is exactly meant to represent your queue in the kernel.
```

![스크린샷 2024-01-09 20.46.29](https://hackmd.io/_uploads/Skx7Gn9dT.png)


슬라이드의 점선으로 표시된 스레드 request는 한동안 실행되지 않을 수 있다. 백그라운드 스레드이기 때문일 수 있고 혹은 시스템이 충분히 로드 되어서 스레드를 제공할 가치가 없을 수 있기 때문이다. 

```
The thread request, that dotted line on the slide, may not be fulfilled for some time, because here that's a background thread, and maybe the system is loaded enough that it's not even worth giving you a thread for it.
```

그러고 나서 앱의 다른 경로로 더 많은 작업을 queue에 추가할 수도 있다. 

![스크린샷 2024-01-09 20.54.31](https://hackmd.io/_uploads/Byr-4ncOa.png)
 
placeholder를 enqueue하기 위해 차단되어야 할 때 Unified Queue Identity가 있으므로 이제  스레드의 sync 실행을 Unified Queue Identity에서 차단할 수 있습니다.

비동기 작업에 사용되는 것과 동일한 queue identity를 사용하며 모든 우선 순위 역전을 고려한다.

그러나 큐의 비동기 및 동기 부분을 단일 식별자로 통합했기 때문에 최적화를 적용할 수 있고, 스레드를 민감하게 전환하여 queue delay를 스케줄러에서 등록할 수 있다.

```
Now that we have that Unified Queue Identity, we can actually since that thread has to block to enqueue the placeholder that Daniel told you about a bit earlier, we can block the synchronous execution of that thread on the Unified Queue Identity. The same on that we use for asynchronous work, with all the priority inversion.

But noew that we unified the asynchronous and the synchronous part of the queue in a single identity, we can apply an optimization and delicately switch the thread that's blocking you by passing the scheduler queue and registering the queue delays that Daniel introduced while talking about the scheduler very early.
```

34:05