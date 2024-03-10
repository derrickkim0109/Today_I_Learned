<h1><center> Dependency-Injection(3) </center></h1>

###### tags: `💻 TIL`, `DI`
###### date: `2024-01-15T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Dependency-Injection-part01](https://www.phaverageprogrammer.com/post/dependency-injection-part-1) <br>
> [Dependency Injection Wikipedia](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85)

# 개요 

> 어제에 이어서 오늘은 Dependency Injection과 Protocol을 어떻게 활용하는지에 대해 학습해보자.

## Dependency Injection Part 2

이전 글에서는 다양한 타입의 Dependency Injection 공부했다. protocol을 사용하기 전에,  클래스로 테스트가 얼마나 가능한지 확인해보자.

```swift 
class APIClient {}

final class MyAPIClient: APIClient {
    func execute() async -> Result<Void, Error> {
        // API Call
    }
}

final class MyRepository {
    private var service: MyAPIClient
    init(service: MyAPIClient) {
        self.service = service
    }
    
    func someFunction() async -> Result<Void, Error> {
        return await service.execute()
    }
}
```

**MyRepository 클래스에 Success, Failure 이벤트에 대한 Unit Test를 추가하려고 할 때**

```swift 
import XCTest
@testable import Project_Name

final class Project_NameTests: XCTestCase {
    private var sut: MyRepository!

    override func setUpWithError() throws {
        let service = MyAPIClient()
        sut = MyRepository(service: service)
    }

    func testSomeFunctionSuccess() async {
        let result = await sut.someFunction()
        
        switch result {
        case .success:
            /// ...
        case .failure(_):
            XCTFail(("Should not fail"))
        }
    }
	
    func testSomeFunctionFailedWithError() async {
        let result = await sut.someFunction()
        
        switch result {
        case .success:
            XCTFail("Should not succeed")
        case .failure(let error):
            // You may check if it's returning specific error that you expect to receive
        }
    }
}

```

위의 코드는 Service 객체의 Dependency를 Repository 클래스에 주입할 수 있다. 근데 이 코드의 문제는 MyAPIClient 구현체를 직접 호출한다는 것이다. 이 말은, 테스트에서 실제로 사용할 API 객체를 호출한다는 것을 말하는데, 테스트 할 상황에서 이런 방식은 좋지 않다. 

**왜 구현체를 호출하는게 좋지 않을까?**

1. 테스트할 때마다, 이런 호출로 서버에 과부하를 줄 수 있다. 생각해봐라.

> 테스트할 때만 해당되는 것이 아니라 CI/CD가 실행되어 P에서 테스트를 실행할 수 있는 경우에 더 많이 호출될 수 있다. 그리고 여러 개의 PR이 동시에 열린 경우도 고려해야 한다. 

-> 결국 트래픽이 증가할 것이다. 이렇게 되면 서버가 다운되서 테스트에 영향을 미칠 수 있고, API가 Success or Failure를 반환해야 하는데 언제 발생할지 예측할 수 없다. 

그래서 특정 이벤트에 대한 API를 호출하여 추적할 때 Wrong / Fake Data가 추가될 수 있다. Unit Test는 빠르게 완료되어야 하는데, 네트워크 상태가 Test case에 영향을 줄 수도 있다. 

**그래서 Mock Data를 만드는 것이다. 근데 Mock Data를 만들기 전에 해결해야 할 문제들이 있다.**

우리는 Unit Test 객체를 만들어서 테스트할 때, Sut(system under test)인 MyRepository 클래스를 초기화 과정에서 MyAPIClient클래스를 주입받을 것이다. 

**근데 MyAPIClient가 왜 필요하냐?**

-> 이 클래스의 execute()메서드를 호출해야하니까..

그렇다면 구현체의 메서드를 호출하지 않고 다른 방법은 없나..?

이제 `Protocol`이 나온다. 이걸 사용하면 해결할 수 있다. 

![스크린샷 2024-01-15 21.02.52](https://hackmd.io/_uploads/S1DlyjzK6.png)

이전엔 MyRepository클래스가 MyAPIClient 클래스에 종속되어있었다. MyRepository 클래스는 특정한 것이 필요한데, 바로 비동기적으로 수행되는 execute() 메서드를 호출할 수 있어야 합니다.

![스크린샷 2024-01-15 21.05.14](https://hackmd.io/_uploads/H1UFyizFT.png)

위의 다이어그램에서는 MyAPIClient 클래스가 보이지 않는다. MyRepository에는 execute라는 기능을 가진 이 Protocol만 있으면 된다. 

하지만, 그냥 Protocol에 의존하면 끝인가? 아니다. MyAPIClient가 Protocol을 채택하도록 구현해야 한다.

![스크린샷 2024-01-15 21.07.08](https://hackmd.io/_uploads/H1wgxsMFp.png)

- 이게 그 유명한 의존성이 역전된 상황이다.

이전에 MyRepository에는 그냥 MyAPIClient가 제공하는 execute메서드를 호출하기만 했다. 하지만 이제는 MyRepository가 필요한 것들을 프로토콜을 통해서 결정하고 프로토콜을 채택한 구현체에게 요구 사항을 준수하도록 강제로 요청할 수 있게 되었다.

![스크린샷 2024-01-15 21.10.06](https://hackmd.io/_uploads/rkqogjGtp.png)

```swift 
class APIClient {}

protocol MyAPIClientProtocol {
    func execute() async -> Result<Void, Error>
}

final class MyAPIClient: APIClient, MyAPIClientProtocol {
    func execute() async -> Result<Void, Error> {
        return .success(())
    }
}

final class MyRepository {
    private var service: MyAPIClientProtocol
    init(service: MyAPIClientProtocol) {
        self.service = service
    }
    
    func someFunction() async -> Result<Void, Error> {
        return await service.execute()
    }
}

```

MyRepository는 이제 구현체에 의존하지 않고 Protocol에 의존한다. 

이제 Unit Testf를 해보자.

**successful scenario**

```swift
private var sut: MyRepository!

override func setUpWithError() throws {

}

func testSomeFunctionSuccess() async {
    let service = SuccessfulServiceMock()
    sut = MyRepository(service: service)
    let result = await sut.someFunction()

    switch result {
    case .success:
        XCTAssertTrue(true)
    case .failure(_):
        XCTFail("Should not fail")
    }
}
    
private class SuccessfulServiceMock: MyAPIClientProtocol {
    func execute() async -> Result<Void, Error> {
        return .success(())
    }
}
```

**failure scenario**

```swift 
func testSomeFunctionFailedWithError() async {
    let service = FailedServiceMock()
    sut = MyRepository(service: service)
    let result = await sut.someFunction()

    switch result {
    case .success:
        XCTFail("Should not succeed")
    case .failure(let error):
        XCTAssertTrue(true)
        // You may check if it's returning specific error that you expect to receive
        //if let error = error as? MyError {
        //    XCTAssertEqual(MyError.invalid, error)
        //}
    }
}
    
private class FailedServiceMock: MyAPIClientProtocol {
    func execute() async -> Result<Void, Error> {
        return .failure(MyError.invalid)
    }
}

```

 MyRepository와 MyAPIClient 클래스를 분리하고 제어를 역전시켰다.
 
그리고 클래스를 테스트할 수 있게 만들어봤다! 심지어 확장 가능하다..

![스크린샷 2024-01-15 21.14.48](https://hackmd.io/_uploads/S1QT-jGY6.png)

