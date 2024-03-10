<h1><center>Class </center></h1>

###### tags: `💻 면접 스터디`
> [color=#724cd1][name=데릭]

## Class의 성능을 향상 시킬수 있는 방법을 나열해보시오.

- [참고](https://jcsoohwancho.github.io/2019-10-11-Dynamic-Dispatch%EC%99%80-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94/)
- [Increasing Performance by Reducing Dynamic Dispatch](https://developer.apple.com/swift/blog/?id=27)


```swift 
class Parent {
    func someMethod() { 
        //... 
    }
}

class Child: Parent {
    override func someMethod() {
        // ...
    }
}


let object: Parent = Child()

object.someMethod() // Parent의 someMethod를 호출할 것인가, Child의 someMethod를 호출할 것인가?
```

1) Static Dispatch(Direct Call): 변수의 명목상 타입에 맞춰서 메서드와 프로퍼티를 참조합니다. 이 경우 참조될 요소를 컴파일 타임에 결정할 수 있습니다. 위의 예제에서 Parent의 메서드를 호출하는 경우가 이에 해당합니다. 컴파일 타임에 결정이 끝나기 때문에 성능상의 이점이 있으나, 자식 클래스의 요소를 호출하고 싶으면 명시적인 타입 캐스팅으로 변수를 자식 타입으로 만들어줘야 한다. 따라서 프로그램이 다형성을 활용하기 어렵게 만드는 단점이 있다.

2) Dynamic Dispatch(Indirect Call): 변수의 실제 타입에 맞춰 메서드와 프로퍼티를 호출한다. 코드상으로는 이것이 드러나지 않기 때문에 실제 참조될 요소는 런타임에 결정된다. 어떤 서브클래스가 들어와도 실제 타입에 맞는 요소를 참조하기 때문에 다형성 활용에 유리하다. 다만, 런타임에 실제 참조할 요소를 찾는 과정이 있기 때문에(이 과정은 O(1)의 시간복잡도를 가지도록 구현되어 있다.) Static Dispatch보다 성능상에서 손해를 보게 된다는 단점이 있다.

- Swift에서는 일반적으로 Dynamic Dispatch를 채택하였다. 일반적인 경우에는 Dynamic Dispatch가 편리하긴 하지만, 성능을 신경써야 하는 코드에서는 Dynamic Dispatch의 오버헤드도 거슬릴 수 있다. 또한, Dynamic Dispatch의 가능성이 있는 코드에서는 컴파일러의 최적화가 제한되어버린다. 따라서, Swift는 Dynamic Dispatch가 필요하지 않을 경우에 사용할 수 있는 3가지의 성능 최적화 방법을 제시한다. 

1. Final 키워드

- 오버라이드 될 필요가 없는 요소들은 해당 키워드를 붙인다. 
- final 키워드를 붙여서 선언된 클래스, 메서드, 프로퍼티는 오버라이드 할 수 없게 된다. 이렇게 되면 컴파일러는 Dynamic Dispatch를 사용하지 않아도 됨을 알고 이 부분을 최적화 해줄 수 있게 됩니다.


2. Private

- private 키워드를 붙여서 선언하게 되면 해당 요소는 한 파일 내에서만 참조되는 것이 자동으로 보장된다. 따라서 한 파일내에 해당 요소에 대한 오버라이드가 없는 경우 컴파일러가 이를 자동으로 Static Dispatch로 바꿔줄 수 있다.


3. Whole Module Optimization

- internal 선언만으로 final을 추론해낼 수 있다.(물론 모듈 내에 오버라이드가 없는 경우에서만.)
- Swift는 기본적으로 모듈을 이루는 파일들을 개별적으로 컴파일 하기 때문에 다른 파일에서 오버라이딩이 일어났는지 알 수 없습니다. 하지만 WMO를 활성화하면 모듈 전체가 하나의 덩어리로 취급되어 컴파일 되기 때문에 이를 확인하고 더 강력한 추론을 할 수 있게 된다. 이는 모듈 내의 internal 선언은 모듈 바깥에 드러나지 않아 오버라이딩이 불가능 하다는 것이 보장되기 때문이다.

## Convenience init은?

- Class가 정의할 수 있는 보조 이니셜라이저이다. 클래스의 모든 저장 프로퍼티는 초기값을 받기 위해 지정 이니셔라이저와 편의 이니셔라이져를 정의해야 한다.
- 필요하지 않으면 정의하지 않아도 된다.

## Static Dispatch, Dynamic Dispatch에 대해 설명해주세요

- Dispatch란 어떤 메서드를 호출할 것인지를 결정하고 실행하는 매커니즘이다.

### Static Dispatch
- Static Dispatch는 컴파일 타임에 실제 호출할 함수를 결정할 수 있기 때문에 함수 호출 과정이 간단하고, 컴파일러가 이를 최적화할 여지가 많습니다. 즉, 속도가 빠르다는 장점이 있습니다. 하지만, 참조 타입에 따라 호출될 함수가 결정되기 때문에 서브클래싱의 장점을 누리기는 어렵습니다.

>  서브클래싱은 클래스 간에 상속 관계를 통해 기존 클래스를 확장하고, 새로운 기능을 추가하여 코드 재사용성과 유지보수성을 향상 시키는 방법입니다. 하지만, Static Dispatch를 사용하는 경우, 컴파일 타임에 함수 호출에 필요한 정보가 결정되기 때문에, 함수가 호출될 때 객체의 실제 타입을 고려하지 않습니다. 따라서, 서브클래싱 관계를 활용하여 객체의 동적인 특성을 활용하는 것이 어려워집니다. 

> 예를 들어, 만약 여러 클래스에서 상속받은 메서드를 호출해야 하는 경우, Static Dispatch를 사용하면 그 메서드가 호출될 때 객체의 실제 타입이 아닌 참조 타입을 기반으로 호출됩니다. 상속받은 메서드가 서브클레스에서 오버라이딩이 된 경우, 원하는 결과를 얻지 못할 수 있습니다. 
>
> 따라서, Static Dispatch를 사용하면 성능이 향상되는 대신, 서브클래스의 장점을 제대로 활용하기 어렵다는 것이죠.

- Dynamic Dispatch는 런타임에 호출될 함수를 결정합니다. 이를 위해서 Swift에서는 클래스마다 vtable(Virtual Dispatch Table)이라는 것을 유지합니다. 이는 함수 포인터들의 배열로 표현되며, 하위 클래스가 메소드를 호출할 때 이 배열을 참조하여 실제 호출할 함수를 결정합니다. 이 모든 과정이 런타임에 결정되기 때문에 Static Dispatch에 비하면 추가적인 연산이 필요할 수 밖에 없고, 컴파일러가 최적화 할 여지도 많지 않습니다. 

### Dynamic Dispatch

-  Dispatch란 어떤 메서드를 호출할 것인지를 결정하여, 그것을 실행하는 메커니즘이다

- 내가 호출할 함수를 "컴파일 타임"에 결정하냐, "런타임"에 결정하냐에 따른 방식임 

### Value Type에서의 Dispatch

- Value Type에 해당하는 struct와 enum은 상속을 사용할 수 없다는 특징을 가지고 있습니다. 즉, static Dispatch의 단점인 서브클래싱이 불가능하다는 단점을 완벽히 피해갑니다. 따라서, Value Type에는 Static Dispatch가 적용됩니다. 

```swift 
struct someStruct: someProtocol {
    func action() -> Int {
        return 1 
    }   
}

let s = someStruct()

print(s.action()) // 1

```

### Reference Type에서의 Dispatch

- Reference Type에 해당하는 class는 반대로 항상 상속의 가능성에 노출되어 잇습니다. 따라서 Dynamic Dispatch를 사용합니다. 그 대신 오버라이딩이 되지 않는다는 것을 컴파일러가 알 수 있다면, 컴파일러가 이를 Static Dispatch로 바꿔줄 수 있습니다.

```swift
 class SomeClass{
  func action() -> Int { 2 }
  }

  class DerivedClass: SomeClass {
      override func action() -> Int { 3 }
  }

  var c:SomeClass = SomeClass()
  print(c.action()) // 2

  c = DerivedClass()
  print(c.action()) // 3
```

### Protocol에선 method Dispatch

- 프로토콜은 기본적으로 구현체를 제공하지 않아도 선언부만 선언한다. 따라서 프로토콜을 채택한 타입은 이를 구현해야 하며, 프로토콜을 통해 호출하는 메서드는 프로토콜을 채택한 타입들이 실제로 구현한 메서드들이다. 그런데 프로토콜 타입의 참조만으로 이들을 사용해야 한다면, 해당 인스턴스의 타입에 맞는 메서드를 호출해야 할 것이다. 이를 위해, 프로토콜은 고유의 vTable을 가지게 되며, 특별히 이를 Witness Table이라고 한다. 즉, 프로토콜 역시 Dynamic Dispatch를 사용합니다.


```swift 
  protocol SomeProtocol {
  func action() -> Int
  }

  class SomeClass: SomeProtocol {
  func action() -> Int { 2 }
  }

  class DerivedClass: SomeClass {
      override func action() -> Int { 3 }
  }

  var c:SomeProtocol = SomeClass()
  print(c.action())

  c = DerivedClass()
  print(c.action())

```

### Extension에서 Dispatch


1. 값 타입: 상속 가능성이 없기 때문에 Static Dispatch로 수행된다.
```swift 
  struct SomeStruct: SomeProtocol {
      func action() -> Int { 1 }
  }

  extension SomeStruct {
      func anotherAction() -> Int { 4 }
  }  

  let s = SomeStruct()

  print(s.action()) // 1
  print(s.anotherAction()) // 4      

```

2. 클래스 타입: extension으로 클래스에 추가 기능을 구현할 경우에 Override는 다음과 같은 규칙을 따른다. 
    - 하위 메서드를 오버라이드 하는 경우: 기본적으로 extension에서는 메서드 오버라이드를 금지한다. 이는 클래스 본체에 선언된 메서드를 오버라이드 하는 것과 extension에서 선언된 메서드를 오버라이드 하는 것을 모두 포함한다. 이러한 기능을 사용하기 위해서는, Objective-C 런타임의 힘을 빌려야 한다. 
    - 오버라이드 하거나, 오버라이드 되지 않는 경우: 자기 자신 뿐 아니라 하위 타입에서도 동일한 메서드를 참조함을 보장할 수 있습니다. 따라서 클래스 타입이여도 Static Dispatch가 가능해집니다. 

```swift 
  class SomeClass {
  func action() -> Int { 2 }
  }

  class DerivedClass: SomeClass {
      override func action() -> Int { 3 }
  }

  extension SomeClass {
      func anotherAction() -> Int { 5 }
  }  

  var c:SomeClass = SomeClass()
  print(c.anotherAction())// 5

  c = DerivedClass()
  print(c.anotherAction())// 5

```

3. 프로토콜 타입: 프로토콜 역시 extension을 통해 기능을 추가할 수 있습니다. 이 때, 프로토콜 본체에 선언된 멤버의 디폴트 구현체를 제공해주거나 프로토콜 본체에는 없는 기능을 추가적으로 제공해 줄 수 도 있습니다.
  - 본체에 선언된 멤버의 디폴트 구현체를 제공하는 경우: 하위 클래스들이 메서드를 구현하고 잇음이 반드시 보장됩니다. 설령 구현하지 않았다 하더라도 디폴트 메서드를 이용하면 됩니다. 따라서 Witness Table을 이용한 Dynamic Dispatch가 이루어집니다. 

```swift 
protocol SomeProtocol {
    func action() -> Int
}

extension SomeProtocol {
    func action() -> Int {
        return 4
    }
}

class SomeClass: SomeProtocol {
    func action() -> Int {
        return 2
    }
}

class DerivedClass: SomeClass {
    override func action() -> Int {
        return 3
    }
}

var c: SomeProtocol = SomeClass()
print(c.action()) // 2

c = DerivedClass()
print(c.action()) // 3

``` 
  - 본체에 없는 기능을 추가한 경우: 본체에 선언하지 않고 extension으로 추가한 메서드들은 Witness Table을 이용할 수 없습니다. 즉, Dynamic Dispatch를 사용할 수 없고, Static Dispatch가 적용됩니다. 

```swift 
protocol SomeProtocol {}

extension SomeProtocol {
    func action() -> Int {
        return 4
    }
}

class SomeClass: SomeProtocol {
    func action() -> Int {
        return 2
    }
}

class DerivedClass: SomeClass {
    override func action() -> Int {
        return 3
    }
}

var c: SomeProtocol = SomeClass()
print(c.action()) // 4

c = DerivedClass()
print(c.action()) // 4
```
