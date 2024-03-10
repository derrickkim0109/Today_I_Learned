<h1><center> List - Linked List </center></h1>

###### tags: `Algorithm`
###### date: `2024-02-05T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [List - Nossi.DEV](https://www.nossi.dev/749e7a89-e312-4a52-9e9b-55ea87fff4e9)

## List 

> List는 순서를 가지고 원소들을 저장하는 자료구조를 말한다. Sequence라고도 부른다. 구현방법으로는 Array, Linked List가 있다. swift에서는 Dynamic Array가 있다.

### Array List 

> 배열을 기반으로 구성된 List 자료구조를 말한다. static array로 구현할 수 있고, Dynamic Array로도 구현할 수 있다. 

### 배열 Static Array

**특성**

1. 고정된 저장 공간(fixed-size)
2. 순차적인 데이터 저장(order)

배열은 선언 시 size를 정해 해당 size만큼의 연속된 메모리를 할당 받아 data를 연속적/순차적으로 저장하는 자료구조이다. 

```c
int arr[5] = {3, 7, 4, 2, 6} // size가 5인 int형 배열 선언
```

위의 예시에서는 배열의 size를 5고 정했기 때문에 int형 데이터(4 byte) 5개를 저장할 메모리 공간인 20 byte를 미리 할당 받는다. 이처럼 고정된 size를 갖게 되기 때문에 static array라고 부르기도 한다. 또한, 배열의 데이터를 연속적이고 순차적으로 메모리에 저장한다. 

**Random Access**

메모리에 저장된 데이터에 접근하려면 주소값을 알아야 한다. 배열 변수는 자신이 할당받은 메모리의 첫번째 주소값을 가리킨다. 

배열은 연속적/순차적으로 저장되어 있기 때문에 첫 주소값만 알고 있다면 어떤 index에도 즉시 접근이 가능하다. 이를 direct access 또는 random access라고 부른다. 

예를 들어, 첫 번째 데이터가 저장된 주소값이 0x4AF55라면 2 번째 데이터는 0x4AF55 + 4 * 1(byte)에 저장되어 있을 것이다. 3번째 데이터는 0x4AF55 + 4 * 2에 저장되어 있을 것이고, n번째 데이터는 0x4AF55 + 4 * (n - 1)에 저장되어 있을 것이다. 

아무리 긴 배열이더라도 한번의 연산으로 원하는 데이터에 바로 접근할 수 있다. 즉, $O(1)$의 시간복잡도를 갖는다.

**NOTE**

> Link List는 메모리상에서 불연속적으로 저장되어 있기 때문에 Random Access가 불가능 하다. n 번째 데이터에 접근하기 위해서 Array는 한 번의 연산으로 되지만 $O(1)$, Linked List는 n번의 연산을 해야 하기 때문에 시간복잡도 $O(n)$이 된다.

**고정된 저장 공간**

데이터의 개수가 정해져 있는 경우, static array를 사용하는 것이 매우 효율적이다. 

하지만, 선언시에 정한 size보다 더 많은 데이터를 저장해야 하는 경우, 공간이 없어 문제가 발생할 수 있다. 그렇다고 매번 크기가 엄청 큰 배열을 선언한다면 그 만큼 메모리가 비효율적으로 할당받게 된다. 이런 문제를 해결할 수 있는 것이 Dynamic Array이다.
