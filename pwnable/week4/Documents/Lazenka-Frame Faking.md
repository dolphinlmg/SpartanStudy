# Frame faking (Fake ebp)

## Frame faking

frame faking이란 스택 프레임 포인터를 조작해 프로그램의 실행 흐름을 변경하는 것이다.

- ret역역이나 sfp영역까지 덮어 쓸 수 있을 때도 사용 가능하다.

## leave & ret

Instruction | Explanation
---- | ----
leave | mov esp, ebp; pop ebp
ret | pop eip; jmp eip   

`leave & ret` 의 특성상 frame faking은 main함수에서는 불가능한 기법이다. `leave`를 통해 그 함수를 호출한 함수의 ebp를 조작 할 수 있기 때문이다. 

## Stack Frame

```c
void vuln(){
    char buf[0x20];
    read(0, buf, 0x24);
}

void main(){
    vuln();
}
```

위와 같은 코드가 있다고 가정해보자. 위 코드를 어셈블리어로 고치면 대략 다음과 같을 것이다.

```asm
    *main

    push ebp
    mov ebp, esp
    call vuln
    leave
    ret

    *vuln

    push ebp
    mov ebp, esp
    sub esp, 0x20
    push 0
    lea eax, [ebp - 0x20]
    push dword ptr [eax]
    push 0x24
    call read@plt
    add esp, 0xc
    leave
    ret
```

그렇다면, `vuln`의 `leave`가 실행되기 직전의 스택은 아래와 같을 것이다. 

Stack | Explanation
--- | ----
**envp |
**argv |
argc | 
ret of main |
sfp of main |
ret of vuln |
sfp of vuln |   &larr; ebp
buf[0x20] |     &larr; esp

이때 `vuln`의 `sfp`의 값을 `buf`를 가리키게 한 후 `main`함수로 돌아갔을 때의 상태를 보자.


Stack | Explanation
--- | ----
**envp |
**argv |
argc | 
ret of main |
sfp of main |   &larr; esp
ret of vuln |
sfp of vuln |
buf[0x20] |     &larr; ebp

위와 같이 `ebp`는 `buf`를 가리키고 있고, `esp`는 정상적으로 돌아왔다. 이 상태에서 `main`의 `leave`를 수행하게 되면 다음과 같은 현상이 벌어진다.

Stack | Explanation
--- | ----
**envp |
**argv |
argc | 
ret of main |
sfp of main |
ret of vuln |
sfp of vuln |
buf[0x1c] |     &larr; esp
buf[0x4] | 

이렇게 되면 `ebp`는 `buf`의 첫 dword에 담긴 주소로 변하고, `esp`는 `buf+4`를 가리키게 된다. 결국 덮어버린 `vuln`의 `sfp+4`바이트에 있는 dword의 값을 `eip`에 넣고 `ret`을 진행하게 되어 프로그램의 흐름을 바꿀 수 있다.