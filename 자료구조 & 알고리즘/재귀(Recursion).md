<h1><center> 재귀(Recursion) </center></h1>

###### tags: `💻 TIL`
###### date: `2024-02-20T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Nossi.DEV - Recursion](https://www.nossi.dev/edd45041-6c62-4d69-badc-b14af14def31)

# 개요 

> 재귀는 추후 그래프 탐색, 트리, dp 등의 자료구조와 알고리즘에 적목되기에 아주 중요하다. 

## 재귀 

> 자신을 정의할 때, 자기 자신을 재참조하는 것을 말한다.

대표적인 재귀함수로는 factorial, fibonacci가 있다. 

### Example

**factorial** 

```swift 
func factorial(_ number: Int) -> Int {
    if number == 1 {
        return 1 
    } 
    
    return number * factorial(number - 1)
}
```

`factorial`의 반환결과에 `factorial`를 재참조하는 것을 확인할 수 있다. 

**fibonacci**

```swift 
func fibonacci(_ number: Int) -> Int {
    if number == 1 || number == 2 {
        return 1
    } 
    
    return fibonacci(number - 1) + fibonacci(number - 2)
}
```

## 재귀의 수학적 접근

fibonacci와 factorial을 자세히 살펴보면, n번째 함수를 n-1번째 함수로 표현하는 것을 알 수 있다.

**점화식(Recurrence Relation)**

- factorial: $f(n) = n * f(n-1)$
- fibonacci: $f(n) = f(n-1) + f(n-2)$

무한 루프를 방지하기 위해 base case를 설정한다. 

```swift 
func factorial(number: Int) -> Int {
    if number == 1 {
        return 1
    } 
    
    return number * factorial(n-1)
}
```

- Base Case: $f(1) = 1$ 
- Recurrence Relation: $f(n) = n * f(n-1)$

## 시간복잡도

재귀의 시간복잡도 = 재귀함수 호출 수 * (재귀함수 하나당)시간복잡도

```swift 
func factorial(number: Int) -> Int {
    if number == 1 {
        return 1
    } 
    
    return number * factorial(n-1)
}
```

이 메서드로 보자.

- 재귀함수 호출 수
$=> n$
- 재귀함수 하나 당 시간복잡도
$=> O(1)$
- 재귀의 시간복잡도
$=> O(n*1)$

```swift 
func fibonacci(number: Int) -> Int {
    if number == 1 || number == 2 {
        return 1
    } 
    
    return fibonacci(n-1) + fibonacci(n-2)
}
```

이 메서드로 보자.

- 재귀함수 호출 수
$=> 2^n$

피보나치 수열을 계산하는 과정에서, 각 숫자는 이전 두 숫자의 합으로 정의된다. 예를 들어, 피보나치 수열의 n번째 숫자를 계산하려면 n-1번째와 n-2번째 숫자를 알아야 한다. 

각 호출에서 두 번째 재귀 호출이 발생하기 때문에, 트리 형태로 표현할 때 각 레벨은 두 배씩 늘어나게 된다. 따라서, 호출 트리는 2의 거듭제곱 형태로 증가하게 된다.

- 재귀함수 하나 당 시간복잡도
$=> O(1)$
- 재귀의 시간복잡도
$=> O(2^n*1)$
