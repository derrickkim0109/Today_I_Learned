<h1><center> Process </center></h1>

###### tags: `💻 TIL`, `Operation System`
###### date: `2024-02-02T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Process를 간단히 설명해 주세요.](https://www.nossi.dev/17c076cb-c6df-49f4-bd33-4bef52c2aeb8)
> [Process - 위키백과](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4)

## Process

> 프로세스는 컴퓨터에서 연속적으로 실행되고 있는 컴퓨터 프로그램을 말한다. 종종 스케줄링의 대상이 되는 Task라는 용어와 거의 같은 의미로 쓰인다. 

**NOTE**
> 멀티프로세싱: 여러 개의 프로세서를 사용하는 것<br>
> 멀티태스킹: 여러 개의 프로그램을 띄우는 시분할 방식

**프로그램과 프로세스**

프로그램은 일반적으로 하드 디스크 등에 저장되어 있는 실행코드를 뜻하고, 프로세스는 프로그램을 구동하여 프로그램 자체와 상태가 메모리 상에서 실행되는 작업 단위를 지칭한다. 예를 들어, 하나의 프로그램을 여러 번 구동하면 여러 개의 프로세스가 메모리 상에서 실행된다. 

### 프로세스의 상태

커널 내에는 준비 큐, 대기 큐, 실행 큐 등의 자료 구조가 있으며 커널은 이것들을 이용하여 프로세스의 상태를 관리한다. 

![image](https://hackmd.io/_uploads/B1OSioF56.png)

- 생성(created): 프로세스가 생성되는 중이다.
- 실행(running): 프로세스가 CPU를 차지하여 명령어들이 실행되고 있다.
- 준비(ready): 프로세스가 CPU를 사용하고 있지는 않지만 언제든지 사용할 수 있는 상태로, CPU가 할당되기를 기다리고 있다. 일반적으로 준비 상태의 프로세스 중 우선순위가 높은 프로세스가 CPU를 할당받는다. 
- 대기(waiting): 보류(Block)라고 부르기도 한다. 프로세스가 입출력 완료, 시그널 수신 등 어떤 사건을 기다리고 있는 상태를 말한다.
- 종료(terminated): 프로세스의 실행이 종료된다.

## 면접 준비

**Process를 간단히 설명해 주세요.**

실행파일(program)이 메모리에 적재되어 CPU를 할당받아 실행되는 것을 Process라고 합니다.

![image](https://hackmd.io/_uploads/ryrS2iY9a.png)

**메모리에 적재**

메모리는 CPU가 직접 접근할 수 있는 컴퓨터 내부의 기억장치입니다. Program이 CPU에서 실행되려면 해당 내용이 메모리에 적재된 상태여야만 한다. 

프로세스에 할당되는 메모리 공간은 Code, Data, Stack, Heap 4개의 영역으로 이루어져 있으며, 각 Process마다 독립적으로 할당을 받는다. 

![image](https://hackmd.io/_uploads/BJjB2iYca.png)

**Process의 메모리 영역에 대해서 설명해 주세요.**

- Code 영역: 실행한 프로그램의 코드가 저장되는 메모리 영역
- Data 영역: 프로그램의 전역 변수와 static 변수가 저장되는 메모리 영역
- Heap 영역: 프로그래머가 직접 공간을 할당(malloc)/해제(free)하는 메모리 영역
- Stack 영역: 함수 호출 시 생성되는 지역 변수와 매개 변수가 저장되는 임시 메모리 영역

**CPU의 연산과 PC Register**

프로그램의 코드를 토대로 CPU가 실제로 연산을 해야만 프로그램이 실행된다고 볼 수 있다. 그럼 어떤 코드를 읽어야 하는가를 정하는 것은 CPU 내부에 있는 PC(Program Counter) register에 저장되어 있다. PC register에는 다음에 실행될 코드(명령어, instruction)의 주소값이 저장되어 있다. 

즉, 메모리에 적재되어있는 process code영역의 명령어 중 다음번 연산에서 읽어야할 명령어의 주소값을 PC register가 순차적으로 가리키게 되고, 해당 명령어를 읽어와서 CPU가 연산을 하게 되면 Process가 실행이 되는 것이다.