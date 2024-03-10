<h1><center> Queue, Priority Queue </center></h1>

###### tags: `💻 TIL`, `Computer Science`, `Data Structre`
###### date: `2024-02-14T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Nossi.DEV](https://www.nossi.dev/8a438391-0de3-4a6c-bd4f-32c38b385521)
> [Heap - 위키백과](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%83%9D)

## Queue, Priority Queue

Queue 자료구조는 시간 순서상 먼저 집어 넣은 데이터가 먼저 나오는 선입선출 FIFO구조로 저장하는 타입이다. 

하지만, Priority Queue는 들어간 순서에 상관없이 우선순위가 높은 데이터가 먼제 나온다. 

**시간복잡도**
1. Queue 
    - enqueue $O(1)$
    - dequeue $O(1)$

2. Priority Queue
    - push $O(logn)$
    - pop $O(logn)$

우선순위큐를 구현하려고 하면 Heap을 구현하면 된다. Heap 자료구조는 완전이진트리를 활용하는 것이다. 

tree가 그려져 있는 상태에서 최대힙, 최소힙의 삽입과 삭제시에 어떻게 node가 삭제되고 연결되는지의 과정을 그려서 설명할 수 있어야 한다.

## Heap

완전이진트리 구조이다. 

- 완전이진트리는 항상 left child node부터 차례차례 할당한다.

**Heap이 되기 위한 조건**

**Max Heap**

