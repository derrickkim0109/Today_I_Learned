<h1><center> Modernizing Grand Central Dispatch Usage-Concurrency, Context Switching(1) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`
###### date: `2024-01-04T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 7:57부터 다시 시작이다. (1)에서는 Parallelism에 대해 학습했다.

## Concurrency 

> Composition of independently executed tasks

![스크린샷 2024-01-04 20.23.21](https://hackmd.io/_uploads/rk74BGNuT.png)

> CPU 트랙 상단에 표시되는 타임라인

- 단 하나의 코어만 사용할 수 있다고 가정한다.
    - 언제든지 위의 이미지 처럼 여러 스레드 중 하나만 CPU에서 실행할 수 있다.

사용자가 버튼을 클릭하고 뉴스 앱에서 기사 리스트를 refresh하는 상황은 어떻게 될까? 

![스크린샷 2024-01-04 20.26.12](https://hackmd.io/_uploads/rkJyUz4d6.png)

-> 이 인터페이스는 버튼에 대한 응답을 통해 렌더링한 다흠 데이터베이스에 비동기로 요청할 것이다.


![스크린샷 2024-01-04 20.27.46](https://hackmd.io/_uploads/HJRNUGE_6.png)

-> 그 다음 데이터베이스는 기사를 refresh가 필요하다는 것을 결정하게 되고 네트워크 subsystem에 또 다른 명령을 실행한다. 

![스크린샷 2024-01-04 20.29.34](https://hackmd.io/_uploads/Hyqj8GNuT.png)

> 이 시점에서 사용자는 다시 앱을 터치하게 된다.

그리고 데이터베이스가 앱의 메인 스레드에서 실행되기 때문에 OS는 CPU를 UI Thread에서 작업하도록 즉시 전환할 수 있으며, 데이터베이스가 스레드에서 완료될 때까지 기다릴 필요없이 사용자에게 즉시 응답할 수 있다.(비동기라서..?)

![스크린샷 2024-01-04 20.30.46](https://hackmd.io/_uploads/HkblPfEOT.png)

사용자 인터페이스의 응답이 완료되면 CPU는 데이터베이스 스레드로 다시 전환한 다음 네트워킹 작업을 완료할 수 있다. 

> 이처럼 동시성을 활용하면 반응형 앱을 구축할 수 있다. 

메인 스레드는 앱의 다른 부분이 완료될 때까지 기다리지 않고도 항상 사용자 액션에 응답할 수 있다. 

그럼 CPU에 어떤 영향을 미치는지 보자.

### Context Switching

![스크린샷 2024-01-04 20.32.18](https://hackmd.io/_uploads/rJ3rwMEuT.png)

> 이미지의 흰색 선은 subsystem 간의 Context switching을 보여준다.

Context Switching은 CPU가 앱을 구성하는 서로 다른 subsystem이나 스레드 간의 전환을 말한다.

이것들이 앱에서 어떻게 나타나는지 시각화하려면 `Instrument System Trace`를 사용해야 한다. 

![스크린샷 2024-01-04 20.34.54](https://hackmd.io/_uploads/BJ6ydzV_T.png)

> `Instrument System Trace`를 사용하면 앱에서 실행 중일 때 CPU와 스레드가 수행하는 작업을 확인할 수 있다. 더 자세한 내용은 `System Trace In-Depth`를 보자.

```
So, this concept of context switching is where the power of concurrency comes from.
```

이제 Context Switching이 언제 발생할 수 있는지, 그리고 그 원인이 무엇인지 살펴보자.

앞서 본 것처럼 높은 우선순위를 갖는 UI 스레드가 CPU를 필요로 할 때, 데이터베이스 스레드의 실행을 중단하고 UI 스레드에게 CPU를 할당하는 상황에서 Context Switching을 볼 수 있다. 

```
they can start when a high priority thread needs the CPU as we saw earlier, with the UI thread preempting the database thread.
```

또한, 현재 작업을 완료하거나 리소스를 얻을려고 기다리는 경우에도 Context Switching은 발생할 수 있다. 혹은 비동기 요청이 완료되기를 기다리는 상황에서 발생할 수 있다. 

**The OS can choose a new thread at any time**

- A higher priority thread needs the CPU
- A thread finishes its current work
- Waiting to acquire a resource
- Waiting for an asynchronous request to complete

## Excessive Context Switching 

> Too much of a good thing.

However, with this great power of concurrency comes great responsibility as well. 

You can have too much of a good thing.

CPU에서 네트워크 스레드와 데이터베이스 스레드를 전환한다고 가정해보자. 


- Repeatedly bouncing between contexts can become expensive.

다른 작업 간에 전환시키는 게 Concurrency의 Power라 a few Context Switching은 괜찮다고 한다..

![스크린샷 2024-01-04 20.47.35](https://hackmd.io/_uploads/SJ-JoGVd6.png)

> CPU runs less efficiently

하지만..! 수천 번 수행하는 건 문제가 발생한다.

아까 위에서 설명했듯이 흰 막대기가 Context Switching이 발생하고 있는 것인데, 이러면 성능이 저하되기 시작한다. 

그리고 Context Switching으로 Overhead가 추가된다. Context Switching이 발생할 때마다 실행되는 코드를 다시 구축해야 한다. 이것은 컨텍스트 스위치가 발생할 때마다 현재 상태를 저장하고, 다른 컨텍스트로 전환된 후에는 해당 상태를 복원해야 한다는 것을 의미한다. 


```
it's not just the time we spend executing the context switch, it's also the history that the code has built up, it has to regain that history after every context switch.
```

![스크린샷 2024-01-04 20.52.57](https://hackmd.io/_uploads/ry4Q3MN_T.png)

> There may be others ahead in line for CPU access

또 다른 예는, CPU에 접근하기 위해 기다리는 다른 작업이 있을 때이다. 

컨텍스트 스위치를 기다리는 동안 대기해야 하며, 대기 중에 큐(Queue)에 있는 다른 작업들이 먼저 처리되기 때문에 다른 작업이 대기열에서 앞서 진행될 수 있다.

```
You have to wait each time you context switch for the rest of the queue to drain out and so you may be delayed by somebody else ahead of you in line.
```

**그렇다면 과도한 Context Switching을 일으키는 원인은?**

- Repeatedly waiting for exclusive access to contended resources
- Repeatedly switching between independent operations
- Repeatedly bouncing an operation between threads

1. 경쟁적인 자원 접근 (Contended Resources): 여러 스레드 또는 프로세스가 공유 자원에 대한 배타적인 접근을 기다리는 경우, 경쟁이 발생하여 Context Switching이 과하게 발생할 수 있다. 락(Lock)이나 세마포어(Semaphore)와 같은 동기화 메커니즘을 사용할 때 더 과하게 나타날 수 있습니다.

2. 독립된 작업 간의 반복적인 스위칭 (Independent Operations): 서로 독립된 작업을 반복적으로 실행하는 경우에도 Context Switching이 발생할 수 있다. 시스템이 다수의 작업을 동시에 처리하려고 할 때 발생할 수 있다는 거다.

3. 스레드 간 작업 반복적인 이동 (Bouncing Between Threads): 작업이 다른 스레드 간에 반복적으로 이동하는 경우에도 과도한 컨텍스트 스위칭이 발생할 수 있다. 스레드 간의 작업 분배 또는 스케줄링 알고리즘이 효율적으로 수행되지 않을 때 나타날 수 있다.


#### Contended Resources

![스크린샷 2024-01-04 21.01.11](https://hackmd.io/_uploads/SyfMRM4_T.png)

> Lock Contention

여러 스레드에서 잠금 장치를 얻을려고 할 때이다.

**Visualization in Instruments**

![스크린샷 2024-01-04 21.02.19](https://hackmd.io/_uploads/SyUIAM4_6.png)

잠금장치에 여러 스레드가 접근하는 것을 앱에서 어떻게 돌아가는지 확인하고 싶다면 Instrument Trace를 보라.

![스크린샷 2024-01-04 21.03.41](https://hackmd.io/_uploads/r1wjCz4dp.png)

> Thread가 CPU에 있을 때 파란색 트랙으로 표시된다.<br>
> sys를 호출할 때 빨간색 트랙이 표시된다. 위의 이미지는 `mutex_wait sys`를 호출한 것이다.

거의 대부분의 시간이 뮤택스를 사용할 수 있을 때까지 기다리고 있음을 보여주는 것이라고 한다.

![스크린샷 2024-01-04 21.06.02](https://hackmd.io/_uploads/rJS4J74da.png)

on core tiem은 10 마이크로초로 엄청나게 짧다.

![스크린샷 2024-01-04 21.06.31](https://hackmd.io/_uploads/S1GLkQVda.png)

> The cotext switches track at the top

**그래서 문제의 원인이 뭐야?**

![스크린샷 2024-01-04 21.08.07](https://hackmd.io/_uploads/Skf3kXVu6.png)

간단한 타임라인으로 Excess Contention이 어떻게 진행될 수 있는지 살펴보자.

짧은 시간 동안 각 스레드가 실행되고, 그 후에 CPU를 다음 스레드에 양보하며, 이를 반복하는 상황이다.

```
Where each thread is running for a short time, and then giving up the CPU to the next thread, rinse and repeat, for a long time
```

![스크린샷 2024-01-04 21.10.21](https://hackmd.io/_uploads/HkO4x7N_p.png)

> 이런식으로 작업이 되게 하려면?

CPU가 한 번에 한 가지 작업만 집중해서 작업을 완료하고 다음 작업을 수행하는 것이다.

**이 상황에서 무슨 일이 일어날까?**

![스크린샷 2024-01-04 21.12.18](https://hackmd.io/_uploads/rJnjxmNda.png)

> Fair locks

녹색 스레드와 파란색 스레드에 중점을 두고 보자.

이 이미지는 장금 장치의 상태와 이를 소유한 스레드를 보여주는 새로운 트랙을 추가했다.

- 잠금 장치는 파란색 스레드가 소유
- 녹색 스레드 대기 중

![스크린샷 2024-01-04 21.16.37](https://hackmd.io/_uploads/B1ehbXNua.png)

파란색 스레드가 잠금 장치를 해제하면, 장금 장치는 녹색 스레드로 간다. 

![스크린샷 2024-01-04 21.19.04](https://hackmd.io/_uploads/BJ7Bf7VuT.png)

파란색 스레드가 잠금 장치를 다시 획득하려고 할 때이다. 이미 장금장치가 녹색 스레드에 의해 예약되어 있어서 획득할 수 없다

```
However, when the blue thread turns around and grabs the lock again, it can't because the lock is reserved for the green thread.

```

![스크린샷 2024-01-04 21.20.28](https://hackmd.io/_uploads/SJLcMQN_p.png)

이제 다른 작업이 수행되기 때문에 Context Switching이 강제로 된다.

그리고, 녹색 스레드로 Context Switching이 이루어지고, CPU가 잠금장치 작업을 마치면 다시 반복할 수 있다.

```

And we switch to the green thread, and the CPU can then finish the lock and we can repeat.

```

### Unfair Locks

이번엔 파란색 스레드가 잠금 장치를 해제할 때, fair 때와는 반대로 녹색이 예약할 수 없도록 가정한다. 

![스크린샷 2024-01-04 21.25.31](https://hackmd.io/_uploads/SyIpXQ4uT.png)

> 파란색 스레드는 잠금 장치의 소유권을 확보한 상태

![스크린샷 2024-01-04 21.26.08](https://hackmd.io/_uploads/SJckEXNda.png)

14:51