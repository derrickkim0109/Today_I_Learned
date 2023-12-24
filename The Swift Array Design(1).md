<h1><center> The Swift Array Design(1) </center></h1>

###### tags: `TIL`,`Swift`, `SwiftDocs`, `Array`
###### date: `2023-12-2317:21:33.284Z`

> [color=#724cd1][name=데릭]
> [The Swift Array Design](https://github.com/apple/swift/blob/main/docs/Arrays.md)

# 개요

> Apple Github에 작성되어 있는 Swift Docs를 공부할 것이다. 오늘의 주제는 `The Swift Array Design`에 대한 내용이다.

## 목표

1. Class가 아닌 element 타입의 subscript 할 때, get/set의 기능을 C의 Array와 동등한 성능을 가질 수 있도록 하는 것이 가장 중요한 목표이다. 

2. Cocoa에서 NSArray를 받아서 `Array<AnyObject>`로 표현한다.해당 Array는O(1)의 효율성을 가지고 메모리 할당 없이 NSArray로 Cocoa에 전달 할 수 있어야 한다.

3. Array는 스택으로 사용할 수 있어야 하므로 O(1) append와 popBack이 가능해야 한다.

1번, 2번 목표를 성취하기 위해서는 Static knowledge를 사용한다(Element 타입이 클래스가 아니라는 것이 정적으로 알려진 경우, NSArray의  Wrapping 가능성을 설명하는 코드와 검사가 없어지게 된다.) Swift의 값 타입인 Array는 항상 `ContiguousArray`와 동일하게 가장 효율적인 표현으로 사용한다. 
    
## 구성요소
    
Swift는 세 가지 Generic Array 타입을 제공하는데, 모든 타입들은 평균적으로 O(1)의 시간복잡도의 성장을 가진다. 

- `ContiguousArray<Element>`는 세 타입 중 가장 빠르고 단순하다. 그래서 C Array의 Performance가 필요할 때 사용하라고 한다. ContiguousArray의 Element는 항상 메모리에 연속적으로 저장된다. ![image](https://hackmd.io/_uploads/Sya7JG4D6.png)

- `Array<Element>`는 `ContiguousArray<Element>`와 비슷하지만 Cocoa간의 효율적인 변환에 최적화되어 있다. (Swift에서 제네릭 배열을 사용할 때, Element가 클래스 타입인 경우 해당 배열은 Swift의 ContiguousArray가 아니라 잠재적으로 연속되지 않은 (non-Contiguous) 임의의 NSArray로 지원될 수 있다.) 
- `Array<Element>`는 연관된 Class 타입의 배열 간의 upcast와 downcast를 지원한다. Element가 클래스 타입이 아닌 경우, `Array<Element>`의 성능은 `ContiguousArray<Element>`와 동일하다.

> 만약 배열의 요소인 Element가 클래스 타입이고 이 배열이 Swift의 ContiguousArray를 사용하지 않고 Objective-C의 NSArray와 같은 구조를 사용한다면, 해당 배열은 잠재적으로 연속되지 않은 메모리에 저장될 수 있다. 이는 특히 Objective-C와의 상호 운용성이나 다른 Cocoa 프레임워크와의 통합을 고려할 때 발생할 수 있는 상황이다. 

![image](https://hackmd.io/_uploads/S1w1GGEvT.png)

