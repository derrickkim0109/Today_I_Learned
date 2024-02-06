<h1><center> Array </center></h1>

###### tags: `💻 TIL`, `Computer Science`, `DataStructure`
###### date: `2024-02-06T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Nossi.DEV - Array](https://www.nossi.dev/ba77bf13-fdd7-4d3c-a88e-9b54f749434c)
> [Nossi.DEV - Linked List](https://www.nossi.dev/e2a9c68a-98d4-477a-8369-fb6dc367df2b)

## Array

> Array는 연관된 데이터를 메모리상에 순차적으로 미리 할당된 크기만큼 저장하는 자료구조이다. 

**특징**

- 고정된 저장공간 (Fixed-Size)
- 순차적으로 저장되는 데이터

**장점**

Array의 append가 빠르다는 점이다. 따라서, 조회를 자주 해야되는 작업에서 Array 자료구조를 많이 쓴다고 한다. 

**단점**

Fixed-Size 특성상 선언시에 크기를 미리 정해야 한다는 점이다. 이는 메모리 낭비나 추가적인 오버헤드가 발생할 수 있다. 

**Function 시간복잡도**

- append : $O(1)$
- append : $O(1)$
- contains(where:) : $O(N)$
- insert(_:at:) : $O(N)$
- remove(at:) : $O(N)$
- removeFirst() : $O(N)$
- removeLast() : $O(1)$

### 미리 예상한 것보다 더 많은 수의 Data를 저장하느라 Array의 Size를 초과하는 경우, 어떻게 해결해야 하나?

1. 동적 배열(Dynamic Array) 사용

- 정적인 크기를 가진 배열 대신 동적 배열을 사용한다. 하지만, Swift에서는 Array가 동적 배열로 자동으로 관리되므로, 일반적으로 이 문제는 발생하지 않는다. 

2. Linked List 사용

- 배열 대신 Linked List를 사용하여 데이터를 저장한다. 이렇게 되면 동적으로 크기가 조절되므로, 데이터 양에 따라 메모리를 동적으로 할당할 수 있다. 

3. Collection Data Structure 사용

- Swift에서는 배열 이외에도 Set, Dictionary와 같은 컬렉션 데이터 구조를 제공한다. 

4. 데이터베이스 사용
- 대량의 데이터를 효과적으로 관리하려면 데이터베이스를 고려할 수 있다. Core Dat와 같은 데이터베이스를 사용하면 데이터를 영구적으로 저장하고 관리할 수 있다.

## Linked List

> Node라는 구조체로 이루어져 있는데, Node는 데이터 값과 다음 Node의 address를 저장한다. Linked List는 물리적인 메모리상에서는 비연속적으로 저장되지만 Linked List를 구성하는 각 Node들이 Next Node의 address를 가리킴으로써 논리적인 연속성을 가진 자료구조이다.

- Tree, Graph 등의 다른 자료구조를 구현할 때 자주 쓰이는 자료구조이다. 

- 물리 메모리 상에서는 불연속적으로 데이터가 저장되는 점과 Node의 Next Address를 통해 불연속적인 데이터를 연결하여 논리적 연속성을 보장한다는 점

- 데이터가 추가되는 시점에서 메모리를 할당하기 때문에 좀 더 효율적으로 사용할 수 있다

### 논리적 연속성

각 Node들은 Next Address에 대한 정보를 가지고 있기 때문에 논리적으로 연속성을 유지하면서 연결되어 있다. 

Array의 경우 연속성을 유지하기 위해 물리적 메모리 상에서 순차적으로 저장하는 방법을 사용했지만, Linked List는 메모리에서 연속성을 유지하지 않아도 되기 때문에 메모## 다이리 사용이 좀 더 자유로운 대신, Next Address를 추가적으로 저장해야 하기 때문에 데이터 하나당 차지하는 메모리가 더 커지게 된다. 

![스크린샷 2024-02-06 15.15.26](https://hackmd.io/_uploads/SygcKRrJip.png)

> 물리적 불연속성

![image](https://hackmd.io/_uploads/ByBuCS1o6.png)

> 논리적 연속성

### 시간복잡도

Array의 경우 중간에 데이터를 삽입 / 삭제하게 되면 해당 인덱스의 뒤에 있는 모든 원소들이 이동해야 했다. 그러다 보니 $O(n)$의 시간복잡도를 갖게 되었다. 

하지만, Linked List를 물리적으로 옮길 필요없이 Next Address가 가리키는 주소값만 변경하면 되기 때문에 $O(1)$의 시간복잡도로 삽입 / 삭제가 가능하다.

### Array vs Linked List

**Array**

- 메모리상에서 연속적으로 데이터를 저장하는 자료구조
- 데이터 조회 $O(1)$
- 삽입 / 삭제 $O(n)$
- 얼마만큼의 데이터를 저장할지 미리 알고 있고, 조회를 많이 한다고 했을 때 사용하면 좋다.

**Linked List**

- 메모리상에서는 연속적이지 않지만, 각각의 원소가 다음 원소의 메모리 주소값을 저장해 놓음으로써 연속성을 유지
- 데이터 조회 $O(n)$
- 삽입 / 삭제 $O(1)$
- 몇개의 데이터를 저장할 지 불확실하고 삽입, 삭제가 많이 된다면 사용

**Memory Allocation**

Array의 주된 장점은 데이터 접근과 append가 빠르다는 것이다. 하지만, 메모리 낭비라는 단점이 있다. 배열은 선언시에 fixed size를 설정하여 메모리를 할당한다. 즉, 데이터가 저장되어 있지 않더라도 메모리를 차지하고 있기 때문에 메모리 낭비가 발생한다. 

반면, Linked List는 Runtime 중에 Size를 늘리고 줄일 수 있다. 그래서 initial size를 고민할 필요가 없고 필요한 만큼 메모리에 할당하여 낭비가 없다.

## Dynamic Array 

Array의 경우 size가 고정된다고 했다. 그래서 선언시에 설정한 size보다 많은 개수의 Data가 추가되면 저장할 수 없다. 

이에 반해, Dynamic Array는 저장공간이 가득 차게 되면 resize를 하여 유동적으로 size를 조절하여 데이터를 저장하는 자료구조이다. 

- 기존 Array의 특징 중 Fixed-size의 한계점을 보완하고자 고안된 자료구조이다.
    - resize하는 방식
    - 데이터를 추가할 때의 시간복잡도

![image](https://hackmd.io/_uploads/HyR5k8kop.png)

Dynamic Array는 size를 자동적으로 Resizing하는 배열이다. 기존에 고정된 size를 가진 Static Array의 한계점을 보완하고자 고안되었다. 

Dynamic Array는 Data를 계속 추가하다가 기존에 할당된 Memory를 초과하게 되면, size를 늘린 배열을 선언하고 그곳에 데이터를 옮김으로써 늘어난 크기의 사이즈를 가진 배열이된다. 이를 resize라고 한다. 

Resizing을 하는 방법에는 여러가지가 있는데, 대표적으로 기존 Array Size의 2배 사이즈를 할당하는 Doubling이 있다. 

**Doubling**

resize의 대표적인 방법이다. 데이터를 추가하다가 메모리를 초과하게 되면 기존 배열 size보다 두 배 큰 배열을 선언하고 데이터를 일일이 옮기는 방법이다.

**분할상환 시간복잡도(Amortize time complexity)**

Dynamic Array에 데이터를 추가할 때마다, $O(1)$의 시간이 걸린다. 

-> 추가를 하다가 미리 선언된 사이즈를 넘기는 순간에 Resize를 하게 된다. 

-> 이 때는 일일이 데이터를 모두 옮겨야 되기 때문에 이때만큼은 $O(n)$의 시간이 걸린다. 

**그렇다면 결과적으로 append의 시간복잡도는 $O(1)$일까 $O(n)$일까요?**

append의 총 과정을 살펴보면 데이터를 마지막 인덱스에 추과하는 ($O(1)$)작업이 대다수이고, size를 넘어설 때는 size를 두 배 늘리고 데이터를 일일이 옮기는 과정(resize - $O(n)$)이 아주 기끔 발생한다. 

결론은 append의 전체적인 시간복잡도는 $O(1)$이다.