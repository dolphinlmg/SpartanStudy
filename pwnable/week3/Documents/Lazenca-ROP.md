# ROP - Return Oriented Programming

## What is ROP?
ROP는 NX-Bit 같은 보호기법이 걸려있을 때 사용할 수 있는 공격 기법 이다.

- RTL + Gadgets

이 기법을 사용하기 위해서는 RTL이 가능해야 하며 해당 프로그램이 사용하는 메모리 안에 존재하는 Gadget이 필요하다.

일반적으로 RTL만으로는 연속적인 함수를 호출하기 어려운데, 그 이유는 스택으로 전달하는 함수의 인자를 전달하기 어렵기 때문이다. 무작정 인수를 스택에 쌓는다고 해도, ESP의 값이 고정되어 있으므로 호출된 함수는 그 인수를 사용할 수 없다. 이 ESP의 값을 움직이기 위해 사용하는 것이 Gadget이다.

### What is RTL?

간단하게 RTL에 대해 정리하고 넘어가도록 하자. RTL이란 Return To Libc의 약자로, NX-Bit가 걸려있을 때 사용가능하다. 이는 `RET` 을 우리가 메모리 어딘가에 적재한 쉘코드의 시작 주소로 돌리는 것이 아닌, 바이너리가 실행 될 때 함께 적재되는 `Libc`의 주소로 돌리는 것이다.

이렇게 `Libc`로 `RET`을 덮어 쓰게 되면 덮어 씌워진 `RET`을 통해 `Libc`의 함수로 이동할 것이다. 이 때 우리는 해당 함수에게 인자를 넘겨 줘야 하는데, 이는 다음과 같은 규약에 의해 전달된다.

#### Cdecl
- 함수의 인자를 스택에 저장하며, 오른쪽 인수부터 왼쪽 순서대로 저장한다.
- 함수의 반환은 EAX
- 호출한 함수가 스택을 정리

따라서 다음과 같이 스택을 구성하면 된다.

Address | Value | Explanation
--------|--------|--------
High | 3 | Argv3
-| 2 | Argv2
-| 1 | Argv1
-| &exit | RET of System()  
Low | &System | RET


## Gadget?
가젯은 `pop ~ ret`의 형태를 띄고 있는 코드 조각을 의미한다. 호출하려는 함수가 세 개의 인수를 받는다면 `pop; pop; pop; ret`을, 하나를 받는다면 `pop; ret`을  사용하게 된다.

이 가젯의 역할은 스택 포인터를 증가시키는 것이다. 그러므로 x86 바이너리에서는 `pop`의 피연산자는 중요하지 않다. 

이 Gadget을 이용하여 여러 함수를 호출하는 방법은 다음과 같다.

먼저 BOF가 발생하는 함수의 `RET`을 호출할 첫번째 함수의 주소로 덮어 쓴다. 그 다음 4바이트를 호출할 함수의 인자 개수만큼 `pop`을 가진 Gadget의 주소로 덮어쓴다. 그 후 인자를 순차적으로 넣은 후, 그다음 호출할 함수의 주소를 덮어 쓴다.

정리하면 아래와 같이 스택을 구성하면 된다.

Address | Value | Explanation
---- | ---- | ----
High | Argv1 | Argv for second Func
-| &exit | Return address of second func
-| &second | Address of second func
-| Argv3 | Argv for first Func
-| Argv2 | 
-| Argv1 | 
-| Address of Gadget(pop;pop;pop;ret) | Address of Gadget 
Low | &first | Address of first func

## PLT & GOT

- plt (Procedure link table)에는 동적 링커가 공유 라이브러리의 함수를 호출하기 위한 코드가 저장되어 있다.
- got(Global offset table)에는 동적 링커에 의해 공유 라이브러리에서 호출할 함수의 주소가 저장된다.

### How dose it works?

gdb로 디버깅을 하다 보면 다음과 같은 코드를 볼 수 있다.

