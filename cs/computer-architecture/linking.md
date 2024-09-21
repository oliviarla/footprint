# Linking

## 링커

* 하나의 거대한 프로그램으로 구성하면 일부를 수정하더라도 전체를 다시 컴파일해야 하고 자주 사용하는 일부 함수만 가져와서 사용하려고 할 때 전체를 가져와야 하므로 비효율적이고 모듈 별로 관리가 불가능해 불편하다.
* 따라서 자주 사용되는 함수 등을 각각의 파일로 분리한다. 이 때 각 파일들을 컴파일하면 relocatable object file들이 된다.
* relocatable object file은 외부에 정의된 함수, 변수에 대한 참조를 사용할 수 없는 상태이므로 실행 불가능하다.
* Linker는 이러한 object file들을 하나로 묶어 실행 가능한 object 파일을 만들어내는 linking을 수행한다.
* static linker
  * 컴파일 타임에 linking을 수행한다.
  *   다음 두 가지 핵심 작업을 처리한다.

      * **Symbol resolution**: 함수나 변수같은 symbol이 어떤 파일에 정의된 것인지 찾아 연결한다. 즉, definition과 reference를 연결해준다.
      * **Relocation**: 각각의 object 파일에서 코드는 코드끼리, 데이터는 데이터끼리 묶어 단일 text segment, data segment 등으로 만들어낸다. segment에 가상 메모리 주소를 할당하고

      <figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>
  * dynamic linker를 사용하지 않고 static linker만 사용할 경우 컴파일 시점에 프로그램이 실행하기 위한 모든 코드, 데이터가 들어간다.
* dynamic linker
  * 실행 중이거나 로딩될 때 linking을 수행한다.

### 컴파일러 드라이버

* translation과 linking 과정을 수행해주는 도구이다.
* preprocessor, compiler, assembler, linker를 사용한다.
* gcc는 GNU C Compiler의 약자로, c언어의 컴파일러이다.
  * \-O2 는 최적화 수준을 지정하는 옵션이다. 성능을 개선하기 위해 코드의 불필요한 부분을 제거하거나 반복을 줄이는 등의 최적화가 이뤄진다.
  * \-g는 디버깅 정보를 포함하도록 하는 옵션이다. 컴파일된 바이너리 파일에 디버깅을 위한 추가 정보가 포함되며, `gdb`와 같은 디버거를 통해 소스 코드 수준에서 디버깅이 가능하다.
  * \-o는 object 파일의 이름을 지정하는 옵션이다.
  * 맨 마지막에는 컴파일할 소스 파일을 지정해야 한다.

```
gcc -O2 -g -o p main.c swap.c
```

## Object File

### 종류

* 컴파일러가 생성해내는 object 파일에는 다음과 같은 종류가 있다.
* Relocatable object file
  * 소스 코드를 컴파일하여 만들어진 파일
* Executable object file
  * 링커에 의해 여러 object 파일들이 실행 가능하도록 묶여 만들어진 파일
* Shared object file
  * 로드 혹은 런타임에 연결될 수 있는 형태로 만들어진 파일

### 포맷

#### ELF(Executable and Linkable Format)

* 현재 UNIX에서 object file에 대한 표준 바이너리 포맷이다.
* 모든 object file 종류에 대해 ELF 포맷이 적용된다.
*   아래와 같이 구성 요소가 나뉜다.

    <figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

    * ELF header
      * 파일을 실행할 때, 커널이나 로더가 프로그램을 어떻게 메모리에 로드하고 실행할지 결정하는 데 사용된다.
      * object file의 종류, 섹션 헤더의 위치(offset)
      * Segment header table
        * 페이지 크기, 메모리 세그먼트의 주소, 페이지의 access permission 속성(readable, writable, ..) 이 저장된다.
      * Section header table
        * locators와 각 section들의 크기를 저장한다.
        * 각 섹션은 고정된 크기를 갖는다.
      * .text section
        * 함수 등 직접 작성한 코드를 저장한다.
      * .data section
        * 초기화된 전역 변수
      * .bss section
        * 초기화 되지 않은 전역 변수
        * 초기화되지 않은 전역 변수에는 placeholder를 할당하여 object file에 임의의 초기 데이터를 저장하는 대신 로드될 때 초기화 코드에 의해 초기 데이터가 할당되도록 한다.
        * 예를 들어 1000개의 원소를 갖는 int 배열(`int arr[1000]`)을 초기화하지 않은 전역 변수로 선언했을 때, `.bss` 파일에는 placeholder만 저장되고, c에서 로드될 때 0이 1000개 들어간 초기 배열이 할당된다.
      * .symtab section
        * symbol table로, 함수와 전역 변수에 대한 인덱스가 저장된다.
      * .debug section
        * 디버깅 시 필요한 추가적인 정보를 저장한다.
      * .line section
        * 디버깅 시에 사용되는 정보를 저장하며, text에 있는 명령어와 소스 코드의 line과 매칭하는 정보를 저장한다.
      * .strtab section
        * .symtab, .debug에 속한 symbol들의 이름을 저장한다.

### Life & Scope

* object의 **life**는 메모리에 존재하는 지에 따라 달라지고, **scope**은 코드의 어떤 영역에서 접근 가능한 지에 따라 달라진다.
* scope의 종류는 프로그램 전체에서 접근 가능한 global scope, 함수 내에서 접근 가능한 local scope, 파일내에서 접근 가능한 file scope가 있다.
* 따라서 object는 visible할 수 없지만 live한 상태일 수 있다.
* **local variable**은 함수 내에 선언된 변수이므로 함수 호출 시 생성되고 결과 반환 시 사라진다.
* **static local variable**은 함수 내에 선언되어 있고 static 공간에 할당되어 계속 메모리에 저장되어 있다. 따라서 항상 live하다.
* **global variable**은 모든 object file에서 접근 가능하고 프로그램 시작 시 생성되고 프로그램이 종료 시 사라진다.
* **static variable**은 함수 외부에 static으로 선언되어 static 공간에 할당되고 파일 내부에서만 접근이 가능하다.

### Symbol

* 변수나 함수에 대한 참조(reference)를 의미하며, linker에서 사용되는 개념이다.
* c 파일들에 존재하는 함수 이름, 전역 변수, 정적 변수 등이 컴파일되면 symbol이 된다.
* global symbol
  * 특정 모듈에 정의되어 있고 외부 모듈에서 접근이 가능한 symbol
  * 함수 이름, 전역 변수는 여기에 해당한다.
  * global symbol을 선언된 파일에서 접근하면 local symbol이고, 외부 파일에서 접근할 경우 external symbol이라고 부른다.

### Symbol table

* 컴파일러가 만들어내는 object들에 대한 정보이다.&#x20;
* object의 이름, 크기, 타입, 스코프 등의 정보가 저장된다.
* 프로그램 symbol은 strong, weak로 나뉜다.&#x20;
  * **strong symbol**에는 모든 프로시져 함수 이름, 초기값을 갖는 전역 변수가 해당되며, 이름이 중복되면 안된다.
  * **weak symbol**은 초기값을 갖지 않는 전역 변수가 해당된다. 이름이 중복될 수 있으며, strong symbol의 이름과 중복될 경우 strong symbol이 우선시되고 weak symbol 끼리 이름이 중복되면 링커 임의대로 하나만 링킹된다.











