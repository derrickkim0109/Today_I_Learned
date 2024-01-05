<h1><center> Modernizing Grand Central Dispatch Usage(3) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`
###### date: `2024-01-05T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 14:51부터 다시 시작이다. (2)에서는 Concurrency, Contended Resources에 대해 학습했다.

## Lock Contention

> Unfair Locks

이번엔 파란색 스레드가 잠금 장치를 해제할 때, fair 때와는 반대로 녹색이 예약할 수 없도록 가정한다. 

![스크린샷 2024-01-04 21.25.31](https://hackmd.io/_uploads/SyIpXQ4uT.png)

> 파란색 스레드는 잠금 장치의 소유권을 확보한 상태

![스크린샷 2024-01-04 21.26.08](https://hackmd.io/_uploads/SJckEXNda.png)

녹색이 예약할 수 없으니까 파란색 스레드가 다시 CPU를 점유해서 Context Switching을 하지 않는다. 

### Use the right lock for the job

![스크린샷 2024-01-05 21.47.06](https://hackmd.io/_uploads/ByBU9_S_a.png)

## Lock Ownership

Ownership helps resolves priority inversion 

- High priority waiter
- Low priority owner

![스크린샷 2024-01-05 21.48.38](https://hackmd.io/_uploads/BJl2qOrdT.png)

Lock Track을 생각해보면 런타임은 다음에 잠금 장치를 가질 스레드를 알고 있다. 이 기능을 활용하면 앱에서 waiter와 잠금 장치 소유한 스레드 사이의 우선 순위 역전을 자동으로 해결할 수 있다. 


> Priority Inversion (우선 순위 역전): 이는 다중 스레드 환경에서 발생할 수 있는 문제로, 높은 우선 순위의 스레드가 낮은 우선 순위의 스레드가 가지고 있는 리소스(예를 들면 락)를 기다려야 하는 상황이다. 이를 해결하기 위해서는 낮은 우선 순위의 스레드가 리소스를 사용하고 있을 때, 높은 우선 순위의 스레드가 이를 대기하지 않고 계속 진행할 수 있도록 하는 것이 중요하다.

> Directed CPU Handoff (지정된 CPU 양도): 이는 특정 스레드가 작업을 수행하고 있는 동안 해당 작업이 실행되는 CPU를 명시적으로 지정하는 것을 의미한다. 이렇게 하면 스레드 간의 효율적인 리소스 공유가 가능하며, 성능을 향상시킬 수 있다고 한다.

> 위의 문장에서는 이러한 문제들을 자동으로 해결하는 기능이나 최적화를 할 수 있다는 것을 언급하고 있다. 어떤 소프트웨어나 시스템에서 이러한 최적화를 구현하면, 우선 순위 역전 같은 문제를 자동으로 처리하거나 작업을 효율적으로 분배하여 전체 성능을 향상시킬 수 있다.


```
We can take advantage of that power to automatically resolve priority inversions in your app between the waiter and the onwers of the lock.

And even enable other optimizations, like directed CPU handoff to the owning thread.
```

**low-level primitives**

![스크린샷 2024-01-05 21.55.37](https://hackmd.io/_uploads/B1483uS_p.png)

> Single Owner가 가지는 기본 요소들

![스크린샷 2024-01-05 21.56.50](https://hackmd.io/_uploads/SJko2ur_a.png)

> No Owner가 가지는 기본 요소들

- 비대칭 Primitive

Dispatch Semaphore or Dispatch Group과 같은 비대칭 Primitive들은 Single Owner가 가지는 기능들이 없다. 

-> 런타임은 어떤 스레드가 sync Primitive를 신호할지 모르기 때문

> Asymmetric Primitives (비대칭 프리미티브): 이는 동시성(Concurrency)을 다룰 때 사용되는 동기화 기법이나 메커니즘을 나타낸다. "Asymmetric"이라는 용어는 이러한 메커니즘이 특정 스레드 또는 작업에 대한 정보를 갖지 않고 대칭성이 없다는 것을 나타낸다.

> Dispatch Semaphore and Dispatch Group: 이는 Swift와 Objective-C에서 사용되는 Grand Central Dispatch(GCD) 프레임워크에서 제공하는 동기화 기법이다. Semaphore는 리소스에 대한 액세스를 제어하는 데 사용되며, Group은 여러 비동기 작업을 추적하고 완료를 기다릴 수 있도록 도와준다.

> 문장에서는 이러한 비대칭 프리미티브 중에서도 dispatch semaphore와 dispatch group은 특정 문제를 해결하는 데 있어서 제약이 있다고 언급하고 있다. 그 이유는 런타임이 (프로그램의 실행 환경이) 해당 동기화 기법을 신호를 보낼(thread에서 특정 동작을 수행할) 스레드를 미리 알지 못하기 때문이다. Concurrency와 관련된 우선순위 역전 같은 문제를 자동으로 해결하거나 최적화하기 위해서는 어떤 스레드가 신호를 보낼 것인지에 대한 정보가 필요한데, 이러한 비대칭 프리미티브는 이 정보를 런타임에 제공하지 않는다고 한다.

```
However, asymmetric primitives, like dispatch semaphore and dispatch group don't have this power, because the runtime doesn't know what thread will signal the sync primitive.
```

![스크린샷 2024-01-05 22.02.43](https://hackmd.io/_uploads/HygWCuBdp.png)

> Multiple Owner

**요즘은 사용하지 않는다.**

> Primitives with Multiple Owners (여러 소유자를 가진 원시 동기화 메커니즘): 이는 여러 스레드 또는 작업이 공유 자원에 대한 액세스를 동기화하기 위해 사용되는 동기화 메커니즘을 나타낸다. 여러 소유자를 가진다는 것은 여러 스레드나 작업이 이 메커니즘을 소유할 수 있다는 것을 의미한다.

> Private, Concurrent Queues, and Reader or Writer Locks (개인, 동시 큐 및 읽기 또는 쓰기 락): 여러 스레드 간의 자원 공유 및 동기화를 위해 사용되는 다양한 메커니즘들을 나타낸다. Private queues는 개별 스레드 또는 작업에 속하는 큐를 의미하며, Concurrent queues는 여러 작업이 동시에 실행될 수 있는 큐를 나타낸다. Reader-writer locks는 여러 스레드가 동시에 자원을 읽을 수 있고, 쓰기 작업을 수행하는 경우에는 단일 스레드만이 이를 소유하도록 하는 동기화 메커니즘이다.

> 문장에서는 이러한 여러 소유자를 가진 원시 동기화 메커니즘들이 현재 시스템에서 충분히 효과적으로 활용되지 못하는 이유에 대해 언급하고 있다. 그 이유는 현재의 시스템에서는 이러한 메커니즘이 하나의 단일 소유자를 갖지 않기 때문이다. 시스템이 이러한 여러 소유자 메커니즘을 최적화하거나 효과적으로 활용하려면 어떻게 소유권이 공유되고 해제되는지에 대한 명확한 방법이 필요할 것이다.

```
Finally, primitives with multiple owner like private, concurrent queues and reader or writers locks, the system doesn;t take advantage of that today, because there isn't a single owner.

```

primitive를 선택할 때, 서로 다른 우선 순위의 스레드가 상호 작용하는지 여부를 고려해야 한다. 

우선 순위가 낮은 백그라운드 스레드를 실행하면서 UI 스레드가 지연되지 않도록 보장하는 ownership이 있는 primitive를 활용하라.

### Optimizing Lock Contention

- Inefficient behaviors are often emergent properties
- Visualize your app's behavior with Instruments
- Use the right lock for the job


#### Too Much of a Good Thing

- Repeatedly waiting for exclusive access to contended resources
- Repeatedly switching between independent operations
- Repeatedly bouncing an operation between threads

