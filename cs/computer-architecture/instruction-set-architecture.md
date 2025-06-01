# Instruction Set Architecture

## 기계어

* MIPS, ARM 등이 존재한다.
* 사람이 기계어로 프로그래밍을 하는 것은 사실상 불가능하므로, 고수준의 프로그래밍 언어로 프로그램을 작성하면 이를 기계어로 번역해 줄 Translator가 필요하다.

### 프로그래밍 언어

* Procedural Language
  * 절차(how, 어떻게 수행할 지)를 명시한다.
* Object Oriented Language
  * 객체 단위로 코드를 작성한다.
* Functional Language
  * 무엇(what)을 할 지 명시한다.

### Translator

* Complier
  * 고수준 언어로 작성된 프로그램을 기계어 프로그램으로 변환한다.
* Assembler
  * 컴파일러의 일부로, 어셈블리 언어로 작성된 프로그램을 기계어 프로그램으로 변환한다.
* Interpreter
  * 언어를 기계어로 변환하는 즉시 실행한다.
  * 자바의 경우 Java Compiler가 자바 프로그램을 바이트 코드로 변환한다. 바이트 코드는 JVM의 instruction set으로 볼 수 있다. JVM은 이 바이트코드를 적절한  machine instruction으로 변환하는 동시에 바로 실행한다.

## 컴파일러

* Source Program을 target program으로 번역하는 프로그램이다.
* 컴파일러에게는 다음 사항이 요구된다.
  * 정확하게 번역, 코드 최적화 , 빠른 번역 , 컴파일 분리 지원, 에러 메시지의 정확성

### C의 컴파일 과정

* 정의해둔 매크로들을 코드에 넣고, 헤더 파일들을 include하여 확장된 소스 프로그램을 만든다.
* 이를 컴파일러로 컴파일하면 각각 object 파일들이 생성되는데 이 파일들을 relocate하면서 하나의 파일로 합친다.
* 링커에서는 라이브러리, 다른 object 파일들을 묶어 실행 가능한 파일로 만든다.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXe90khgpJb55YkwVfWfKW2_X0Ttc4bvS-yGr2cm8pForLyDuSpKC66M68ErGTDS1-WI9uXJN21sFJUIbrngWZb-zeBx1kBDOpK8nM9YpuFa_RMC-IRQ4Adj-8hc9aSQSYyK3ho1J87DMzcloDwV6zq__ZkMR8pvR4Fi3dY4fhoKHq8B1YRXSQ?key=llHr_iIySL62nsDT1kIdwg" alt="" width="375"><figcaption></figcaption></figure>

### 컴파일 순서

* Lexical(구문), Syntax(문법) 분석을 진행한다.
* Intermediate Code Generator에서는 현재 context에 맞는 의미를 가진 행위인지 확인한다.
* Frontend에서는 머신에 상관없는 표준 Intermediate Code를 반환한다.
* Code Optimizer에서는 중복되는 것을 삭제/축약하여 코드의 양 줄이면서 빠르게 동작하도록 한다.

> Frontend, Backend를 나눈 이유는 코드 최적화와 타겟 코드 생성 부분만 machine에 따라 다르게 작성하여 코드 분석 부분을 재활용하기 위함이다.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdTTo-P3x30NDeuLPY93FkdT6ZJRhKQKWgc9seCzPySQPikOMIkJ8sWjCyuTBKF-Z56jYwdkihRLjB-5seDI6OP1_YlbnN0pV8lvHRnK98lFob2L2KoKR0HdsMKHshGf-Whu5Q_x-7XKS98xcR6IZ4mOCpDR2_Y9wEPhbFfQl68iPkM-_e_3sQ?key=llHr_iIySL62nsDT1kIdwg" alt=""><figcaption></figcaption></figure>

### 레지스터

* 메모리에서 가져온 데이터를 저장하는 CPU 내부 저장소
* 일반적으로 Single cycle에 읽거나 쓸 수 있다.
* MIPS는 32-bit 레지스터를 가지며, 각 레지스터는 32개의 플립플롭 가진다.
* RISC 환경에서 CPU에서 연산을 하기 위해서는 데이터를 레지스터에 올려두어야 한다.
* 명령어는 레지스터를 변경하거나 메모리의 상태를 변경한다.
* 보통 현재 계산을 수행중인 값을 저장하는 데 사용된다.
* 대부분의 현대 프로세서는 메인 메모리에서 데이터를 레지스터로 옮겨와 처리한 후 해당 내용을 다시 메인 메모리에 저장하는 방식을 사용한다.
* CPU내의 메모리 계층 최상위에 위치하며 가장 빠른 속도로 접근 가능한 메모리이다.

