<h1><center> 시간복잡도 - Time complexity </center></h1>

###### tags: `💻 TIL`
###### date: `2024-02-01T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [sorted() - Apple Documentation](https://developer.apple.com/documentation/swift/array/sorted())
> [시간복잡도 - 남도씨](https://www.nossi.dev/cote/timecomplexity)

# 개요 

> Run Time에 이어서 Nossi.Dev에서 시간복잡도에 대해 학습해보자.

## 시간복잡도

> 문제를 해결하는데 걸리는 시간과 입력값 n의 함수 관계를 말한다.

이제 Runtime(n)값을 토대로 `Big-O`표기법에 대해 알아보자

### Big-O

> input n의 크기가 커지만 작은 차수의 항들이 runtimed에 미치는 영향에 대해 무시한다. 즉, 작은차수, 계수는 무시한다.

**최고차항부터 최저차항**

$n! > 2^n > n^3 > n^2 > nlong_n > n > log_n > 1$

![image](https://hackmd.io/_uploads/H1mcrYu9p.png)

$O(1)$

```swift 
var a = 0
a += 1
```

$O(n)$

```swift 
for i in 0..<n {
    print("Hello")
}
```

$O(n^2)$

```swift 
for i in 0..<n {
    for j in 0..<n {
        print("Hello")
    }
}
```

$O(2^n)$
- 재귀함수 (recursion)

```swift 
func dp(_ num: Int) -> Int {
    if num == 1 {
        return 0 
    } else if (num == 2) {
        return 1 
    } else {
        return dp(num - 1) + dp(num - 2)
    }
}
```

$O(log_n)$
- 탐색해야하는 데이터가 절반씩 줄어든 함수
    - 이진탐색 알고리즘

```swift 
func binarySearch(_ data: [Int], num: Int, target: Int) -> Int {
    var start = 0 
    var end = n 
    var mid = 0 
    
    while(end >= start) {
        mid = (end + start) / 2
        
        if data[mid] == target {
            return 1
        } else if (data[mid] > target) {
            end = mid - 1
        } else {
            start = mid + 1
        }
    }
    
    return 0
}
```

## 코테를 위한 시간복잡도

> 외워라!

1. sorted()
정렬은 $O(nlog_n)$의 시간 복잡도를 가진다. 

2. Dictionary

구축: $O(n)$
검색: $O(1)$

Swift에서 Dictionary는 HashTable을 사용해서 구현된 자료구조이다. 검색의 시간복잡도가 $O(1)$이다. hashtable은 공간(메모리)를 사용함으로써 시간을 절약할 수 있다.

3. Binary Search

$O(logn)$

**정렬된 배열**에서 특정 숫자를 찾는 알고리즘이다. 반복문, 재귀 함수가 재호출될 때마다 탐색해야할 데이터의 크기가 절반씩 줄어들기 때문에 $O(logn)$의 시간복잡도가 된다.

**만약 정렬이 안된 배열이라면,** 정렬($O(nlogn))$을 먼저 해야함.

4. Heap (Priority Queue)

Swift에서는 따로 제공하지 않는다. 직접 구현해야함

- 길이가 n인 배열을 heap으로 만들 때 $O(nlongn)$의 시간복잡도를 가진다.
- push()메서드 $O(logn)$
- pop()메서드 $O(logn)$