![image](https://hackmd.io/_uploads/BJcKpk9oT.png)

- 각 Node에 저장된 값은 child node에 저장된 값보다 크거나 같아야 한다. 

-> Root node에 저장된 값이 가장 크다.

**Min Heap**

![image](https://hackmd.io/_uploads/HJRYT1qsT.png)

- 각 Node에 저장된 값은 child node에 저장된 값보다 작거나 같다(Min Heap - 최소힙)

-> Root node에 저장된 값이 가장 작은 값이 된다.

**NOTE**

> max heap의 Root node는 항상 최댓값이 되고, min heap의 Root node는 항상 최솟값이 된다.

### 구현

트리는 보통 Linked list로 구현한다. 하지만 Heap은 tree임에도 불구하고 array를 기반으로 구현해야 한다. 

그 이유는 새로운 node를 heap의 **마지막 위치**에 추가해야 하는데, 이 때 Array 기반으로 구현해야 이 과정이 수월해지기 때문이다. 

**설정**

![image](https://hackmd.io/_uploads/HyzY4ab2T.png)

- 구현의 편의를 위해 Array의 0 번째 index는 사용하지 않는다. 
- 완전이진트리의 특성을 활용하여 array의 index만으로 부모 자식간의 관계를 정의한다.

> 기준이 되는 n 번째 node는 child node이다. 

- n 번째 node의 left child node = 2n
- n 번째 node의 right child node = 2n + 1
- n 번째 node의 parent node = n / 2

**Heap Method**

#### Push 

- 시간복잡도: $O(logn)$

Heap tree의 높이는 $logN$이다. push()메서드를 call했을 때, swap 하는 과정이 최대 $logN$번 반복되기 때문에 시간복잡도는 $O(logn)$이다. 

![스크린샷 2024-02-20 15.21.42](https://hackmd.io/_uploads/Hyl-Bpb3a.png)

> 기본 트리 구조 

**기본 트리 구조에서 push(190)을 호출하게 될 경우**

![스크린샷 2024-02-20 15.22.30](https://hackmd.io/_uploads/HJbEra-36.png)

> 새로운 데이터를 push하면 heap의 맨 마지막 인덱스에 값을 저장하게 된다.

![스크린샷 2024-02-20 15.23.12](https://hackmd.io/_uploads/SJ9LST-h6.png)

> 부모 Node의 값이 더 작다면 swap메서드를 통해 parent node와 child node를 swap한다.

![스크린샷 2024-02-20 15.23.53](https://hackmd.io/_uploads/H1VFH6-2p.png)

> 위의 과정을 parent Node보다 작은 child node가 될 때까지 반복해서 swap한다. 

#### pop 

- 시간복잡도: $O(logn)$

pop() 메서드를 호출 했을 때, swap하는 과정이 최대 $logN$번 반복되기 때문에 시간복잡도는 $O(logN)$이다.

- 각 Node에 저장된 값은 child node들에 저장된 값보다 크거나 같다(max heap)
=> root node에 저장된 값이 가장 큰 값이 된다.

![스크린샷 2024-02-20 15.27.09](https://hackmd.io/_uploads/HkdHU6W3a.png)

> 첫 번째 인덱스에 저장되어 있는 값을 pop한다.

**첫 번째 인덱스의 값이 pop된 후**

![스크린샷 2024-02-20 15.27.41](https://hackmd.io/_uploads/rkOP8aZ2a.png)

> 마지막 index에 저장된 값을 top으로 옮긴다. 

![스크린샷 2024-02-20 15.28.25](https://hackmd.io/_uploads/Hy75LaZhT.png)

> Top에 저장된 값이 child node보다 작다면 child node들 중 큰 값과 swap 해준다. (190, 170과 비교해 190이 더 크다는 것을 알고 swap)


![스크린샷 2024-02-20 15.29.37](https://hackmd.io/_uploads/SJjALaZh6.png)

> child node가 더 크지 않은 값이 될 때까지 swap과정을 반복한다.

### Swift Heap 구현

```swift 
struct Heap<T: Comparable> {
    var elements: Array<T> = []

    init(data: T) {
        elements.append(data) // 0번 index 채우기용
    }

    mutating func push(_ data: T) {
        elements.append(data)

        let insertIndex = elements.count - 1

        siftUp(from: insertIndex)
    }

    mutating func pop() -> T? {
        guard !elements.isEmpty else {
            return nil
        }

        elements.swapAt(1, elements.count - 1)
        let poppedElement = elements.removeLast()

        siftDown(from: 1)
        return poppedElement
    }
    
    
    func printHeap() {
       var level = 1
       var nodesToPrint = 1
       var nodesPrinted = 1
 
       while nodesPrinted < elements.count {
           var str = ""
           for _ in 0..<nodesToPrint {
               if nodesPrinted < elements.count {
                  str += "\(elements[nodesPrinted]) "
                  nodesPrinted += 1
               }
           }
           print("Level \(level): \(str)")
           nodesToPrint *= 2
           level += 1
       }
     }
}

extension Heap {
    mutating private func siftUp(from index: Int) {
        var childIndex = index
        var parentIndex = getParentNodeIndex(of: childIndex)

        /// max Heap
        /// elements[childIndex] > elements[parentIndex]
        /// min Heap
        /// elements[childIndex] < elements[parentIndex]
        while childIndex > 1 && elements[childIndex] > elements[parentIndex] {
            elements.swapAt(childIndex, parentIndex)
            childIndex = parentIndex
            parentIndex = getParentNodeIndex(of: childIndex)
        }
    }

    mutating private func siftDown(from index: Int) {
        var parentNodeIndex = index

        while true {
            let leftChildNodeIndex = getLeftChildNodeIndex(of: parentNodeIndex)
            let rightChildNodeIndex = getRightChildNodeIndex(of: parentNodeIndex)
            var targetNodeIndex = parentNodeIndex

            /// max Heap
            /// elements[leftChildNodeIndex] > elements[targetNodeIndex]
            /// min Heap
            /// elements[leftChildNodeIndex] < elements[targetNodeIndex]
            if leftChildNodeIndex < elements.count && elements[leftChildNodeIndex] > elements[targetNodeIndex] {
                targetNodeIndex = leftChildNodeIndex
            }

            /// max Heap
            /// elements[rightChildNodeIndex] > elements[targetNodeIndex]
            /// min Heap
            /// elements[rightChildNodeIndex] < elements[targetNodeIndex]
            if rightChildNodeIndex < elements.count && elements[rightChildNodeIndex] > elements[targetNodeIndex] {
                targetNodeIndex = rightChildNodeIndex
            }

            if targetNodeIndex == parentNodeIndex {
                break
            }

            elements.swapAt(parentNodeIndex, targetNodeIndex)

            parentNodeIndex = targetNodeIndex
        }
    }

    private func getParentNodeIndex(of index: Int) -> Int {
        return index / 2
    }

    private func getLeftChildNodeIndex(of index: Int) -> Int {
        return 2 * index
    }

    private func getRightChildNodeIndex(of index: Int) -> Int {
        return (2 * index) + 1
    }
}

```
- 비교가 가능한 데이터면 모두 넣을 수 있도록 Comparable protocol 채택
- 실제 Node는 1번 index부터 시작한다. 