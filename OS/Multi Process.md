<h1><center> Multi Process </center></h1>

###### tags: `💻 TIL`, `Computer Science`, `Operation System`
###### date: `2024-02-01T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Nossi.DEV](https://www.nossi.dev/0d7981c6-1aae-4471-aae9-2dfa3051e2c5)

## Multi Process 

> 2개 이상의 Process가 동시에 실행되는 것을 말한다. 동시에라는 말은 동시성(concurrency)와 병렬성(parallelism) 두 가지를 의미한다. 

- 동시성: CPU core가 1개 일 때, 여러 process를 짧은 시간동안 번갈아 가면서 연산을 하게 되는 시분할 시스템(time sharing system)으로 실행되는 것이다. 
- 병렬성: CPU core가 여러 개이고, 각 core가 여러 process를 연산함으로써 process가 동시에 실행되는 것을 말한다. 

메모리의 경우 여러 프로세스들이 각자의 메모리 영역을 차지하여 동시에 적재된다. 

반면에, 하나의 CPU는 매 순간 하나의 프로세스만 연산할 수 있다. 하지만 CPU의 처리 속도가 빨라서 n ms 이내의 짧은 시간동안 여러 프로세스들이 CPU에서 번갈아 실행되기 때문에 사용자 입장에서는 여러 프로그램이 동시에 실행되는 것처럼 보인다. 

이처럼 CPU의 작업시간을 여러 프로세스들이 조금씩 나누어 쓰는 시스템을 시분할 시스템이라고 부른다. 

![스크린샷 2024-02-02 20.15.47](https://hackmd.io/_uploads/H1CJ1I996.png)

### 메모리 관리

여러 프로세스가 동시에 메모리에 적재된 경우, 서로 다른 프로세스의 영역을 침범하지 않도록 각 프로세스가 자신의 메모리영역에만 접근하도록 운영체제가 관리해준다고 한다. 

이때, 알아야 하는 것은 각 프로세스는 가상 메모리를 만들어서 각 함수 별로 필요한 자원들이 번갈아 가면서 물리 메모리에 적재되는 것이다. 만약에 한꺼번에 각 프로세스 별로 가상 메모리에 올라간 것들이 물리 메모리에 할당되면 메모리 공간은 남지 않을 것이다. 이런 반복 작업을 운영체제가 관리해주는 것이다. 

![image](https://hackmd.io/_uploads/rJWWlLccT.png)

### CPU의 연산과 PC Register

CPU는 PC(Program Counter) register가 가리키고 있는 명령어를 읽어들여 연산을 진행한다. PC Register에는 다음에 실행될 명령어의 주소값이 저장되어 있다. multi process 시스템에서는 프로세스 1이 진행되고 있을 때 프로세스 1의 코드 영역을 PC register가 가리키다가, 프로세스 2가 진행되면 프로세스 2의 코드 영역을 가리키게 된다. CPU는 PC Register가 가리키는 곳에 따라 프로세스를 변경해 가면서 명령어를 읽어들이고 연산한다. 

### Context

시분할 시스템에서는 한 프로세스가 매우 짧은 시간동안 CPU를 점유하여 일정부분의 명령을 수행하고, 다른 프로세스에게 넘긴다. 그 후 차례가 되면 다시 CPU를 점유하여 명령을 수행한다. 따라서 이전에 어디까지 명령을 수행했고, register에는 어떤 값이 저장되어 있는지에 대한 정보가 필요하게 된다. 

프로세스가 현재 어떤 상태로 수행되고 있는지에 대한 총체적인 정보가 바로 context이다. 

context정보들은 PCB(Process Control Block)에 저장된다.

- 반효경 교수님의 운영체제 세 번째 강의에서도 같은 설명을 한다. 다시 반복해보자.

### PCB(Process Control Block)

PCB는 운영체제가 프로세스를 표현한 자료구조이다. PCB에는 프로세스의 중요한 정보가 포함되어 있기 때문에, 일반 사용자가 접근하지 못하도록 보호된 메모리 영역 안에 저장된다. 일부 운영 체제에서 PCB는 커널 스택에 위치한다. 이 메모리 영역은 보호를 받으면서도 비교적 접근하기 편리하기 때문이다.

커널 스택은 각 프로세스 별로 생성된다. 

PCB에는 아래 정보들이 포함된다. 

- Process State: new, running, waiting, halted 등의 state가 있다.
- Process Number: 해당 프로세스의 number
- Program Counter(PC): 해당 프로세스가 다음에 실행할 명령어의 주소를 가리킨다. 
- Registers: 컴퓨터 구조에 따라 다양한 수와 유형을 가진 Register 값들이다.
- Memory limits: base register, limit register, page table 또는 segment table 등 

![image](https://hackmd.io/_uploads/HJ7SG89cp.png)


### Context Switch

Context Switch란 한 프로세스에서 다른 프로세스로 CPU 제어권을 넘겨주는 것을 말한다. 

이때 이전의 프로세스의 상태를 PCB에 저장하여 보관하고 새로운 프로세스의 PCB를 읽어서 보관된 상태를 복구하는 작업이 이루어진다. 

![image](https://hackmd.io/_uploads/BJpKGI9cT.png)