```asm
call   0x8048320 <write@plt>
```
이 `write@plt`를 찾아보면 다음과 같이 Data Segment의 `0x804a014(write@got.plt)`로 jmp하는 것을 볼 수 있다.
`write`가 한번도 호출되지 않았을 때 `write@got.plt`의 값은 `write@plt + 6`을 가리키고 있다. `write@plt + 6`는 `libc`에 존재하는 `write`함수의 주소를 `write@got.plt`에 기록하는 코드이다.
``` asm 
gdb-peda$ disass 0x8048320
Dump of assembler code for function write@plt:
   0x08048320 <+0>:	jmp    DWORD PTR ds:0x804a014
   0x08048326 <+6>:	push   0x10
   0x0804832b <+11>:	jmp    0x80482f0
End of assembler dump.
gdb-peda$ x/wx 0x804a014
0x804a014:	0x08048326
gdb-peda$ disass 0x08048326
Dump of assembler code for function write@plt:
   0x08048320 <+0>:	jmp    DWORD PTR ds:0x804a014
   0x08048326 <+6>:	push   0x10
   0x0804832b <+11>:	jmp    0x80482f0
End of assembler dump.
gdb-peda$ 
```
이후 `write`가 호출 되면 `write@got.plt`의 값은 `Libc`에 있는 `write`함수의 주소로 변경된다.
```asm
gdb-peda$ disass 0x8048320
Dump of assembler code for function write@plt:
   0x08048320 <+0>:	jmp    DWORD PTR ds:0x804a014
   0x08048326 <+6>:	push   0x10
   0x0804832b <+11>:	jmp    0x80482f0
End of assembler dump.
gdb-peda$ x/wx 0x804a014
0x804a014:	0xf7ed9b70
gdb-peda$ disass 0xf7ed9b70
Dump of assembler code for function write:
   0xf7ed9b70 <+0>:	cmp    DWORD PTR gs:0xc,0x0
   0xf7ed9b78 <+8>:	jne    0xf7ed9ba0 <write+48>
   0xf7ed9b7a <+0>:	push   ebx
   0xf7ed9b7b <+1>:	mov    edx,DWORD PTR [esp+0x10]
   0xf7ed9b7f <+5>:	mov    ecx,DWORD PTR [esp+0xc]
   0xf7ed9b83 <+9>:	mov    ebx,DWORD PTR [esp+0x8]
   0xf7ed9b87 <+13>:	mov    eax,0x4
   0xf7ed9b8c <+18>:	call   DWORD PTR gs:0x10
   0xf7ed9b93 <+25>:	pop    ebx
   0xf7ed9b94 <+26>:	cmp    eax,0xfffff001
   0xf7ed9b99 <+31>:	jae    0xf7e1c730 <__syscall_error>
   0xf7ed9b9f <+37>:	ret    
   0xf7ed9ba0 <+48>:	call   0xf7ef7ba0 <__libc_enable_asynccancel>
   0xf7ed9ba5 <+53>:	push   eax
   0xf7ed9ba6 <+54>:	push   ebx
   0xf7ed9ba7 <+55>:	mov    edx,DWORD PTR [esp+0x14]
   0xf7ed9bab <+59>:	mov    ecx,DWORD PTR [esp+0x10]
   0xf7ed9baf <+63>:	mov    ebx,DWORD PTR [esp+0xc]
   0xf7ed9bb3 <+67>:	mov    eax,0x4
   0xf7ed9bb8 <+72>:	call   DWORD PTR gs:0x10
   0xf7ed9bbf <+79>:	pop    ebx
   0xf7ed9bc0 <+80>:	xchg   DWORD PTR [esp],eax
   0xf7ed9bc3 <+83>:	call   0xf7ef7c10 <__libc_disable_asynccancel>
   0xf7ed9bc8 <+88>:	pop    eax
   0xf7ed9bc9 <+89>:	cmp    eax,0xfffff001
   0xf7ed9bce <+94>:	jae    0xf7e1c730 <__syscall_error>
   0xf7ed9bd4 <+100>:	ret    
End of assembler dump.
gdb-peda$ 
```
ROP Exploit 도중 `write`함수를 통해 `got`에 있는 값을 출력하도록 해 `libc` 에 존재하는 함수의 주소를 가져오면 ASLR이 걸려있어도 해당 함수의 주소를 찾을 수 있다.