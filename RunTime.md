<h1><center> Run Time </center></h1>

###### tags: `💻 TIL`, `Time Complexity`, `Run Time`
###### date: `2024-01-16T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [시간복잡도 - nossi.dev](https://www.nossi.dev/cote/timecomplexity)

# 개요 

> 남도씨가 만든 Nossi.DEV에서 시간복잡도의 설명을 읽고 따라 써보면서 학습하자. 시간복잡도에 대해 학습하기 전에, Run Time에 대해 알 필요가 있다.

## Run Time

> 프로그램이 시작되고 모든 코드를 실행하는 데 걸리는 시간을 말한다. 

**Run Time에 영향을 주는 요소**

- 한 줄의 코드당 걸리는 시간
- 프로그램 내 실행되는 코드의 개수

**CPU 1tick**당 걸리는 시간을 **1 unit time**이라고 해본다. 한 줄의 코드라도 코드의 종류에 따라 unit time이 다를 수 있다. 변수에 값을 대입하는 코드는 1 unit time이 걸리지만, 두 변수를 더한 값을 변수에 대입하는 코드는 3 unit time이 걸릴 수 있다. 입출력을 하는 코드라면 훨씬 더 많이 걸릴 수 있다.

```swift 
let num1 = 100 // 1 unit time
let num2 = 200 // 1 unit time
let sum = num1 + num2 // 3 unit time
print(sum) // 100 unit time
```

**NOTE**

> CPU 1틱: CPU가 한 번의 기본 작업을 수행하는 데 걸리는 시간을 나타낸다. 이는 CPU의 클럭 속도와 관련이 있다. <br>
> 클럭: 초당 발생하는 전기 신호의 횟수로 표시되며 헤르츠(Hertz)단위로 측정한다. <br>
> 예를 들어, 1GHz(기가 헤르츠)의 CPU는 1초에 10억 번의 클럭 사이클을 실행한다. 따라서 1틱은 이 클럭 사이클 중에서 어떤 하나를 나타내며, 이것이 CPU가 한 번의 기본 작업을 수행하는 데 걸리는 시간을 말한다. 클럭 속도가 높을수록 CPU는 단위 시간당 더 많은 작업을 수행할 수 있다.<br>
> 헤르츠: SI 단위계의 주파수 단위를 말한다. 1Hz는 1초에 한 번을 의미한다. 


이렇게 한줄의 코드당 걸리는 시간은 다를 수 있다. 하지만, CPU입장에서 큰 문제가 되는 크기는 아니다. 코딩테스트를 위한 알고리즘에서는 **프로그램 내에서 실행되는 코드의 개수**만 신경쓰면 된다고 한다. 

```swift 
var sum = 0 // C1 unit time
sum += 1 // C2 unit time
sum += 2 // C3 unit time
sum += 3 // C4 unit time
sum += 4 // C5 unit time
sum += 5 // C6 unit time

print(sum) // C7 unit time

```

한 줄의 코드당 걸리는 시간은 무시해도 된다고 했으니 각각의 unit time을 C1, C2, ..., C7 unit time으로 표현했다. CPU입장에서 매우 작은 숫자니까 하나로 묶어서 봐도 상관이 없으니까 C unit time으로 표현한다. 

```swift 
var sum = 0 // C unit time
sum += 1 // C unit time
sum += 2 // C unit time
sum += 3 // C unit time
sum += 4 // C unit time
sum += 5 // C unit time

print(sum) // C unit time
```

`var sum = 0`부터 `print(sum)`까지 C unit time으로 묶은 것이다.

그렇다면 시간이 많이 걸리는 코드는 무엇인가? 

보통 **반복문**에서 발생할 수 있다. 

```swift 
var sum = 0

for i in 0..<10000000 {
    sum += 1
}

print(sum)
```

for 문을 10,000,000번 반복하면 대략 38ms(0.038s)정도 걸린다. 만약 반복횟수가 * 10번 더 실행되면 1억번 실행되니 207ms로 늘어날 수 있다. 

여기서는 어느정도 걸릴지 예상이 가능하지만 예상할 수 없는 상황도 있다. 

```swift 
var sum = 0 // C1 unit time
var num = Int(readLine()!)! // C1 unit time

for i in 1..<num {
    sum += i // C3 unit time
}

print(sum) // C2 unit time

```
n의 값을 사용자로부터 받는 경우, 반복문이 몇회가 될지 예상할 수 없다. 입력 n에 따라 실행시간이 변하는 경우 runtime을 n에 대한 함수로 표현하면 아래와 같다.

**Formula**
Runtime(n) = C3 * n (C1 + C2) == An + B

여기서 중요한 건 입력된 n의 값에 따라 실행시간이 변한다는 것이다. 입력값 n이외의 상수시간이 걸리는 코드들은 다 묶어서 C unit time으로 통일한다. 

**예제 1)**

```swift 
print("Hello") // C unit time
```

rumtime(n) = C

**예제 2)**

```swift 
for i in 0..<100 {
    print("Hello World") // C1 unit time
}
```

rumtime(n) = 100 * C1 = C

**예제 3)**

```swift 
for i in 0..<100 {
    for j in 0..<100 {
        print("Hello World") // C1 times
    }
}

print("End") // C2 times
```

rumtime(n) = (100 * 100 * C1) + C2 = C

**예제 4)**

```swift 
for i in 0..<n {
    print("Hello World") // C1 times
}

print("End") // C2 times
```

rumtime(n) = (n * C1) + C2 = An + B

**예제 5)**

```swift 
for i in 0..<n {
    for j in 0..<1000 {
        print("Hello World") // C1 times
    }
}
print("End") // C2 times
```

rumtime(n) = (n * 1000 * C1) + C2 = An + B

**예제 6)**

```swift 
for i in 0..<n {
    for j in 0..<n {
        print("Hello World") // C1 times
    }
    
    print("next") // C2 times
}

print("End") // C3 times
```

rumtime(n) = (n * n * C1) + (n * C2) + C3 = A_n^2 + Bn + C