<figure><img src="../../.gitbook/assets/image (40) (1).png" alt=""><figcaption><p><a href="https://leechangyo.github.io/cs/2020/05/19/20.-what-are-CPU-Registers/">https://leechangyo.github.io/cs/2020/05/19/20.-what-are-CPU-Registers/</a></p></figcaption></figure>

#### 32bit 컴퓨터가 RAM을 최대 4GB까지 밖에 사용하지 못하는 이유

* 32bit 컴퓨터는 레지스터의 크기를 32bit로 두고 사용하는 장비이다.
* 즉, 컴퓨터가 처리하는 기본 데이터의 크기(한 번에 처리할 수 있는 데이터의 크기)가 32bit이므로 2^32 만큼을 표현할 수 있다.&#x20;
* 1 byte \* 2^32 = 4,294,967,296 bytes 내의 주소밖에 표현을 못하기 때문에 4GB를 넘게 될 경우 해당 영역을 가리키지 못한다.
* 따라서 32 bit 컴퓨터는 최대 4GB의 프로그램만 사용할 수 있다.

### 메모리

* 메모리는 바이트 단위로 입출력이 가능하며, 모든 바이트마다 주소를 가진다.
* 따라서 메모리의 데이터에 접근하려면 주소가 필요하다.
* 명령어, 데이터를 포함한 프로그램을 저장하는 공간이다.

### **MIPS Memory Allocation**

* 0\~7fffffff까지만 있는 이유는 이후 메모리 주소들에는 커널 영역이 사용하는 데이터가 들어가기 때문이다.
* 아래에서 프로그램이 저장되어있는 곳은 virtual memory이다.
* text 영역에는 instruction들이 저장되어 있으며, 나머지 부분에는 데이터들이 저장된다.
* static 영역에는 global variable, static variable들이 저장되어 있다.

> static 영역인 10000000(16진수) 이전에는 항상 0xxxxxxx 값이므로 이를 이진수로 표현하면 맨 앞 4bit는 항상 0이다. 그리고 instruction은 4bytes이고 Memory Alignment 특성 상 항상 4의 배수부터 할당되므로 이진수의 맨 뒷자리 2bit도 항상 0일 것이다. 이진수로 나타내면 `0000(6*4 + 2 bit)00` 가 되기 때문에 총 32bit 데이터를 26bit로만으로도 구분이 가능하다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 데이터 크기

* Word: computation을 위한 기본 데이터 크기를 의미한다. 예를 들어 32bit 프로세서에게 1 Word는 32bit이다. Word와 레지스터의 크기는 항상 같다.
* Double word: Word의 2배 크기를 의미한다.
* Half word: Workd의 1/2배 크기를 의미한다.
* Byte Addressability: 8bit인 Byte 단위에 각각 주소를 가진다.

### Alignment

* 메모리에 저장된 명령어나 데이터를 가리키는 주소가 해당 크기의 배수에 저장되는 것을 의미한다.
* Word 단위(32bits == 4 bytes)를 저장하려면 0, 4, 8, 12, 16, ... 바이트 주소값에 저장될 수 있다.
* Byte 단위(1 bytes)를 저장하려면 0, 1, 2, ... 바이트 주소값에 저장될 수 있다.
* 하드웨어의 복잡성을 줄이고 속도를 빠르게 하기 위함이 목적이다.
* 프로세서는 메모리 접근 단위를 워드 단위로 제한한다.
* 컴파일러는 자동으로 Align되도록 데이터를 저장하며, Misalign 상태가 되면 프로그램을 실행할 수 없다.

## Machine Instruction

### 명령어

* Opcode
  * 수행될 operation을 명시한다.
  * ex) ADD, MULT, LOAD, STORE, JUMP, ...
* Operands
  * Operation의 대상이 되는 데이터를 의미한다.
  * `Source Operands`는 input data를 의미하고, `Destination Operands`는 output data를 의미한다.
  * 예를 들어 Load 명령을 사용할 경우 Source Operands는 메모리, Destination Operands는 레지스터가 된다.
  * 명령어의 길이는 32bit이므로 레지스터에 하나 올리면 공간을 다 차지한다. 따라서 레지스터 번호에 특정 값을 더해 명령어의 주소를 사용한다.

### 명령어 타입

> 모든 명령어는 machine state를 변경한다.

* Arithmetic and logic instructions
  * 피연산자에 대한 계산을 수행하는 명령
  * ex) ADD, MULT, XOR, SHIFT, ..
