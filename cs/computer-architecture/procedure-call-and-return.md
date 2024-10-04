# Procedure Call & Return

### 프로시저 콜 & 리턴

* procedure caller는 callee가 접근 가능한 argument register들에 함수의 파라미터들을 저장한다.
* callee 주소로 `jump and link`하여 callee 명령어를 수행하고 수행이 완료되면 돌아올 주소값을 $ra에 저장한다. 이 때 jal 명령은 $ra에 관련된 값을 덮어쓰므로, jal 명령어를 호출하기 전 $ra의 값을 메모리 등 다른 곳에 저장해야 한다.
* 이 때 callee는 stack frame을 할당받아 로컬 변수를 저장한다.

### stack

* argument register, return address register의 경우 프로시저 내에서 또다른 프로시저, 함수를 콜 할 때마다 덮어쓰이기 때문에 **stack**에 저장해두고 하위 작업이 완료되면 다시 register에 불러와서 사용한다.
* stack은 높은 주소값에서부터 낮은 주소값으로 자라난다.
* stack pointer: 가장 최근에 스택으로 할당된 주소값에 대한 포인터
* frame pointer: top stack frame의 맨 아래(가장 높은) 주소값에 대한 포인터

### MIPS Register Convention

* MIPS에서는 각 Register의 역할을 정의해두고 있다.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

### factorial 예제

* factorial을 구하는 간단한 c 코드를 어셈블리어를 사용하여 직접 구현할 수 있다.

```c
int fact (int n) {
    if (n < 1) 
        return (1);
    else
        return (n * fact(n1));
}
```

```
fact:
    addi $sp, $sp, -8 // 스택 포인터를 이동시켜 스택에 기존 레지스터 값을 저장해둔다.
    sw $ra, 4($sp)
    sw $a0 0($sp)
    slti $t0, $a0, 1 // 2보다 작을 경우 $t0 에 1을 대입한다
    beq $t0, $zero, L1
    addi $v0, $zero, 1 // $v0이 최종 결과가 될 레지스터이며 해당 값에 1을 넣는다.
    jr $ra
L1:
    addi $sp, $sp,8
    sw $ra, 4($sp)
    sw $a0, 0($sp)
    addi $a0, $a0,1
    jal fact
    lw $a0, 0($sp) // 스택 데이터를 레지스터에 복원한다.
    lw $ra, 4($sp)
    addi $sp, $sp, 8
    mul $v0, $a0, $v0 // 최종 결과 값에 현재 $a0을 곱한다. 여기가 재귀적으로 호출되면서 팩토리얼 결과가 계산된다.
    jr $ra
```
