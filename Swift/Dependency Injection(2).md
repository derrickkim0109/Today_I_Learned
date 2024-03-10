<h1><center> Dependency-Injection(2) </center></h1>

###### tags: `💻 TIL`, `DI`
###### date: `2024-01-14T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Dependency-Injection-part01](https://www.phaverageprogrammer.com/post/dependency-injection-part-1) <br>
> [Dependency Injection Wikipedia](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85)

# 개요 

> 토요일에 이어서 조금 씩이라도 Dependency Injection에 대해 학습해보자.

## Dependency Injection

### Types of Dependency Injections

오늘은 다양한 타입의 Dependency Injection에 대해 알아보자. 처음부터 protocol을 사용하지 않을 것이지만, 왜 Protocol을 사용하는 것이 테스트에 도움이 되는지 알게 될 것이다.

**Constructor Injection - Passing your dependency via contructors**

```swift 
final class MyRepository {
    private var service: MyAPIClient
    
    init(service: MyAPIClient) {
        self.service = service
    }
}
```

예제 코드를 보면 MyAPIClient는 initialization을 통해서 MyRepository에 주입된다. 

![스크린샷 2024-01-14 22.50.15](https://hackmd.io/_uploads/Skbo8PbtT.png)

Builder는 위에서 본 것처럼, Dependency 인스턴스를 생성한 다음, 이를 종속 클래스에 주입하는 것이다.

**Setter Injection - Passing your dependencies via setter methods**

```swift 
final class MyRepository {
    private var service: MyAPIClient

    func setServiceClient(_ service: MyAPIClient) {
        self.service = service
    }
}
```

이 예제 코드는 MyAPIClient를 setter 메서드를 통해 MyRepository에 주입해주는 방식이다.

![스크린샷 2024-01-14 22.52.35](https://hackmd.io/_uploads/BJaXvv-Kp.png)

> 여기서 종속 클래스 MyRepository는 빌더가 클래스에 필요한 Dependency를 전달할 수 있도록 setter 메서드를 제공한다. 하지만, 종속성이 늘어날 경우, setter 메서드를 더 많이 만들어야 한다.

**Property Injection - Passing your dependencies via properties**

![스크린샷 2024-01-14 22.55.02](https://hackmd.io/_uploads/HJgpwPbta.png)

종속 클래스 내에 프로퍼티를 선언하여 필요한 Dependency를 주입한다. init을 사용하지 않아야 하니까 Optional이다. 

> 기본적으로 class의 프로퍼티는 초기값을 가지고 있어야 한다. 아니면, init에서 정의되어야 함.

**Annotation Injection - when using thrid-party libraries**

Resolver같은 외부 라이브러리를 사용할 때, Dependency를 생성한 다음 라이브러리에서 제공하는 프로퍼티 래퍼를 사용해 주입해줄 수도 있다. 

```swift 
final class MyRepository {
	@Injected private var service: MyAPIClient
	
	// If you're using protocols
	// @Injected private var service: MyAPIClientProtocol
}
```

Resolver는 Dependency를 해결하거나 주입한다는 의미라 한다. 그리고 사용하려면 주입할 클래스를 등록해야 한다.

```swift 
import Resolver
extension Resolver: ResolverRegistering {
    public static func registerAllServices() {
         register { MyAPIClient() }.scope(.unique)
         register { MyAPIClient() as MyAPIClientProtocol }
    }
}
```

Graph, Application, Cached, Shared, Unique 5개를 설정할 수 있다고 한다. 

```swift 
register { dependency }.scope(.application) // the same as a singleton
register { dependency }.scope(.graph) // creates new instance
register { dependency as ProtocolName } // uses default scope
```