* Data transfer instructions (memory instructions)
  * 데이터를 메모리 <-> 레지스터로 이동하는 명령
  * 데이터를 계산하기 위해서는 레지스터에 올려야 한다.
  * ex) LOAD, STORE
* Control transfer instructions (branch instructions)
  * 프로그램의 control flow를 변경하는 명령
  * 내부적으로 다음에 읽어올 명령을 가리키는 주소인 PC(Program Counter) 값을 변경해준다.
  * 조건없이 JUMP 하는 goto 같은 명령(**Unconditional JUMP**)이 있고, 특정 조건을 만족해야 브랜치를 JUMP하는 if-else 같은 명령(**conditional branches**)이 있다.
  * ex) JUMP, CALL, RETURN, BEQ

### 명령어 주소 지정 방식

* Register Addressing
  * 5bit로 입력된 레지스터 주소를 사용한다.
  * return register(맨 마지막 31번째 register)의 값으로 점프한다. 항상 주소가 바뀔 수 있으므로 indirect branch이다.
  * `jr $ra` 명령은 $ra 주소로 이동시킨다. $ra는 특정 함수가 끝나고 이동할 메모리 주소를 저장해두는 register이다.
* Base Addressing
  * base register와 상수인 offset을 더해 주소를 지정한다.
* Immediate Addressing
  * 상수를 사용하여 연산하도록 한다.
  * `add $t1, $t2, 3` 명령은 t1 register의 값과 상수인 3을 더해 t2 register에 값을 저장하도록 한다.
* PC-relative Addressing
  * PC에 상수를 더하여 얻은 위치로 PC를 이동시킨다.
* Psuedo-direct Addressing
  * full 32 bit 주소를 사용해 direct로 jump하는 것은 아니지만 이와 유사한 명령이다.
  * 입력된 26bit의 명령어 주소로 바로 이동시킨다.
  * `j L1` 명령은 L1라는 주소로 이동시킨다.

### MIPS 명령어 포맷

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXebpMr9LtBw0tT_0n9hYtzLvQdSdQacWY0Rs9TABi83H-FNmTpNHSdg0AQduMLTjdeJuH8hnS21MNFr9gTTd7htD2dc6FjMUN9E1XgRs_tazchmEVjVKWw4afaYhb3otcB5eqYFslZ-1WCXzWOXXZSX03PRHU-iDDTSaNQPm3rmvOc-K2nV6pc?key=llHr_iIySL62nsDT1kIdwg" alt="" width="375"><figcaption></figcaption></figure>

* 모든 명령어는 32 bit 크기를 가진다.
* CPU 내부에는 32개의 Register가 존재하므로 모든 Register를 5bit(2^5 = 32)로 표현 가능하다.

#### R-type

* 연산 명령어의 타입이다.
* `op`: Opcode, 명령어의 기본 동작
* `rs`: 첫번째 source register
* `rt`: 두번째 source register
* `rd`: destination register
* `shamt`: shift amount, 어떤 register를 몇 bit shift할 지 명시하는 constant 값이다.&#x20;
* `funct`: function code, op코드에서 추가적으로 필요한 것들 명시한다. op를 0으로 두고 funct 값을 조정해 수행될 명령을 선택할 수 있다.
* 아래를 보면 add 명령어를 사용할 때 10진수 값, 2진수 값을 알 수 있다. 이 때 2진수를 4자리씩 끊어 16진수로 표현할 수 있다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### I-type

* 데이터 이동, 조건부 branch, immediate 명령어 타입이다.
* 메모리는 보통 32bit 주소로 명시하는데, 이를 다 명시하면 operation의 길이가 32 bit를 넘어가게 된다. 따라서 데이터 주소를 그대로 사용하는 대신 base register에 offset을 더한 값을 주소로 사용한다.
* `rs`: base register를 지정한다.
* `rt`: register operand, STORE할 때에는 destination register가 되고, LOAD할 때에는 source register가 된다.
* `address(offset)`: 16bit 상수가 위치한다. 값은 +/-(2^15)가 될 수 있으며, 프로그래밍 시 상수를 데이터 영역에 저장하는 대신 이곳에 저장하여 메모리 사용량, 하드웨어 동작을 절감할 수 있다.

#### J-type

* jump 명령어 타입이다.
* `address`: 목적지를 나타내는 명령어를 지정한다. 앞서 MIPS Memory Allocation에서 설명했듯 명령어는 실제로 32bit이지만, 공통된 부분을 생략하여 26bit로도 표현할 수 있다.
