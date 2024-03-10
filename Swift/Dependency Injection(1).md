<h1><center> Dependency-Injection(1) </center></h1>

###### tags: `💻 TIL`, `DI`
###### date: `2024-01-12T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Dependency-Injection-part01](https://www.phaverageprogrammer.com/post/dependency-injection-part-1) <br>
> [Dependency Injection Wikipedia](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85)
# 개요 

> 오늘은 [jennifer-eve-vega](https://www.linkedin.com/in/jennifer-eve-vega-b88040181/?originalSubdomain=ph)가 쓴 Dependency Injection에 대해 학습해보고자 한다. 위키백과도 같이 읽어본다. 

## Dependency Injection

> 의존성 주입은 하나의 객체가 다른 객체의 의존성을 제공하는 기술이다. `의존성`은 예를 들어 서비스로 사용할 수 있는 객체를 말한다. 클라이언트가 어떤 서비스를 사용할 것인지 지정하는 대신, 클라이언트에게 무슨 서비스를 사용할 것인지를 말해주는 것이다. `주입`은 의존성(서비스 객체)을 사용하려는 클라이언트 객체로 전달하는 것을 의미한다. 서비스는 클라이언트의 상태 프로퍼티 중 하나라고 할 수 있다. 클라이언트가 서비스를 구축하거나 찾는 것을 허용하는 대신 클라이언트에게 서비스를 전달하는 것이 패턴의 기본 요건이다. 

**의존성 주입의 의도는 객체의 생성과 사용의 관심을 분리하는것이다.** 이는 가독성과 코드의 재사용성을 높혀주는 효과가 있다.

그리고 어떤 서비스를 호출하려는 클라이언트는 그 서비스가 어떻게 구성되어있는지 몰라야 한다. 대신 클라이언트는 서비스 제공에 대한 책임을 외부 코드(injection)로 위임하게 된다. 즉, 클라이언트는 외부 코드를 직접적으로 호출할 수 없다. 주입자에 의해 서비스를 주입받은 클라언트는 실제 서비스의 생성이나 구현체 부분에 대해 알 필요가 없다. 단지 주입된 서비스를 활용하여 원하는 기능을 수행할 뿐이다. 

즉, 클라이언트는 서비스의 사용 방식을 정의하고 있는 서비스의 고유한 인터페이스에 대해서만 알면 된다. 이것은 `구성`의 책임으로부터 `사용`의 책임을 구분한다.

## 의존성 주입이 해결할 수 있는 문제

특정 객체를 필요로 하는 클래스 내에서 직접 객체를 생성하는 것은 클래스를 특정 객체에 의존하게 하는 것이기 때문에 유연하지 못한 코드를 만들 수 있다. 이는 다른 객체를 필요로 하는 경우 재사용을 할 수 없게 만들 수 있다. 또한, 객체를 Mock 객체로 대체할 수 없기 때문에 클래스를 테스트하기 힘들게 한다. 

## 예제

예를 들어, 어떤 작업을 수행하기 위해 Class B에 의존하는 Class A가 있다고 가정해보자. 이 상황에서 클래스 내에서 의존성을 생성하는 변수를 선언하는 대신에 의존성을 주입할 수 있다.

![스크린샷 2024-01-12 23.05.29](https://hackmd.io/_uploads/rkV4DaAuT.png)

```swift 
final class MyAPIClient: APIClient {
    func execute() async throws -> Result<Void, Error> 
}

final class MyRepository {
    private let apiService = MyAPIClient() // 의존하는 객체를 내부에서 생성 

    func didSomething() {
        let result = try await apiService.execute()
        ...
    }
}
```

![스크린샷 2024-01-12 23.08.14](https://hackmd.io/_uploads/BkdAwaRO6.png)

MyRepository가 MyAPIClient에 의존하는 상황이다. 문제는 MyRepository 내부에 MyAPIClient 객체를 생성한 것이다. 이런 구조는 MyRepository의 Unit test를 하기 어렵다. 그 이유는, MyAPIClient를 내부에서 직접 생성하여 호출하기 때문이다. 

하지만 테스트를 진행할 때, Mock 객체로 만들어서 상황을 가정하려면 외부에서 Dependency injection과 Protocol을 사용하면 쉽게 Mock 객체를 만들어서 MyRepository를 테스트할 수 있다. 

위에서 설명한 것처럼, Dependency Injection과 Protocol을 사용하면 문제를 해결할 수 있다. Dependency Injection은 클래스를 분리하기 아주 좋은 방법이고, Scalable and Testable한 객체를 만들 수 있다. 

![스크린샷 2024-01-12 23.17.09](https://hackmd.io/_uploads/Bylx9aAua.png)
