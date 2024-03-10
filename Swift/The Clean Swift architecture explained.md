<h1><center> The Clean Swift architecture explained </center></h1>

###### tags: `💻 TIL`,`Swift`, `Clean Swift`, `Karrot Using`
###### date: `2024-01-0217:21:33.284Z`

> [color=#724cd1][name=데릭]
> [The Clean Swift architecture explained](https://zonneveld.dev/the-clean-swift-architecture-explained/)


## 개요

오늘은 zonneveld라는 분이 쓴 Clean Swift Achitecture에 대해 공부해보자. 당근에서 주로 사용하는 아키텍처 중 하나다. (RIBS, Clean Swift, Reactor Kit)

VIP cycle에 대해 학습하기 전에 먼저 zonneveld가 말한 내용을 정리해보자. 

smooth한 디자인과 훌륭한 사용자 경험을 갖춘 앱을 만드는 것도 중요하지만, 앱 개발자라면 집중적으로 알아야 할 몇 가지 사항들이 있다고 한다. 그 중 하나가 코드의 유지보수를 향상 시키는 아키텍처를 선택하는 것이라고 한다. 좋은 아키텍처를 선택하면 기존 소프트웨어에 새로운 기능들을 더욱 쉽게 추가할 수도 있다. 

Clean Swift는 엉클 밥이 쓴 Clean Architecture를 기반으로 만들어졌다..ㅎ(역시 클린아키텍처..)

## The VIP cycle

Clean Swift 아키텍처는 VIP cycle을 사용하여 앱에서 로직을 분리한다. VIP cycle은 `ViewController`, `Interactor`, `Presenter`로 구성된다. 그리고 모든 클래스는 Logic을 담당한다. 

- ViewController는 Display logic
    - 사용자 인터페이스와 관련된 모든 것을 처리하며, 유저 입력을 Presenter로 
- Interactor는 Business logic
- Presenter는 Presentation logic


![스크린샷 2024-01-02 20.48.37](https://hackmd.io/_uploads/rkgX_uWO6.png)

> **VIP cycle Flow**

먼저 ViewController에 대해 설명한다. ViewController는 Action이 트리거되는 첫 번째 클래스이다. 이 클래스는 뷰를 관리하는 역할을 담당한다. 그래서 모든 outlet, IBAction function들이 나열되어야 한다.

ViewController에서 Action이 시작되자마자, ViewController는 액션을 Interactor에게 전달한다. 이 Interactor는 모든 UseCase들이 구현되어야 하는 클래스를 말한다. 그래서 이 안에서 모든 비지니스 로직을 포함하고 있다는 말이다. 이 객체는 Unit Test를 작성할 때 큰 이점이 있다. 그 이유는 모든 Interactor를 테스트 하는 것이 앱의 모든 비지니스 로직을 테스트 하는 것이기 때문이다.

그리고 Interactor는 요청 사항을 처리하고 Presenter에게 객체를 반환한다. Presenter도 클래스인데 Interactor가 만든 객체를 표현하는 역할을 담당한다. 그래서 Presenter는 viewmodel 객체로 인식되고 이 객체를 viewController에게 반환하여 뷰가 보이게 된다. 

**Clean Swift 아키텍처를 사용하면 어떤 로직이 어떤 클래스에 위치해야 하는지 정확히 알 수 있는 장점이 있다.**
그리고 버그를 해결해야 하거나 기능을 추가할 때, 코드의 어느 부분을 변경해야 하는지 정확히 알 수 있기 때문에 유지보수를 하기 쉽게 만든다.

## Data cycle

앱에서 A ViewController가 B ViewController를 present해야 하는 상황이 있을 수 있다. 이럴 때, Clean Swift는 ViewController 간의 navigating은 Router에 의해 수행된다. (Coordinator와 같은 역할인가..?)

ViewController에 사용할 수 있는 navigation 옵션이 있는 경우에 Router 클래스가 ViewController에 추가된다고 한다. 음.. 그러니까 쉽게 말해 Router에서 NavigationController를 관리한다는 말인 것 같다.  

또한, Router에는 특정 ViewController가 탐색할 수 있는 모든 탐색 옵션이 포함되어 있다고 한다. (Coordinator 맞는 것 같다..) 
이렇게 설정하면, 각 VIP 주기에 Router가 있어 ViewController가 어떤 탐색 옵션들이 있는지 명확하게 알 수 있다.

Presenter는 Presentation Logic을 담당한다. (Presentation Layer 느낌인가..?)

화면 전환이 발생하면 ViewController는 Interactor에게 요청하고 Interactor는 Presenter에게 요청한다 

ViewController -> Interactor -> Presenter 

```

From there, the Presenter will decide that the ViewController may route to the next ViewController using the Router

```
![스크린샷 2024-01-02 21.09.33](https://hackmd.io/_uploads/r1u-6_-dp.png)

아직 감이 안 잡힌다..

## Worker

Interactor에 모든 비지니스 로직이 있을 때, viewmodel이 뚱뚱해지는 것 처럼 엄청 큰 클래스가 될 수 있다. 

이를 방지하기 위해 Interactor가 여러 개의 Worker를 만들 수 있다. Worker는 데이터 수신을 도와주는 Interactor의 helper 역할을 한다. 

Worker는 객체를 생성하고 네트워크 호출을 수행하는 일을 담당한다. (Repository같은 건가?) 그 외에도 오부 SDK를 사용해서 객체를 구현할 때 Worker를 사용할 수 있다. 

예를 들어 네트워크 요청을 수행하는 데 Alamofire를 사용하지만 모든 네트워크 요청을 Worker에서 수행하는 경우, Alamofire SDK만 Worker에서 가져오면 됩니다. -> DataSource에 가까운듯?

![스크린샷 2024-01-02 21.14.53](https://hackmd.io/_uploads/rkvBAOZOp.png)
