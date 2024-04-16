<h1><center> Combine Chapter 01 - Hello Combine </center></h1>

###### tags: `💻 TIL`
###### date: `2024-04-1617:21:33.284Z`

> [color=#724cd1][name=데릭]
> [Hello, Combine!](https://www.kodeco.com/books/combine-asynchronous-programming-with-swift/v4.0/chapters/1-hello-combine)
> [Publisher](https://developer.apple.com/documentation/combine/publisher)
> [Subscriber](https://developer.apple.com/documentation/combine/subscriber)

# 개요 

> Kodeco의 Combine: Asynchronous Programming With Swift를 보면서 학습하자.

## Hello, Combine

Combine 프레임워크를 사용하면 선언형 접근 방법으로 앱에서 필요한 이벤트를 처리할 수 있다. 즉, 여러 개의 Delegate 콜백이나 Completion Handler를 사용하는 대신 이벤트를 하나의 프로세스 체인으로 만들 수 있다. 

체인의 각 Part는 이전 단계에서 받은 Element들에 대해 별도의 작업을 수행하는 Combine Operator이다.

## Asynchronous Programming

Single Thread 언어에서 프로그램은 한 줄씩 순차적으로 실행된다.(ex: javascript)

```swift 
    // begin
    var name = "Derrick"
    print(name)
    name += " Kim"
    print(name)
    // End
```

> Single Thread를 사용하면 데이터의 현재 상태를 항상 확인할 수 있다. 

**다중 스레드**

```swift 
--- Thread 1 ---
begin
  var name = "Tom"
  print(name)

--- Thread 2 ---
name = "Billy Bob"

--- Thread 1 ---
  name += " Harding"
  print(name)
end

```

> 여러 스레드로 name이란 공유자원에 접근할 경우 항상 같은 값을 보장할 수 없다. 

## Foundation and UIKit / AppKit

NSThread와 같은 저수준 API부터 Swift의 현대적 동시성을 활용한 async/await 구조까지 다양한 API를 사용할 수 있습니다.

**Notification Center**: 사용자가 Device의 방향을 변경하거나, 화면에 키보드가 표시되거나 숨겨질 때와 같은 이벤트가 발생할 때마다 Observing된 코드를 실행한다. 

**Delegate Pattern**: 다른 객체를 대신하거나 다른 객체와 상호작용응 통해 행동할 객체를 정의한다. 예를 들어, AppDelegate에서 새로운 Remote notification이 도착했을 때 무엇이 실행되야는지 하는지 정의하지만, 이 코드가 언제 실행될지, 몇 번 실행될지는 알 수 없다.

**GCD(Grand Central Dispatch), Operations**: Task의 실행을 추상화하는 데 도움을 준다. 이 기능을 사용해 코드를 순차적으로 실행하도록 예약하거나, 다양한 우선순위를 가진 다양한 Queue에서 동시에 여러 작업을 실행할 수 있다. 

**Closure**: 코드에서 다른 객체의 실행 여부, 횟수 및 실행할 Context를 결정할 수 있도록 코드를 분리하여 전달할 수 있다. 

> 대부분의 전형적인 코드는 비동기적으로 작업을 수행하지만 실행될 순서에 대해 가정하는 것은 불가능하다. 그럼에도 불구하고 좋은 비동기 프로그램을 작성하는 것은 가능하다. 그저 복잡할 뿐이다. 

![스크린샷 2024-04-13 22.01.44](https://hackmd.io/_uploads/rJ_rf-OeA.png)

Apple은 Timer, NotificationCenter, Core Data와 같은 프레임워크에 Combine을 통합하고 있다. 

### Publisher 

> Publisher는 Subscriber가 하나이상 있을 때, 시간이 지남에 따라 값을 Subscriber에게 내보낼 수 있게 하는 타입이다. 수학 계산, 네트워킹, 사용자 이벤트 처리 등 거의 모든 것이 Publisher가 될 수 있다. 

Publisher의 Input, Failure 관련 타입은 Publisher가 선언한 Output, Failure 타입과 일치해야 한다. 
Pulblisher는 Subscriber를 수락하기 위해 receive(subscriber:)메서드를 구현해야 한다.

그런 다음 Publisher는 Subscriber에서 아래 메서드를 호출할 수 있다. 
- recive(subscription:): 구독 요청을 확인하고 Subsciription 인스턴스를 반환한다. Subscriber는 subscription을 사용하여 Publisher에게 Element를 요구하고 이를 사용하여 Pulisheing을 취소할 수 있다. 

- receive(_:): Publisher에서 Subscriber에게 Element 하나를 전달한다.

- receive(completion:): Publishing이 정상, 오류와 함게 종료되었음을 Subscriber에게 알린다.

모든 Publisher는 Downstream subscriber가 올바르게 작동하려면 이 계약을 준수해야한다. Publisher의 확장 기능은 정교한 이벤트 처리 체인을 만들기 위해 구성하는 다양한 연산자를 정의합니다.

각 Operator는 Publisher는 프로토콜을 구현하는 타입을 반환합니다. 이러한 타입의 대부분은 Publisher 열거형의 확장으로 존재함. 예를 들어, map(_:) Operator는 Publishers.Map의 인스턴스를 반환.

Combine의 AsyncSequence와 유사하지만 구별되는 역할을 수행한다. 

Publisher와 AsyncSequence는 모두 시간이 지남에 따라 Element를 만들지만 차이가 있다.

**Combine**
- pull 모델은 Subsciber가 Publisher로부터 Element를 요청

```swift
import Combine

let subject = PassthroughSubject<Int, Never>()

// 구독자 생성 및 데이터 요청
let subscriber = Subscribers.Sink<Int, Never>(
    receiveCompletion: { completion in
        switch completion {
        case .finished:
            print("Received completion")
        case .failure(let error):
            print("Received error: \(error)")
        }
    },
    receiveValue: { value in
        print("Received value: \(value)")
    }
)

subject.subscribe(subscriber)

// 값 보내기
subject.send(1)
subject.send(2)
subject.send(3)

// 발행자 완료
subject.send(completion: .finished)
```

> 이 코드에서 subject는 요소를 방출할 수 있고, Sink 구독자는 이러한 요소를 받아 처리. subscribe 호출로 구독을 시작하고, send 메소드로 요소를 방출합.



**Async**
- for-await-in 구문을 사용해 AsyncSequence에서 만든 Element를 반복

```swift 
let stream = AsyncStream<Int> { continuation in
    // 요소를 비동기적으로 방출
    continuation.yield(1)
    continuation.yield(2)
    continuation.yield(3)
    
    // 스트림 종료
    continuation.finish()
}

Task {
    // 비동기적으로 요소 반복
    for await number in stream {
        print("Received number: \(number)")
    }
    print("Stream completed")
}
```

**important**

> 두 API 모두 Element를 Mapping, Filtering해서 Sequence를 수정하는 메서드를 제공하지만 Combine만이 **`debounce(for:scheduler:options:)`**,**`throttle(for:scheduler:latest:) `** 와 같은 시간 기반 작업과 **`merge(with:)`** , **`combineLatest(::`** 같은 결합 작업을 제공한다., 

**Combine과 Swift의 동시성 모델 사이의 연결 방법**

- 이 두 기술을 통합하기 위해, 발행자가 내보내는 요소들을 AsyncSequence로 만들 수 있습니다. 이렇게 하면 Subscriber를 연결하는 대신, Swift의 for-await-in 구문을 사용하여 요소들을 반복적으로 처리할 수 있습니다

```swift
import Combine
import Foundation

// Combine Publisher를 AsyncSequence로 변환하는 확장
extension Publisher {
    func asAsyncSequence() -> AsyncThrowingStream<Output, Error> {
        AsyncThrowingStream { continuation in
            let cancellable = self.sink(
                receiveCompletion: { completion in
                    switch completion {
                    case .finished:
                        continuation.finish()
                    case .failure(let error):
                        continuation.finish(throwing: error)
                    }
                },
                receiveValue: { value in
                    continuation.yield(value)
                }
            )
            continuation.onTermination = { @Sendable _ in
                cancellable.cancel()
            }
        }
    }
}

```

> Publisher를 AsyncSequence로 변환하고, for-await-in 구문으로 데이터를 반복 처리

```swift
import Combine

// Combine을 사용한 간단한 Publisher 생성
let numbersPublisher = PassthroughSubject<Int, Never>()
numbersPublisher.send(1)
numbersPublisher.send(2)
numbersPublisher.send(3)
numbersPublisher.send(completion: .finished)

// 비동기 작업
Task {
    for try await number in numbersPublisher.asAsyncSequence() {
        print("Received number: \(number)")
    }
    print("All numbers received and sequence has finished.")
}

// 값을 보내기 위한 시뮬레이션
Task {
    numbersPublisher.send(4)
    numbersPublisher.send(5)
    numbersPublisher.send(6)
    numbersPublisher.send(completion: .finished)
}

```

> PassthroughSubject로 Publisher를 생성하고, AsyncSequence로 변환한 뒤, for-await-in 구문을 사용하여 값을 처리

#### Creating Your Own Publishers

Publisher 프로토콜을 직접 구현하는 대신, Combine 프레임워크에서 제공하는 여러 타입 중 하나를 사용해 Own Publisher를  생성할 수 있다.

- PassthroughSubject와 같은 Subject의 구체적인 하위 클래스를 사용하여 필요할 때 send(_:) 메소드를 호출하여 값을 발행
- CurrentValueSubject를 사용하여 해당 주제의 기본 값이 업데이트될 때마다 값을 발행
- 프로퍼티에 @Published 주석을 추가해서 사용하면, 프로퍼티가 변경될 때마다 이벤트를 방출하는 Publisher가 프로퍼티에 추가된다.

**NOTE**

- Upstream: 데이터 소스 또느 이전 Operator를 지칭한다. 어떤 Publisher에 대해 생각해볼 때, Upstream은 Publisher로 데이터를 보내는 이전 단계나 Element이다. 즉, 특정 Operator 또는 Publisher의 관점에서 데이터가 오는 방향을 말한다. 
    - 여러 데이터 처리 단계를 거치는 경우, 각 단계의 입력 부분은 Upstream이다.

- Downstream: 데이터 수신자 또는 이후 Operator를 의미한다. Downstream은 Publisher에서 데이터를 받아 추가 처리하는 다음 단계나 Element이다.
    - Operator Chain에서 각 Operator는 자신의 입력을 받는 upstream과 출력을 보내는 downstream 사이에서 데이터를 처리한다.

```swift
import Combine

let subject = PassthroughSubject<Int, Never>()

let subscription =
    subject
        .map { $0 * 2 }  // 첫 번째 연산자 (upstream: subject, downstream: filter)
        .filter { $0 > 5 }  // 두 번째 연산자 (upstream: map, downstream: sink)
        .sink(receiveValue: { print("Received \($0)") })

// 값 방출
subject.send(1)
subject.send(3)
subject.send(4)

```

### Subscriber

> Publisher로부터 입력을 받을 수 있는 타입을 선언하는 프로토콜이다.

- Publisher로부터 Element의 스트림을 받는 Subscriber는 관계의 변화를 설명하는 생명주기 이벤트와 함께 받는다. Subscriber의 Input, Failure는 Publisher의 Output, Failure 타입과 일치해야한다. 

Subscriber를 Publisher에 연결하려면 Publisher의 subscribe(:)메서드를 호출한다. 
호출 후, Publisher는 Subscirber의 receive(subscription:)메서드를 호출한다. 이 과정은 Subscriber에게 subscription 인스턴스를 제공하는데 이 인스턴스를 이용해 Publisher에게 Element를 요청하거나 필요할 경우 구독을 취소할 수 있다. 

초기 요청을 받은 후, Publisher는 receive(_:)메서드를 호출해 가능한 비동기적으로 새로운 Elements를 전달한다. 발행이 종료될 때는, 완료 및 오류로 인한 종료를 나타내는 Subscribers.Completion 타입의 매개변수를 사용해 receive(completion:)를 호출.

**Subscriber Operator**

- sink(receiveCompletion:receiveValue:): 완료 신호를 받을 때마다, 새 요소를 받을 때마다 임의의 클로저를 실행

```swift 
let subscription = publisher
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("Completed successfully.")
            case .failure(let error):
                print("Completed with an error: \(error)")
            }
        },
        receiveValue: { value in
            print("Received value: \(value)")
        }
    )
```

- assign(to: on:): 주어진 인스턴스의 키 경로로 식별된 속성에 각 새로 받은 값을 기록, Subsriber는 사용자 정의 코드 없이 결과를 출력해 데이터 모델의 프로퍼티나 UI 컨트롤에 바인딩하여 키 경로를 통해 화면에 데이터를 직접 표시할 수 있게 해준다.

```swift 
let subscription = publisher
    .assign(to: \.someProperty, on: someObject)
```

### Subscription

구동이 끝날 때, Subscriber를 추가하면 Chain 시작 부분에서 Publisher가 활성화 된다. Publisher는 잠재적으로 출력을 받을 Subscriber가 없으면 어떤 값도 내보내지 않는다.

구독은 사용자 지정 코드와 오류 처리를 사용해 비동기 이벤트 체인을 한 번만 선언할 수 있다.

full-Combine으로 전환하는 경우 구독을 통해 앱의 logic을 설명할 수 있으며 완료되면 데이터를 push, pull하거나 객체를 콜백할 필요없이 Combine 시스템이 모든 것을 실행하도록 할 수 있다.

**Cancellable** - Protocol

Cancellable라는 프로토콜로 구독의 메모리를 관리할 필요가 없다. 
시스템에서 제공하는 sink, asign 모두 Cancellable을 따른다. 

> cancel()을 호출하면 할당된 리소스가 해제된다. 또한 타이머, 네트워크 액세스 또는 디스크 I/O와 같은 side-effects도 중지한다.

**AnyCancellable** - Class

취소될 때 제공된 클로저를 실행하는 타입을 지우는 취소 가능한 객체입니다

> AnyCancellable 인스턴스는 초기화가 해제되면 자동으로 cancel()을 호출
