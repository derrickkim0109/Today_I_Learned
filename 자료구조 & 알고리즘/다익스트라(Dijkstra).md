<h1><center> 다익스트라(Dijkstra) </center></h1>

###### tags: `💻 TIL`
###### date: `2024-02-22T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [나무위키 - 다익스트라](https://namu.wiki/w/%EB%8B%A4%EC%9D%B5%EC%8A%A4%ED%8A%B8%EB%9D%BC%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)

> [Nossi.DEV](https://www.nossi.dev/e865a40f-36ab-4a80-908b-c7dfb6f0a98c)

## 다익스트라(Dijkstra)

음의 가중치가 없는 그래프의 한 정점(Vertex)에서 모든 정점까지의 최단 거리를 각각 구하는 알고리즘이다. 

다익스트라가 처음 고안한 알고리즘은 $O(V^2)$의 시간복잡도를 가졌지만, 이후 우선순위 큐(=Heap)등을 이용해서 더욱 개선된 알고리즘 되어 $O((V+E)logV)$의 시간복잡도를 가지게 되었다. 

$O((V+E)logV)$의 시간복잡도를 가지는 이유는 각 노드마다 미방문 노드 중 출발점으로부터 현재까지 계산된 최단 거리를 가지는 노드를 찾는데 $O(VlogV)$의 시간이 필요하고, 각 노드마다 이웃한 노드의 최단 거리를 갱신할 때 $O(ElogV)$의 시간이 필요하기 때문이다. 

그래프 방향의 유무는 상관 없으나, 간선들 중 단 하나라도 가중치가 음수이면 이 알고리즘은 사용할 수 없다. 음의 가중치를 가지는 간선이 있으며, 가중치 합이 음인 사이클이 존재하지 않는 경우 벨만-포드 알고르즘을 사용할 수 있다. 또한, 그래프 내에 가중치 합이 음인 사이클이 존재하여, 무한히 음의 사이클을 도는 경우 경로 합이 음수 무한대로 발산하기 때문에 그래프 내의 최단 경로를 구성할 수 없다. 

- 다익스트라 알고리즘: **하나의 노드로부터 최단경로를 구하는 알고리즘**
- 플로이드-워셜 알고리즘: **가능한 모든 노드쌍들에 대한 최단거리를 구하는 알고리즘**

```swift 
// 그래프의 정점을 나타내는 클래스
final class Vertex: Equatable {
    let name: String
    var edges: [(Vertex, Double)] = [] // (인접 정점, 가중치) 쌍을 저장하는 배열
    var shortestPath = Double.infinity // 최단 경로를 저장하는 변수

    init(name: String) {
        self.name = name
    }

    static func == (lhs: Vertex, rhs: Vertex) -> Bool {
        return lhs.name == rhs.name
    }
}
```

```swift 
// 다익스트라 알고리즘 구현
func dijkstra(start: Vertex, graph: [Vertex]) {
    start.shortestPath = 0
    var queue = graph

    while !queue.isEmpty {
        // shortestPath가 가장 작은 정점을 찾음
        let current = queue.min(by: { $0.shortestPath < $1.shortestPath })!

        // 현재 정점과 연결된 정점들을 확인하고 shortestPath 업데이트
        for edge in current.edges {
            let neighbor = edge.0
            let weight = edge.1
            let distance = current.shortestPath + weight
            if distance < neighbor.shortestPath {
                neighbor.shortestPath = distance
            }
        }

        // 큐에서 현재 정점을 제거
        if let index = queue.firstIndex(of: current) {
            queue.remove(at: index)
        }
    }
}
```

```swift 
// 예시 그래프 생성
let A = Vertex(name: "A")
let B = Vertex(name: "B")
let C = Vertex(name: "C")
let D = Vertex(name: "D")
let E = Vertex(name: "E")

A.edges = [(B, 6), (D, 1)]
B.edges = [(A, 6), (C, 5), (D, 2), (E, 2)]
C.edges = [(B, 5), (E, 5)]
D.edges = [(A, 1), (B, 2), (E, 1)]
E.edges = [(B, 2), (C, 5), (D, 1)]

// 그래프 정점 배열
let graph = [A, B, C, D, E]

// 다익스트라 알고리즘 실행
dijkstra(start: A, graph: graph)

// 결과 출력`ZW1
for vertex in graph {
    print("Shortest path from A to \(vertex.name): \(vertex.shortestPath)")
}
```

![image](https://hackmd.io/_uploads/S1VxzA4hp.png)

**A에서 D로 가는 경로 중 가장 비용이 작게 드는 경로는 뭘까?**

가중치가 없는 그래프는 모든 그래프의 가중치가 같기 때문에 A->B->E->D or A->B->C->D로 가는 경로의 비용이 같았다. 

그러나 가중치 그래프에서는 A->B->E->D로 가는 경로가 더 효율적이다. 왜냐하면 총 비용이 3+5+3=11로 3+8+2=13 보다 작기 때문이다. 

이렇게 더 효율적인 경로를 찾기 위해 다익스트라 알고리즘을 사용해서 해결할 수 있다. 

다익스트라 알고리즘은 가중치 그래프에서 시작점과 도착점이 주어졌을 때, 최소 비용을 return하는 알고리즘이다.

### 다익스트라의 원리

![image](https://hackmd.io/_uploads/B1fTMANh6.png)

> 다음과 같이 그래프가 주어지고 시작점 A, 도착점 H로 가는 경로를 구하자.

1. 각각의 점에 라벨을 매겨 줄 것이다. 시작점의 라벨은 0으로 초기화하고, 나머지 점들은 inifity로 초기화한다. 
2. 사용 안 한 라벨 중 제일 작은 라벨을 찾고, 해당 라벨을 사용했다고 표시해준다. 
3. 해당 점에서 인접한 점들의 라벨들을 업데이트 해준다.(이미 라벨 되어 있는 값보다 작으면 그 값으로 업데이트 해주고, 크면 업데이트 하지 않는다.)
4. 2-3번의 과정을 반복한다.
5. 모든 라벨을 사용했으면 종료한다. 

**다익스트라의 진행과정**

![image](https://hackmd.io/_uploads/B1X_ZoBn6.png)

1. 시작점 A 라벨은 0으로 초기화된다.
2. 나머지 라벨은 Infinity로 초기화

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/Sy0tboS3a.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> A^0$

2. $A^0$ 사용 처리, $A^0$ 인접 정점의 라벨을 업데이트 한다. 
$B^∞ -> B^2$
$D^∞ -> D^1$

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/ByDIzsBnp.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> D^1$

2. $D^1$ 사용 처리, $D^1$ 인접 정점의 라벨을 업데이트 한다. 
$C^∞ -> C^4$
$G^∞ -> G^6$

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/ByuqMiSh6.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> B^2$

2. $B^2$ 사용 처리, $B^2$ 인접 정점의 라벨을 업데이트 한다. 
$C^4 -> C^4$
$E^∞ -> E^4$
$F^∞ -> F^6$

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/SJw6fsBhp.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> C^4$

2. $C^4$ 사용 처리, $C^4$ 인접 정점의 라벨을 업데이트 한다. 

-> 업데이트 할 라벨이 없다.

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/BySZmirna.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> E^4$

2. $E^4$ 사용 처리, $E^4$ 인접 정점의 라벨을 업데이트 한다. 

$H^∞ -> H^5$

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/HkcV7iH36.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> H^5$

2. $H^5$ 사용 처리, $H^5$ 인접 정점의 라벨을 업데이트 한다. 

-> 업데이트 할 라벨이 없다.

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/ryBLmsB2a.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> F^6$

2. $F^6$ 사용 처리, $F^6$ 인접 정점의 라벨을 업데이트 한다. 

<img src="https://hackmd.io/_uploads/Skm5bjS36.png" width="50" height="100">

![image](https://hackmd.io/_uploads/ryBLmsB2a.png)

1. 사용 안 한 라벨 중에서 가장 작은 것은?
$=> G^6$

2. $G^6$ 사용 처리, $G^6$ 인접 정점의 라벨을 업데이트 한다.

모든 라벨들이 업데이트 되었으므로, 알고리즘을 종료한다. 각 라벨은 A에서 H 정점으로 가기 위한 최소 비용 5이다. 

**구현**

```swift 
// 최소 힙 구현
struct Heap <T: Comparable> {
    var heap = [T]()

    private func getParent(_ index: Int) -> T {
        heap[index / 2]
    }

    private func getLeftChild(_ index: Int) -> T {
        heap[index * 2]
    }

    private func getRightChild(_ index: Int) -> T {
        heap[index * 2 + 1]
    }

    func isEmpty() -> Bool {
        heap.count <= 1
    }

    mutating func push(_ data: T) {
        if isEmpty() { heap.append(data) }
        var index = heap.count
        heap.append(data)

        while index > 1 {
            let parent = getParent(index)
            guard parent > data else { break }
            heap[index] = parent
            index /= 2
        }
        heap[index] = data
    }

    mutating func pop() -> T? {
        guard !isEmpty() else { return nil }
        let item = heap[1]
        let data = heap.removeLast()
        let size = heap.count - 1

        guard !isEmpty() else { return item }
        var (parent, child) = (1, 2)
        while child <= size {
            if child < size && getLeftChild(parent) > getRightChild(parent) {
                child += 1
            }
            guard data > heap[child] else { break }
            heap[parent] = heap[child]
            parent = child
            child *= 2
        }
        heap[parent] = data
        return item
    }
}

struct Node: Comparable {
    static func < (lhs: Node, rhs: Node) -> Bool {
        lhs.cost < rhs.cost
    }

    init(_ node: Int, _ cost: Int) {
        self.node = node
        self.cost = cost
    }

    let node: Int
    let cost: Int
}

let input = readLine()!.split { $0 == " " }.map { Int($0)! }, (v, e) = (input[0], input[1])
let start = Int(readLine()!)!
var graph = [Int: [Node]]()
var result = [Int](repeating: Int.max, count: v+1)
for i in 1...v {
    graph[i] = []
}

for _ in 0 ..< e {
    let input = readLine()!.split { $0 == " " }.map { Int($0)! }
    let (start, end, cost) = (input[0], input[1], input[2])
    graph[start]!.append(Node(end, cost))
}

dijkstra(start)
for i in 1...v {
    print(result[i] == Int.max ? "INF" : result[i])
}

func dijkstra(_ start: Int) {
    var queue = Heap<Node>()
    var visited = [Bool](repeating: false, count: v + 1)
    result[start] = 0
    queue.push((Node(start, 0)))

    while let current = queue.pop() {
        let (node, cost) = (current.node, current.cost)
        guard !visited[node] else { continue }
        visited[node] = true
        if let edge = graph[node] {
            for next in edge {
                let (nextNode, nextCost) = (next.node, next.cost)
                guard !visited[nextNode] else { continue }
                let newCost = cost + nextCost
                if newCost < result[nextNode] {
                    result[nextNode] = newCost
                    queue.push(Node(nextNode, newCost))
                }
            }
        }
    }
}
```