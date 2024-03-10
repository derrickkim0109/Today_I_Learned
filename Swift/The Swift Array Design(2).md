<h1><center> The Swift Array Design(2) </center></h1>

###### tags: `TIL`,`Swift`, `SwiftDocs`, `Array`
###### date: `2023-12-2417:21:33.284Z`

> [color=#724cd1][name=데릭]
> [The Swift Array Design](https://github.com/apple/swift/blob/main/docs/Arrays.md)

# 개요

>  [The Swift Array Design(1)](https://hackmd.io/qazgczJtRJioMd3SBKnUKw)에 이어 나머지 내용을 변역하고 공부해보자. 

## Mutation Semantics

ArrayType은 COW(Copy-On-Write)를 통해 값 타입 체계를 가진다. 

```swift 
var a = [1, 2, 3]
let b = a
a[1] = 42
print(b[1]) // prints "2"
```

## Bridging Rules and Terminology for all Types

- 모든 클래스 타입 혹은 `@objc`는 Objective-C에 Bridge되고 ID 변환을 통해 Swift로 Bridge된다.

- 그대로 브리지되지 않는 타입 T는 Objective-C와의 변환을 지정하는 `BridgedToObjectiveC`를 사용하면 된다.

```swift 
protocol _BridgedToObjectiveC {
    typealias _ObjectiveCType: AnyObject
    func _bridgeToObjectiveC() -> _ObjectiveCType
    class func _forceBridgeFromObjectiveC(_: _ObjectiveCType) -> Self
}
```

**NOTE**

> Class와 @objc는 현재 컴파일 타입에 적용되지 않는 제한사항으로 인해 `BridgedToObjectiveC`를 따르지 않는다. 

- 일부 Generic 타입(ex_`Array<T>`)는 Element의 타입이 연결되는 경우에만 Objective-C에 연결됩니다. 이러한 타입은 `_ConditionallyBridgedToObjective-C`를 준수합니다.

```swift 
protocol _ConditionallyBridgedToObjectiveC : _BridgedToObjectiveC {
    class func _isBridgedToObjectiveC() -> Bool
    class func _conditionallyBridgeFromObjectiveC(_: _ObjectiveCType) -> Self?
}
```
    
`T._isBridgedToObjectiveC()`의 상태가 false일 때, `_ConditionallyBridgedToObjective-C`를 준수한느 타입 T에 연결하는 것은 런타입 시 오류가 됩니다.

`_conditionallyBridgeFromObjectiveC`를 사용하여 다시 연결하는 것을 시도하고 전체 Object가 연결될 수 없는 경우 nil을 반환할 수 있습니다.

**Implementation Note**

컴파일 타임에 브리지를 감지하는 것을 시도하기 위한 다양한 방법들이 있습니다. 

- T타입이 그대로 연결되지 않는 경우
    - T가 `BridgedToObjectiveC`를 준수하고 아래 중 하나인 경우
        - T가 `_ConditionallyBridgedToObjectiveC`를 준수하지 않은 경우
        - T가 `._isBridgedToObjectiveC`를 준수하지 않은 경우

그런 다음 T타입의 값 x는 x._bridgeToObjectiveC()를 통해 T._ObjectiveCType으로 연결되고, T._ObjectiveCType의 개체 y는 T._forceBridgeFromObjectiveC(y)를 통해 T로 다시 연결됩니다.

## Array Type Conversions

이 문서에서는 Array 변환의 하위 집합을 지원하는 Slice, ContiguousArray가 아니라 Array 자체에 대해서만 다룹니다. 향후, 개정판에서 Slice, ContiguousArray 변환에 대한 설명이 추가될 예정입니다.

### Kind of Convertions

이 정의에서 `Base`는 AnyObject 또는 그 하위타입을 말하고 `Derived`는 `Base`의 하위 타입이며 `X`는 `_BridgedToObjectiveC`를 준수한다. 

- 간단한 Bridging은 암시적으로 `[Base]`를 시간복잡도 O(1)의 NSArray로 변환한다. 이는 단순히 NSArray 배열의 내부 버퍼를 반환하는 문제이다. 
- 간단한 Bridging Back은 NSArray를 시간복잡도 O(1)의 [AnyObject]로 암시적으로 변환하고 NSArray에서 copy()메서드를 호출하는 비용을 추가한다. 
- Array 타입 간의 암시적 변환
    - 암시적인 upcasting은 O(1)시간복잡도에서 암시적으로 [Derived]를 [Base]로 변환한다.
    - 암시적인 Bridging은 암시적으로 [X]를 O(N)시간복잡도를 가지는 [X._ObjectiveCType]로 변환한다.

**NOTE**

> 암시적 변환 타입 중 하나는 NSArray에 대한 암시적 변환에서 `[trivial bridging](#trivial bridging)`와 결합될 수 있다. 

- 검사된 변환은 [T]를 [U]로 변환하는가? O(N)의 시간복잡도를 가지는 [U]로 변환한다.
    - 검사된 다운캐스팅은 [Base]를 [Derived]로 변환하는가?
    -  여기서 X._ObjectiveCType은 T이거나  [Derived]가 간단한 하위 유형(subtype)인 경우 검사된 [T]를 [X]로 다시 브리징 하는가?

- 강제 변환(Forced Convertions)은 암시적으로 [AnyObject] 또는 NSArray를 [T]로 변환하며, Swift와 Objective-C 사이의 브리징 펑크에서 발생한다.

예를 들어, 사용자가 [NSView]를 인자로 가지는 Swift메서드를 작성할 때, 이는 Objective-C에서 NSArray를 인자로 받는 메서드로 노출되며, Objective-C에서 호출될 때 [NSView]로 강제 변환된다. 

강제 다운캐스팅은 [AnyObject]를 O(1) 시간 안에 [Derived]로 변환한다. 

Forced Bridging Back은 [AnyObject]를 [X]로 O(N) 시간 안에 변환한다.


강제 다운캐스팅은 특정 Element에 대한 변환 오류가 발생하더라도 예외를 즉시 발생시키지 않고, 해당 Element에 접근되는 시점까지 트랩을 연기할 수 있다.

**NOTE**

> 검사된 다운캐스트와 강제 다운캐스트 모두 NSArray로 변환 시 `[trivial bridging back](#trivial bridging back)`와 결합될 수 있다.

### Maintaining Type-Safety

업캐스트와 강제 다운캐스트 둘다 type-safety 문제를 발생시킨다. 

**Upcasts**

[Derived]에서 [Base]로 업개스팅하는 경우, Derived 객체의 버퍼를 unsfaeBitCast를 사용하여 Base 타입의 Element 버퍼오 단순히 변환할 수 있다.(다만, Resulting 버퍼는 절대 변하지 않는다) 예를 들어, 우리는 버퍼에 Base Element를 넣을 수 없다. 왜냐하면, 버퍼의 소멸자는 Derived의 타입으로 가정하고 (잘못된) static 전제로 Element를 없앨 것이기 때문이다.

뿐만아니라, 변환되기 이전에 buffer를 복사할 수 없다. 왜냐하면, [Base]가 변환되기 전에 복사될 수 있고, 공유된 subscript Semantics는 모든 복사본이 subscript 할당을 반드시 준수해야 함을 의미하기 때문이다.

따라서 [T]를 [U]로 변환하는 것은 resizing과 유사하다.(새로운 배열은 독립적으로 된다.) O(N)의 시간 비용이 발생하는 것을 피하고 공유된 subscript semantics를 유지하기 위해 데이터 구조에 간접 계층을 사용한다. 더 나아가, T가 U의 하위 클래스인 경우 중간 객체는 버퍼의 내부 변이를 방지하기 위해 표시되며, 처음 변이시에 복사된다. 
(Copy On Write인가..?)
![image](https://hackmd.io/_uploads/S1k8A7SDp.png)


**Deferred Checking for Forced Downcasts**

강제 다운캐스팅에서 만약 어떤 Element가 동적 타입 Derived를 가질 수 없다면, 이는 트랩을 일으킬 수 있는 프로그래밍 오류로 간주된다. 때로는 소스가 알려진 버퍼 타입을 가지고 있기 때문에 이 검사를 위해 O(1)의 시간 안에 수행할 수 있다. 다른 경우에 대한 검사가 O(N)의 시간을 발생시키는 대신, 새로운 중간 객체는 연기된 검사를 위해 표시되며, 해당 객체를 통해 모든 Element에 대한 접근은 동적으로 타입이 검사되어 실패할 경우 트랩이 발생한다. (단, -Ounchecked 빌드에서는 예외.)

