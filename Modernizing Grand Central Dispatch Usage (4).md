<h1><center> Modernizing Grand Central Dispatch Usage(4) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`, `GCD`
###### date: `2024-01-06T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 17:32부터 다시 시작이다. (3)에서는 Lock Contention에 대해 학습했다.

## Using GCD for Concurrency

### Serial Dispatch Queue

> Fundamental GCD primitive

GCD의 기본 동기화 primitive이다.

- Mutual exclusion
- FIFO ordered
- Concurrent atomic enqueue
- Single dequeuer

`Concurrent atomic enqueue`이 있으므로 여러 스레드가 동시에 queue에 추가되고 작업이 queue에 들어가는 것이 가능하다. 

그리고 시스템이 queue에서 비동기 작업을 실행하기 위해 제공하는 Single dequeue 스레드도 있다.


아래는 디스패치 큐 생성자를 호출해 직렬 큐를 생성하고 있으며, 아직 사용하지 않은 한 앱에서 비활성화 된 메모리 조각을 제공한다.

![스크린샷 2024-01-06 23.14.19](https://hackmd.io/_uploads/HJ8HgkPup.png)

```swift 
let queue = DispatchQueue(label: "com.example.queue")
queue.async { 
    /* 1 */ 
}
queue.async { 
    /* 2 */ 
}
queue.sync { 
    /* 3 */ 
}
```

이제, 몇 개의 비동기 작업을 위의 queue에 넣기 위해 queue.async 메서드를 호출하는 두 개의 스레드가 있다고 상상해보자.

![스크린샷 2024-01-06 23.15.22](https://hackmd.io/_uploads/H1dFeyPOa.png)

위에는 비동기 작업이기 때문에 해당 메서드들은 반환되고 스레드가 완료 될 때까지 진행될 수 있으므로 첫 번째 쓰레드는 queue.sync를 호출할 수 있다. 이게 queue와 동기적으로 상호 작용하는 방식이다.

![스크린샷 2024-01-06 23.15.55](https://hackmd.io/_uploads/B1SoxywdT.png)

![스크린샷 2024-01-06 23.18.13](https://hackmd.io/_uploads/HJxNW1P_6.png)

순서가 지정된 primitive라 스레드의 차례가 될 때까지 기다릴 수 있도록 place holder를 queue에 추가하는 것이라고 한다. 

![스크린샷 2024-01-06 23.21.01](https://hackmd.io/_uploads/rkO0ZkwOp.png)

![스크린샷 2024-01-06 23.21.11](https://hackmd.io/_uploads/rJfJMJD_p.png)


이제 비동기 작업 항목을 실행할 비동기 작업자 스레드가 나타날 것이다. 그러다가 해당 플레이스홀더에 도달하면 queue.sync에서 대기 중인 스레드에게 큐의 소유권이 전달되어 그 블록을 실행할 수 있게 된다.


```
And now, there's an asynchronous worker thread that will come along to execute the asynchronous work items, until you get to that placeholder at which point the ownership of the queue will transfer to the thread waiting in queue.sync so that it can execute its block.
```

19:37