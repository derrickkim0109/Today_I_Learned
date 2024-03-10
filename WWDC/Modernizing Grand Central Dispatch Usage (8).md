<h1><center> Modernizing Grand Central Dispatch Usage(8) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`, `GCD`
###### date: `2024-01-10T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 34:05부터 다시 시작이다. (7)에서는 Unified Queue Identity에 대해 학습했다.

## One Identity to Find Them All

> ...and in the kernel bind them

![스크린샷 2024-01-10 21.06.01](https://hackmd.io/_uploads/ry7NdWhda.png)


```swift
let S1 = DispatchSource.makeReadSource(
    fileDecriptor: fd,
    queue, EQ
)

S1.setEventHanlder { ... }
S1.activate()
```

![스크린샷 2024-01-10 21.09.02](https://hackmd.io/_uploads/S1YJFbnd6.png)

makeResource 메서드를 사용해서 Source를 생성해서 Hanlder, Property 등 다양한 이벤트들을 설정한다. 

activate된 순간 Utility는 QOS가 된다. 그리고 Source에 대한 Hanlder가 항상 실행된다는 것을 알게 될 것이다. 

-> Queue Hierarchy에서 상속되기 때문

그리고 새로운 시스템을 통해 Handler가 EQ의 실행 mutual exclusion context에서 실행될 것임을 알게 될 것이다.

![스크린샷 2024-01-10 21.10.53](https://hackmd.io/_uploads/ByDUY-3u6.png)

이제 좀 전에 얘기했던 sync unified indentity로 Source를 등록하겠다.

![스크린샷 2024-01-10 21.11.38](https://hackmd.io/_uploads/HyVFYZ2O6.png)

트리 상에 가지고 있는 높은 우선순위를 가지는 UI QOS Source를 보면, 다루는 방식은 첫 번째 것과 매우 비슷하다는 것을 알 수 있다. 단, 여기서 Event Hanlder를 설정할 때 실제로 원하는 QOS를 지정하고 있다는 점이 다르다.

```
If we look at the higher UI QOS source that we have on the tree, the way we treat it is very similar to the first one, except that when you're setting the event handler here you're specifying the QOS that you actually want.
```

![스크린샷 2024-01-10 21.15.20](https://hackmd.io/_uploads/SymwcWn_6.png)

activate 될 때 어떤 일이 발생하는 지 보자. 이 때는 스냅샷을 찍을 때인데, 이전 계층 구조에서 Utility QOS를 얻었을 떄와 달리 여기서는 Hint(?)를 얻는다. 

## Too Much of a Good Thing

- Repeatedly waiting for exclusive access to contended resources
- Repeatedly switching between independent operations
- Repeatedly bouncing an operation between threads

이전에 설정한 내용은 동일한 execution context에서 두 소스를 모두 실행한다는 점이다. 그리고 커널에서 동일한 unified identity를 사용하여 두 번째 Source를 다시 등록한다. 

따라서, 실제로 우리가 해결하려고 하는 매우 복잡한 identity는 이전에 배포한 버전에서 발생한 문제로, old thread에서 벗어나는 문제를 말한다.

## Without Unified Identity

> In macOS Sierra and iOS 10

![스크린샷 2024-01-10 21.21.02](https://hackmd.io/_uploads/rku3iWhda.png)

## Leveraging Ownership and Unified Identity

![스크린샷 2024-01-10 21.22.09](https://hackmd.io/_uploads/Skol2-hOp.png)

-> 문제가 발생한 스레드 

![스크린샷 2024-01-10 21.23.07](https://hackmd.io/_uploads/HkSVnbnda.png)

-> iOS 11부터 해겲

![스크린샷 2024-01-10 21.24.38](https://hackmd.io/_uploads/Byeqnbhua.png)

스레드가 EQ로 불리게 바뀌었다. Unified Identity의 핵심인데 스레드와 EQ가 동일한 객체로 되었다. 

그리고 커널은 실제로 큐를 실행하고 있다는 것을 알고 있고, CPU 트랙에 반영된다. 

더 이상 이벤트가 표시되지 않고 queue가 실행될 뿐이다..

![스크린샷 2024-01-10 21.26.36](https://hackmd.io/_uploads/Hyvb6Whd6.png)

이 이미지는 스레드에 보류 중인 이벤트가 있다는 것을 표시한 것이다. 그리고 적절한 시점이 되면 첫 번째 핸들러가 완료된 직후 이벤트를 대기열에서 제거한다. 

그리고 커널에서 이벤트를 가져와서 살펴보고 해당 핸들러를 계층 구조에 추가할 수 있다. 

왜 이렇게 복잡하게 진행될까? 

-> 이걸 통해 런타임 동작을 최대한 활용하는 방법을 이해할 수 있다고 한다.

**The runtime uses every possible hint to optimize behavior**

## Modernizing Existing Code

- No dispatch object mutation after activation
- Project your target queue hierarchy


### No Mutation Past Activation

> Set the properties of inactivate objects before activation

- Source handlers
- Target queues

Activation이후에 mutation이 없다는 것은 실제로 dispatch 객체에 어떤 종류의 property가 있을 때 이를 설정할 수 있지만 activate 되자마자 해당 property의 mutating을 중지해야 함을 말하는 것이라고 한다.

```swift 
let mySource = DispatchSource.makeReadSource(
    fileDesriptor: fd,
    queue: myQueue
)

mySource.setEventHandler(qos: .userInteractive { ... }
mySource.setCancelHandler { close(fd) }
mySource.activate()

```

`mySource.activate()`이 후에는 객체의 mutating을 중단하라 그 말이다!

![스크린샷 2024-01-10 21.33.43](https://hackmd.io/_uploads/Syz2CW2ua.png)

-> 문제의 원인이 된다.

activate 시간에 property의 스냅샷을 찍고 나중에 해당 스냅샷을 기반으로 결정을 내릴 것이기 떄문이다. 


### Effects of Queue Graph Mutation

- Priority and onwership snapshots can become stale
    - Defeats priority inversion avoidance
    - Defeats direct handoff opimization
    - Defeats event delivery optimization

- System frameworks may create sources on your behalf
    - XPC connections are like sources

Target Queue Hierarchy를 변경하면 해당 스냅샷이 부실하게 렌더링되어 GCD의 우선순위 역전 방지 알고리즘(priority inversion avoidanace algorithm), Dispatch sync를 위한 직접적으로 전달하는 방법과 같은 매우 중요한 최적화가 무효화된다.  

43:02