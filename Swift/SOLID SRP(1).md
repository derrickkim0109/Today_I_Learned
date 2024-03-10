<h1><center> Solid Principle - SRP </center></h1>

###### tags: `TIL`,`SOLID`
###### date: `2023-12-2817:21:33.284Z`

> [color=#724cd1][name=데릭]
> [SOLID - 위키백과](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84))

# 개요

컴퓨터 프로그래밍에서 SOLID원칙은 아주 중요한 개념이라고 생각한다. 그렇게 생각하는 이유는 객체의 역할, 책임, 의존성에 대해 더 깊게 생각할 수 있기 때문이다. SOLID 원칙은 Clean Architecture를 만든 로버트 마틴이 만든 다섯 가지 원칙을 마이클 페더스가 두문자어 기억술로 소개한 것이라고 한다. 

조금 더 자세히 설명하자면, 프로그래머가 시간이 지나도 유지 보수와 확장이 쉬운 시스템을 만들고자 할 때 이 원칙들을 함께 적용할 수 있다. SOLID 원칙은 소프트웨어 작업에서 프로그래머가 소스 코드가 읽기 쉽고 확장하기 쉽게 될 때까지 소스 코드를 리팩토링하여 코드 냄새(?)를 제거하기 위해 적용할 수 있는 지침이다. 이 원칙들은 애자일 소프트웨워 개발과 적응적 소프트웨어 개발의 전반적 전략의 일부라고 한다. 

## SRP(Single Responsibility Principle) 

처음으로 소개하고자 하는 원칙은 SRP이다. 한 클래스(객체)는 하나의 책임만 가져야 한다는 원칙이다. 클래스를 기준으로 모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 함을 일컫는다. 클래스가 제공하는 모든 기능은 이 책임과 주의 깊게 부합해야 한다. 

로버트 마틴은 책임을 변경하려는 이유로 정의하고, 어떤 클래스나 모듈은 변경하려는 단 하나의 이유만 가져야 한다는 결론을 지었다고 한다. 

예를 들어, 보고서를 편집하고 출력하는 모듈을 생각해보자. 이 모듈은 두 가지 이유로 변경될 수 있다. 첫 번째로 보고서의 내용 때문에 변경될 수 있다. 두 번째로 보고서의 형식 때문에 변경될 수 있다. 이 두 가지 변경의 성격을 생각해보자.

두 가지는 매우 다른 원인에 기인하는데 하나는 실질적인 부분의 성격을 가지고, 다른 하나는 꾸미는 부분의 성격을 가진다는 것이다. 단일 책임 원칙에 의하면 이 두 가지가 분리되는 이유는 두 가지의 책임을 가지기 때문이다. 다른 이유로 변경되어야 하는 두 가지를 묶는 것은 좋지 않은 설계일 수 있다고 한다. 

또한 클래스는 하나의 관심사에 집중하도록 유지하는 것이 중요한 이유는, 클래스를 더욱 튼튼하게 만들기 때문이다. 튼튼?이란 표현을 생각해보면 한 클래스에서 두 가지의 책임을 가지면 코드가 뒤죽박죽 될 위험이 대단히 높아진다는 의미인 것 같다. 

```swift 

class Banker { 
    func runBank() {
        
    }
    
    func monitorSuspiciousPeople() {
        
    }
    
    func serveClient() {
        
    }
}

```

- 은행을 운영하다
- 수상한 사람들을 감시하다
- 고객을 응대하다

Banker라는 클래스는 여러 개의 책임을 가지고 있다. 

**하나의 클래스가 하나의 책임만 가지도록 분리하자.**

```swift 
class Bank {
    func run() {
        
    }
}

class Banker { 
    func serve() {
        
    }
}

class Guard {
    func monitor() {
        
    }
}
```

단순히 각 기능별로 객체를 만들어 봤다. 3개의 swift 파일이 만들어 질 수 있지만, 각자가 책임질 일들이 명확해진 걸 알 수 있다. 

클린 아키텍처에서 생각해보자.

