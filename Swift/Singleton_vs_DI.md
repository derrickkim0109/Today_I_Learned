<h1><center> Singleton vs Dependency Injection </center></h1>

###### tags: `💻 TIL`,`Swift`
###### date: `2024-01-0117:21:33.284Z`

> [color=#724cd1][name=데릭]
> [Singleton_vs_Dependecy_Injection](https://medium.com/@prithvi2229/singleton-vs-dependency-injection-in-ios-swift-948cfc22a170)

# 개요

Medium 글 중에 Singleton과 Dependency Injection에 대해 쓴 글이 흥미로워 번역해본다.

## Signleton vs Dependecy Injection

> iOS 개발에서 객체 생성과 데이터의 공유를 관리하는 것은 중요하다. iOS에서 객체의 인스턴스를 관리하는 방법 중 싱글톤과 DI(Dependency Injection)에 대해 설명하고자 한다. <br>
> 둘 다 객체에 접근하는 것을 제공하지만 구현 측면, 코드의 유연성, 테스트 가능성 및 유지 관리 가능성에 미치는 영향이 다르다. 

**Singleton**

싱글톤은 클래스의 인스턴스화를 단일 인스턴스로 제한하고 해당 인스턴스에 대해 전역에서 접근이 가능하도록 하는 디자인 패턴이다. 이 패턴을 사용하면 앱의 수명 주기 동안 메모리에 할당되어 있으므로 전역에서 접근할 수 있다. 

**주로 어떨 때 사용하나?**

- 공유 자원, 상태 값을 전역에서 접근할 때
- 객체 인스턴스화를 간단히 관리하고 싶을 때
- 종속성에 대한 변경이 거의 없을 때

**단점은 무엇이 있나?**

- 테스트 가능성: 싱글톤은 위에서 설명한대로 전역에서 상태값을 관리하므로, 구성 요소를 독립적으로 격리하여 테스트하는 것이 어렵다. 
- 높은 결합도: 전역에서 접근하게 되면 각 클래스 별로 결합도가 높아질 수 있어 유연한 변경이 어려워진다.
- 스레드 안정성: Thread safe하게 만들지 않을 경우여러 스레드가 싱글톤 인스턴스에 동시에 접근하면 Race Condition이 발생할 수 있다.

```swift 
class  MySingleton { 
    static  let shared =  MySingleton () 
    
    private  init () { } 
    
    func  doSomething () { 
        print ( "Singleton: Doing Something" ) 
    } 
} 

MySingleton.shared.doSomething()

```

**Dependency Injection(DI)**

Dependency Injection은 객체를 구현부에서 생성하지 않고 외부에서 생성하여 주입해주는 디자인 패턴이다. 이렇게 하면, 구성 요소 간의 느슨한 결합도를 가지게 하며 보다 유연하고 테스트 가능한 코드를 만들 수 있다. 

**Dependency Injection 어떨 때 사용하나?**

- 테스트 용이성을 높이기 위해: 의존성 주입은 테스트 할 때 Mock 객체를 주입함으로써 테스트 케이스를 더 쉽게 작성할 수 있게 도와준다. 
- 코드 결합도를 낮추기 위해: DI를 사용하면 의존성이 모듈 내부에 느슨하게 결합된다. 변경에 조금 더 유연한 코드가 된다.
- 확장성을 항샹시키기 위해
- 의존성의 설정을 중앙화하여 관리
- 런타임에 의존성을 변경하기 위해 

**DI의 단점은?**

- 통합 복잡성(Integration Complexity): 의존성 주입을 도입하는 것은 특히 기존 코드베이스나 프레임워크와 통합할 때 일부 통합 복잡성을 도입할 수 있습니다.

- 의존성 그래프 관리(Dependency Graph Management): 어플리케이션이 성장함에 따라 의존성 그래프를 관리하고 의존성을 올바르게 구성하는 일이 어려워질 수 있습니다.

- 코드의 증가된 중복성(Increased Code Verbosity): 의존성 주입을 구현하려면 종종 의존성 구성을 위한 추가 코드가 필요할 수 있으며, 이는 코드의 중복성을 증가시킬 수 있습니다.

```swift 
protocol MyDependency {
    func performAction()
}

class MyDependencyImplementation: MyDependency {
    func performAction() {
        print("Dependency: Performing an action")
    }
}

class MyClass {
    let dependency: MyDependency

    init(dependency: MyDependency) {
        self.dependency = dependency
    }

    func useDependency() {
        dependency.performAction()
    }
}

// Usage:
let dependency = MyDependencyImplementation()
let myClass = MyClass(dependency: dependency)
myClass.useDependency()

```