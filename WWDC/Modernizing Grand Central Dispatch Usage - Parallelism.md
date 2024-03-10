<h1><center> Modernizing Grand Central Dispatch Usage(1) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`
###### date: `2024-01-03T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 2023년 6월에 공부하다가 말았던 WWDC를 마무리하고자 한다. 예전에는 다음 시간에 나머지를 공부해야지 하고 생각만해서 잊고 지냈었다. 하지만, 작년 11월부터 새롭게 장착한 마인드로 인해 매주 수요일 WWDC를 학습해야 하는 스케쥴을 만들었다. 이제 나머지 공부는 없다. 그냥 꾸준히 하자.<br>

## Modernizing GCD Usage

GCD는 Single Core인 Apple Watch부터 mini Core인 Mac까지 앱의 코드를 동적으로 확장할 수 있도록 설계되었다. 근데 어떤 종류의 하드웨어를 실행하고 있는지에 대해 너무 걱정할 필요가 없다고 한다.

```
But there are problematic problems that can affect the scalability and the efficiency of your code
```
? 

![스크린샷 2024-01-03 20.24.05](https://hackmd.io/_uploads/ByWJVaGOT.png)

GCD API 혹은 다른 API들을 사용해서 큐를 만들고 시스템에 작업을 하도록 dispatch했을 수 있다. 근데 이런 것들은 Grand Central Dispatch라고 하는데 이는 Concurrency 인터페이스의 일부일 뿐이다.

### Efficiency Through Observation

> Going off core during an operation reduces efficiency

코어들이 나날이 발전했다고 말하는데, 코드가 작업을 완료하기 전에 코어에서 벗어나면 이전에 작업한 내용을 더 이상 활용하지 못하는 상황이 있을 수 있다. 혹은, 코어에 다시 돌아올 때 성능이 손해를 볼 수 있습니다.

```
And you might leave performance on the table when you come back on core.
```

![스크린샷 2024-01-03 20.30.38](https://hackmd.io/_uploads/BJ_wBpMu6.png)

이 그래프는 위의 상황에서 발생하는 문제들을 최적화하는 기술을 통하면 효율성이 크게 향상된다는 말이다. 
중요한 내용은 아래에 있다.

- Parallelism and concurrency
    - 병렬성과 동시성을 가장 잘 표현할 수 있는 방법
- Using GCD for concurrency
    - Grand Central Dispatch를 사용하여 동시성을 표현하기 위해 가장 좋은 방법을 선택하는 방법
- Unified Queue Identity
    - GCD의 주요 개선 사항이라고 한다.
- Finding problem spots
    - instruments를 사용해서 코드에서 있는 문제 지점을 찾는 방법을 보여줄거란다.

How you can chose the best way to express concurrency to grand central dispatch.


## Parallelism and Concurrency

**Parallelism**

> Simultaneous execution of closely related computations

-> 간단하게 여러 코어가 필요하고 동시에 그 코어들을 사용하고 싶을 때다.

**Concurrency**

> Composition of independently executed tasks

-> Single Core에서도 수행할 수 있다.

### Parallelism

앱이 있을 때, 그 안에서 사이즈가 엄청 큰 이미지를 처리한다고 가정해보자. 그리고 이 큰 이미지를 Mac Pro의 코어들을 활용해서 더 빠르게 처리하고 싶다는 생각이 들었다. 

![스크린샷 2024-01-03 20.41.58](https://hackmd.io/_uploads/HyJUOpzda.png)

-> 이미지를 덩어리로 나눠.?

![스크린샷 2024-01-03 20.43.40](https://hackmd.io/_uploads/r1iF_afO6.png)
  
-> 그리고 각 코어가 이 덩어리들을 병렬로 처리하게 한다. 

> This gives you a speed up, because the cores are simultaneously working on different parts of the image

병렬 처리를 통해 이미지를 동시에 여러 부분에서 처리함으로써 성능을 향상시킬 수 있다.

**그럼 이걸 어떻게 코드로 구현해?**

**Take Advantange of System Frameworks**

![스크린샷 2024-01-03 20.45.52](https://hackmd.io/_uploads/BycMFpGOT.png)

병렬로 처리할 수 있는 여러 프레임워크가 있는데 한번 보자. 

**Accelerate**

> advanced Image algorithm의 병렬 execution을 지원하는 기능이 있다.

**Metal && Core**

> GPU를 활용할 수 있다.

근데 GCD는 이걸 쉽게 표현할 수 있는 Tool을 제공한다고 한다. 

GCD에서 병렬성을 표현하는 방법은 **ConcurrentPerform API**를 사용하는 것이다.

- Express explicit parallelism with `DispatchQueue.concurrentPerform`
- Parallel for-loop--calling thread participates in the computation
- More efficient than many asyncs to a concurrent queue

```swift
DispatchQueue.concurrentPerform(1000) { i in 
    /* iteration */
}
```

이 프레임워크는 코어에서 병렬 작업을 수행하길 원하는 것을 알고 있어서 Parallelism을 최적화할 수 있다고 한다. 

concurrentPerform는 시스템의 모든 코어에 자동으로 작업 부하를 분산하는 병렬 for 루프입니다.

> concurrentPerform is a parallel for loop that automatically load balances your computation across all the cores in the system.

```swift
dispatch_apply(DISPATCH_APPLY_AUTO, 1000, ^(size_t i)) {
    /* iteration */
}
```

Objective-C에서도 동일한 기능이 있다고 한다. 

#### Dynamic Resource Availability

> Choosing an iteration count

Three-Core system에서 workload가 실행된다고 가정해보자.

![스크린샷 2024-01-03 21.17.10](https://hackmd.io/_uploads/BkWLl0z_6.png)

```swift 
DispatchQueue.concurrentPerform(3) { i in 
    /* iteration */
}
```

이미지에는 세 개의 block이 세 개의 코어에서 벙렬로 실행되는 것을 볼 수 있다.(ideal case다..)

![스크린샷 2024-01-03 21.19.04](https://hackmd.io/_uploads/HkmTx0zup.png)

**UI Rendering으로 3번째 코어를 잠시 차지하면 어떻게 될까?**

세 번째 코어가 잠시 UI 렌더링에 사용되는 상황을 다루고 있습니다. 그리고 그런 경우 로드 밸런서가 해당 세 번째 블록을 실행하기 위해 그 블록을 첫 번째 코어로 이동시켜야 한다고 한다..

이 맥락에서 "load balancer"는 작업을 여러 코어에 분산시키는 역할을 하는데, 세 번째 코어가 UI 렌더링 등 다른 작업에 사용되면서 해당 코어에서의 작업이 중단된 경우, 로드 밸런서는 이 작업을 다른 사용 가능한 코어로 이동시켜 전체 성능을 유지하려고 합니다. 이는 병렬 처리를 최대한 활용하고 성능을 최적화하기 위한 방법 중 하나입니다.

> What might happens if the third core is taken up for awhile with UI rendering? Well, what happens is the load balancer has to move that third block over to the first core in order to execute it, because it's the third course taken up.

7:57