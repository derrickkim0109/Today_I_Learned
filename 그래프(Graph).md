<h1><center> 그래프(Graph) </center></h1>

###### tags: `💻 TIL`
###### date: `2024-02-22T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [위키백과 - 그래프](https://ko.wikipedia.org/wiki/%EA%B7%B8%EB%9E%98%ED%94%84_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0))

> [Nossi.DEV- 그래프](https://www.nossi.dev/dfdd6c3d-46ac-4573-b612-00017e9fdd56)

## 그래프(Graph)

![image](https://hackmd.io/_uploads/SkyJDvVna.png)

> 그래프는 vertex들의 집합 V와 이들을 연결하는 edge들의 집합 E로 구성된 자료구조를 말한다. 

- vertex: 정점
- edge: 정점과 다른 정점을 연결하는 간선

정점: A, B, C, D, E, F
- $V = {A, B, C, D, E, F}$

간선: A-B, B-C, B-E, C-D, E-D, E-F, D-F

- $E = {(A,B), (B,C), (B,E), (C,D), (E,D), (E,F), (D,F)}$

**그래프의 예시**

- 도시들을 연결하느 도로망: 도시(vertex), 도로망(edge)
- 지하철 연결 노선도: 정류장(vertex), 정거장을 연결한 선(edge)
- 컴퓨터 네트워크: 각 컴퓨터와 라우터(vertex), 라우터간의 연결관계(edge)

### 그래프의 종류 

**1. 무향 그래프(undirected graph)**

![image](https://hackmd.io/_uploads/ryM8uDEh6.png)

위와 같은 그래프는 $A->B$, $B->A$ 두 방향 다 갈 수 있기에 무향 그래프라고 한다. 따로 방향이 안 정해져 있기 때문이다.

-> 코테에 자주 나온다

**2. 방향 그래프(directed graph)**

![image](https://hackmd.io/_uploads/HyxTuPN26.png)

그래프를 보면, 들어오는 간선, 나가는 간선으로 구분할 수 있다.

- indegree: 들어오는 간선
- outdegree: 나가는 간선

정점 B를 살펴보면 A와 C로부터 들어오는 간선은 2개, E로 나가는 간선 1개가 있다. 

**3. 다중그래프**

![image](https://hackmd.io/_uploads/H15NKvE3T.png)

A에서 B까지 가능 방법이 총 3개 있다. 다시 말해, 인접해 있는 두 정점 A, B 사이에 A를 기준으로 indegree 3개, outdegree 3개가 있다. 

이렇게 indegree, outdegree가 2개 이상인 그래프를 다중 그래프라고 한다. 

-> 코테에서는 단순 그래프를 주로 다룬다.

![image](https://hackmd.io/_uploads/HJocYDN26.png)

> 단순 그래프

**4. 가중치 그래프**

![image](https://hackmd.io/_uploads/H1vaFD43a.png)

각 경로마다 가중치가 다르다하여 가중치 그래프라고 한다. 

-> 가중치 그래프는 다익스트라 알고리즘에서 쓰인다. 

## 그래프 구현

- 인접 행렬
- 인접 리스트
- 암시적 그래프

1. 인접 행렬

![image](https://hackmd.io/_uploads/HJocYDN26.png)

> 위와 같은 그래프가 있을 때, 각 정점들을 하나의 행(가로)과 열(세로)이라고 생각할 수 있다.

![스크린샷 2024-02-22 15.39.08](https://hackmd.io/_uploads/SyLMhvV26.png)

- 각 정점끼리 연결되어 있으면 1
- 연결되어 있지 않으면 0

![스크린샷 2024-02-22 15.39.44](https://hackmd.io/_uploads/Hy94hwNha.png)

> 행렬

![스크린샷 2024-02-22 15.40.42](https://hackmd.io/_uploads/S14_2w42p.png)

흔히 행렬을 만들기 위해서 이중 배열을 사용한다. 

```swift 
let matrix = [[1, 2, 3], [4, 5, 6]]
```


인접행렬을 이중 배열로 표현하면 아래와 같다.

```swift 
let matrix = [
    [0, 1, 0, 0, 0, 0],
    [1, 0, 1, 0, 1, 0],
    [0, 1, 0, 1, 0, 0],
    [0, 0, 1, 0, 1, 1],
    [0, 1, 0, 1, 0, 1],
    [0, 0, 0, 1, 1, 0]
]
```
위 그래프에서 특징적인 점은 자기 자신(ex: A-A, B-B)을 연결하는 간선이 없기 때문에 대각선의 값이 모두 0이라는 점과 **무향 그래프**이기 때문에 대각선을 기준으로 대칭이라는 점이다.

이제 모든 그래프에 대해 인접 행렬을 그릴 수 있다. 

![스크린샷 2024-02-22 15.44.40](https://hackmd.io/_uploads/By7PTD4na.png)

> 정점은 많고 간선의 개수가 적을 때는 비효율 적이다.

A-B의 관계를 제외하고 모두 0으로 나타내기 때문에, 메모리 사용 측면에서 비효율적이라고 할 수 있다. 

**그래프는 연결 관계를 나타내는 것이 중요한데, 0은 정보로서 가치가 떨어지기 때문이다.**

2. 인접 리스트

인접 리스트는 `Dictionary`로 정의할 수 있다. 

- 정점: key
- 연결관계: value

![image](https://hackmd.io/_uploads/SkpCpvVh6.png)

```swift
let grahp: [String: [String]] = [
    "A": ["B"],
    "B": ["A", "C", "E"],
    "C": ["B", "D"],
    "D": ["C", "E", "F"],
    "E": ["B", "D", "F"],
    "F": ["D", "E"]
]
```

![스크린샷 2024-02-22 15.49.55](https://hackmd.io/_uploads/BJRc0vVnT.png)

위에서 인접 행렬은 정점은 많고 간선이 적은 경우 대부분의 정보가 0으로 채워지기 때문에 비효율이지만 인접리스틀 사용하게 되면 이를 간편하게 표현할 수 있다.

```swift
let grahp: [String: [String]] = [
    "A": ["B"],
    "B": ["A"],
    "C": [],
    "D": [],
    "E": [],
    "F": []
]
```
-> 코테에서는 인접 행렬보단 주로 인접리스트를 사용한다.

3. 암시적 그래프

> 코딩 테스트에서 제일 많이 활용된다. 

![image](https://hackmd.io/_uploads/BJtO6OV2p.png)

위의 이미지에서 흰색 칸은 길을 나타내고, 검정색 칸은 벽을 나타내는 미로이다.

![image](https://hackmd.io/_uploads/Bkhy0d42a.png)

암시적으로 그래프로 표현하면 위와 같이 표현할 수 있다.

- 벽: 1
- 길: 0

![image](https://hackmd.io/_uploads/rkXf0_Enp.png)

> 좌표로 표현한 이미지

```swift 
let graph = [
    [1, 1, 1, 1, 1],
    [0, 0, 0, 1, 1],
    [1, 1, 0, 1, 1],
    [1, 0, 0, 0, 0],
    [1, 1, 1, 1, 1]
]
```

위와 같은 표현 방식은 연결관계를 직접적으로 나타내지는 않는다. 하지만, 상하좌우가 연결되어 있다는 것을 암시적으로 알 수 있다. 예를 들어, (1,2)는 상 (0,2), 하(2,2), 좌(1,1), 우(1,3)와 연결되어 있다. 

![image](https://hackmd.io/_uploads/B1uqR_Ehp.png)

우리는 row, col이라는 변수로 그래프의 값에 접근할 것이다. 

- 가로: row
- 세로: col

> graph[row][col]로 원하는 값에 접근한다.
예) (2,3)에 접근하려면, graph[2][3]이다.

## 그래프 순회

그래프의 순회는 트리의 순회와 마찬가지로, 모든 정점을 지나야 한다. 

- BFS
- DFS

**시간복잡도**

> BFS, DFS 순회방법은 모든 정점($V$)들을 탐색해야 하고 그러기 위해서 정점에 연결된 edge($E$)들을 모두 확인해야 한다. 

- $O(V + E)$

**Breadth First Search**

트리의 `level order traversal`와 유사하다. 그래프 순회를 할 때 주어지는 시작점을 루트 노드라 생각하고 level 별로 탐색하는 것을 BFS라 한다.

```swift 
// 그래프를 표현하는 방법
let graph: [String: [String]] = [
    "A": ["B", "C"],
    "B": ["A", "D", "E"],
    "C": ["A", "E"],
    "D": ["B"],
    "E": ["B", "C"]
]


func BFS(start: String, graph: [String: [String]]) -> [String] {
    var visited: [String] = []
    var queue: [String] = [start]

    while !queue.isEmpty {
        let node: String = queue.removeFirst()

        if visited.contains(node) {
            continue
        }

        visited.append(node)
        queue += graph[node] ?? []
    }

    return visited
}

BFS(start: "A", graph: graph)
```

**Depth First Search**

DFS는 출발점에 시작해서, 막다른 지점에 도착할 때까지 깊게 이동한다. 만약 가다가 막히면 다시 이전 노드로 돌아가고, 길이 있으면 깊게 이동하는 식의 과정을 통해 그래프를 순회한다.

Tree의 inorder, preorder, postorder 순회 방식과 유사하다.

![image](https://hackmd.io/_uploads/ryhTNFV2T.png)

> A -> B -> C -> D -> E -> G -> H -> I -> F

![image](https://hackmd.io/_uploads/rJ21BFV3p.png)

우선 깊게 파고 들어 D까지 이동한다. 그 이후 알파벳 순서를 기준으로 E를 먼저 방문한다. 

![image](https://hackmd.io/_uploads/HkvMHKN3T.png)

I까지 방문 후 막혔으니 다시 왔던 길로 돌아가는 과정을 거친다. H, G, E, D 순으로 올라간다. 

그 이후, 아직 방문하지 않은 F를 방문한다. 

![image](https://hackmd.io/_uploads/B1MBrt4h6.png)

F를 방문하고 나면 막혀있으므로 탐색을 끝낸다. 

```swift 
func DFS(start: String, graph: [String: [String]]) -> [String] {
    var visited: [String] = []
    var stack: [String] = [start]

    while !stack.isEmpty {
        let node: String = stack.removeLast()

        if visited.contains(node) {
            continue
        }

        visited.append(node)
        stack += graph[node] ?? []
    }

    return visited
}

// 예시 그래프 생성
let graph: [String: [String]] = [
    "A": ["B", "C"],
    "B": ["A", "D", "E"],
    "C": ["A", "E"],
    "D": ["B"],
    "E": ["B", "C"]
]

// DFS 실행
DFS(start: "A", graph: graph)
```