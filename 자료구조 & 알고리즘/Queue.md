<h1><center> Queue </center></h1>

###### tags: `💻 TIL`, `Computer Science`, `Data Structre`
###### date: `2024-02-14T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Nossi.DEV](https://www.nossi.dev/c7d72557-2e54-4523-9946-9c4b83ca5044)
> [Queue - 위키백과](https://ko.wikipedia.org/wiki/%ED%81%90_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0))

## Queue

![image](https://hackmd.io/_uploads/BkVQvy5oT.png)

집어넣은 데이터가 먼저 나오는 FIFO(First In First Out)구조로 저장하는 자료구조를 말한다. 

집어 넣은 데이터가 먼저 나오는 스택과는 반대되는 개념이다.

큐는 put(insert)와 get(delete)을 이용하여 구현된다.

front(head)와 rear(tail)는 데이터의 위치를 가리키는데 front는 데이터를 get할 수 있는 위치를 의미하고, rear는 데이터를 put할 수 있는 위치를 의미한다. 

또한, 큐가 꽉 차서 더 이상 자료를 넣을 수 없는 경우(put할 수 없는 경우)를 **오버플로우(Overflow)**, 큐가 비어 이쓴 ㄴ자료를 꺼낼 수 없는 경우(get할 수 없는 경우)를 **언더플로우(Underflow)**라고 한다.

> put은 큐에 자료를 넣는 것 <br>
> get은 큐에서 자료를 꺼내는 것

**시간복잡도**
- enqueue $O(1)$
- dequeue $O(1)$

**활용 예시**

- Cache 구현
- 프로세스 관리
- 너비우선탐색(BFS)

### 구현 방식

![image](https://hackmd.io/_uploads/r1eNukqo6.png)

- Array-Based Queue: enqueue와 dequeue과정에서 남는 메모리가 생긴다. 따라서 메모리의 낭비를 줄이기 위해 주로 Circular queue형식으로 구현된다.
- List-Based: 재할당이나 메모리 낭비의 걱정을 할 필요가 없어진다. 

**Circular Queue**

![스크린샷 2024-02-14 16.15.29](https://hackmd.io/_uploads/Hks5_1qs6.png)

이런 자료구조가 있다고 할 때

![스크린샷 2024-02-14 16.15.45](https://hackmd.io/_uploads/ryosdk5oT.png)

하나씩 dequeue를 하고 

![스크린샷 2024-02-14 16.16.04](https://hackmd.io/_uploads/HkA2_k9ia.png)

enqueue(7)을 하면 이런 방식으로 이어질 것이다.

![스크린샷 2024-02-14 16.16.29](https://hackmd.io/_uploads/rkPROJ5sp.png)

그렇게 마지막 10을 enqueue하고 

![스크린샷 2024-02-14 16.16.53](https://hackmd.io/_uploads/SJ1xK15j6.png)

그 다음으로 11을 enqueue하는 경우 0 번째 인덱스로 돌아와서 데이터를 저장하게 된다.

**Array-Base와 List-Base는 어떤 차이가 있나?**

1. 메모리 관리
    - Array-Based Queue: 고정된 크기의 배열을 사용하므로 enqueue와 dequeue를 사용해 이러한 문제를 해결할 수 있다.
    - List-Based Queue: 연결 리스트를 사용해 메모리를 동적으로 할당하고 해제할 수 있다. 이로 인해 메모리 사용이 더 효율적이다.

2. 접근 시간
    - Array-Based Queue: 배열은 인덱스를 사용해 Random Access가 가능하므로 enqueue 및 dequeue작업은 $O(1)$의 시간복잡도를 가진다. 하지만 배열의 크기가 고정되어 있기 때문에, 배열이 꽉 찬 경우 새로운 배열을 할당하고 모든 요소를 복사해야 하는 추가 작업이 필요할 수 있다.
    - List-Based Queue: 연결 리스트의 경우 삽입과 삭제가 일반적으로 $O(1)$의 시간복잡도를 가진다. 연결 리스트의 특성 상 삽입 및 삭제 작업은 해당 노드에 직접 연산을 수행하므로 배열과 같은 메모리 복사 작업이 필요하지 않다.

3. 크기의 유연성
    - Array-Based Queue: 배열 크기가 고정되어 있으므로 큐의 크기를 동적으로 조절할 수 없다.
    - List-Based Queue: 연결 리스트는 동적으로 크기를 조절할 수 있으므로 크기의 유연성이 있다.

4. 메모리 오버헤드
    - Array-Based Queue: 배열은 각 요소를 저장하기 위해 추가적인 메모리 공간을 필요로 한다. 또한, 크기가 고정되어 있으므로 메모리를 미리 할당해야 한다. 
    - List-Based Queue: 연결 리스트는 각 노드에 대한 포인터를 저장해야 하므로 포인터 오버헤드가 발생할 수 있다. 하지만, 동적 할당의 유연성을 통해 메모리를 더 효율적으로 사용할 수 있다.