```swift 

final class ABCViewModel {
    private let fetchProductListUseCase: FetchProductListUseCaseInterface 
    
    init(fetchProductListUseCase: FetchProductListUseCaseInterface) {
        self.fetchProductListUseCase  = fetchProductListUseCase
    }
    
    
}

```
> ViewModel

```swift 
protocol FetchProductListUseCaseInterface {
    func fetch() -> AnyPublisher<ProductModel, FetchProductListError>
}

final class FetchProductListUseCase: FetchProductListUseCaseInterface {
    private let repository: ProductRepositoryInterface
    
    init(repository: ProductRepositoryInterface) {
        self.repository = repository
    }
    
    func fetch() -> AnyPublisher<ProductModel, FetchProductListError> {
           return repository.fetch()
            .mapFetchProductListError()
    }
}
```
> UseCase

```swift 

final class ProductRepository: ProductRepositoryInterface {
    private let dataSource: ProductDataSourceInterface

    init(dataSource: ProductDataSourceInterface) {
        self.dataSource = dataSource
    }

    func fetch() -> AnyPublisher<ProductFetchEntity, MoyaError> {
        return dataSource.fetch()
            .map { responseDTO in
                let entity = responseDTO.toEntity()
                return entity
            }
            .eraseToAnyPublisher()
    }
}

```
> Repository

```swift 
protocol ProductDataSourceInterface {
    func fetch() -> AnyPublisher<ProductDTO, MoyaError>
}

final class ProductDataSource: ProductDataSourceInterface {
    private let provider: MoyaProvider<ProductAPIService>

    init(provider: MoyaProvider<ProductAPIService>) {
        self.provider = provider
    }

    func fetch() -> AnyPublisher<ProductDTO, MoyaError> {
        return provider.requestPublisher(.fetch)
            .map(ProductDTO.self)
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
}
```

> DataSource

클린 아키텍처 뿐만 아니라 아키텍처를 설계할 때 항상 생각해야 할 부분이 책임을 분리하여 하나의 관심사에 집중하도록 하는 것이 중요하다고 생각하는데 그 이유는 아래에 설명하겠다. 

왜 ViewModel에서 결국 ProductDataSource의 API를 호출하여 받은 Response DTO인 ProductDTO를 Repository -> UseCase -> ViewModel을 거쳐서 오냐면 나는 SRP에 있다고 생각한다. (의존성 역전에 대한 부분은 다음 시간에 다루겠다.) 

구현체에서 필요한 코드를 담당하는 객체(ProductDataSource)를 따로 구현하고 DataLayer와 DomainLayer의 연결을 위한 RepositoryInterface의 객체의 책임, 비지니스 로직으로서의 책임을 가지는 UseCase로 관심사를 분리함을써 각 역할에 맞게 코드를 분리한 상황이다. 

이렇게 하면 굉장히 많은 파일이 생겨서 유지보수에 오히려 안좋은 거 아닌가..? 하는 생각이 들 수 있다. 나도 많이 겪었던 딜레마 중 하나이다. 하지만, 일을 시작하고 중구난방하게 500개가 넘는 파일들이 있는 것을 보게 된다면.. 왜 SOLID가 중요한지 관심사를 분리하는 것이 중요한지 이해할 수 있을 것이다. 각 객체의 책임을 하나로 둠으로써 더욱 명확하게 역할에 대해 알 수 있게 될 것이다. 

실무에서는 빡빡한 업무 일정으로 feature를 만들어야 해서 여러 개 이상의 책임을 가지는 객체를 만들게 될 가능성이 높다. 그런데도 왜 로버트 마틴은 이런 원칙을 생각하게 되었을까? 

왜 유지보수에 필요하다고 할까? 

여러 가지의 책임을 가지게 되면 500줄 혹은 몇 천줄의 코드가 생길 것이다. 그런 상황에서, 기존의 히스토리를 모르는 새로운 개발자가 이 객체의 코드를 수정한다고 가정해보자. 

얼마만큼의 시간이 더 소요될지 예상이 되는가?

그렇다. 굉장히 코드를 읽기 피곤해지고 간단한 기능도 추가하는게 힘들어 질 수 있다. 

그렇기 때문에 SRP를 항상 고민하면서 객체를 만들어야 한다고 생각한다.