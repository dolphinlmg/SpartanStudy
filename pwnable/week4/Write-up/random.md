# random

##random.c

```c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

`random`값을 취하는데, `srand()`로 seed를 초기화 하지 않아 같은 값만 나온다.

gdb로 열어 `random`값을 확인했다.

```asm
   0x00000000004005f4 <+0>:	push   rbp
   0x00000000004005f5 <+1>:	mov    rbp,rsp
   0x00000000004005f8 <+4>:	sub    rsp,0x10
   0x00000000004005fc <+8>:	mov    eax,0x0
   0x0000000000400601 <+13>:	call   0x400500 <rand@plt>
   0x0000000000400606 <+18>:	mov    DWORD PTR [rbp-0x4],eax
   0x0000000000400609 <+21>:	mov    DWORD PTR [rbp-0x8],0x0
   0x0000000000400610 <+28>:	mov    eax,0x400760
   0x0000000000400615 <+33>:	lea    rdx,[rbp-0x8]
   0x0000000000400619 <+37>:	mov    rsi,rdx
   0x000000000040061c <+40>:	mov    rdi,rax
   0x000000000040061f <+43>:	mov    eax,0x0
   0x0000000000400624 <+48>:	call   0x4004f0 <__isoc99_scanf@plt>
   0x0000000000400629 <+53>:	mov    eax,DWORD PTR [rbp-0x8]
   0x000000000040062c <+56>:	xor    eax,DWORD PTR [rbp-0x4]
   0x000000000040062f <+59>:	cmp    eax,0xdeadbeef
   0x0000000000400634 <+64>:	jne    0x400656 <main+98>
   0x0000000000400636 <+66>:	mov    edi,0x400763
   0x000000000040063b <+71>:	call   0x4004c0 <puts@plt>
   0x0000000000400640 <+76>:	mov    edi,0x400769
   0x0000000000400645 <+81>:	mov    eax,0x0
   0x000000000040064a <+86>:	call   0x4004d0 <system@plt>
---Type <return> to continue, or q <return> to quit---
   0x000000000040064f <+91>:	mov    eax,0x0
   0x0000000000400654 <+96>:	jmp    0x400665 <main+113>
   0x0000000000400656 <+98>:	mov    edi,0x400778
   0x000000000040065b <+103>:	call   0x4004c0 <puts@plt>
   0x0000000000400660 <+108>:	mov    eax,0x0
   0x0000000000400665 <+113>:	leave  
   0x0000000000400666 <+114>:	ret
```

`main+18`에 브레이크포인트를 걸고 `rax`값을 확인했다.

```bash
(gdb) b *main+18
Breakpoint 1 at 0x400606
(gdb) r
Starting program: /home/random/random 

Breakpoint 1, 0x0000000000400606 in main ()
(gdb) p/x $rax
$1 = 0x6b8b4567
(gdb)
```

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

random = 0x6b8b4567
key = 0xdeadbeef

shell = ssh('random', 'pwnable.kr', 2222, 'guest')

r = shell.run('./random')

r.sendline(str(random ^ key))

print(r.recv())
```

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;./random': Done
[DEBUG] Sent 0xb bytes:
    '3039230856\n'
[DEBUG] Received 0x37 bytes:
    'Good!\n'
    'Mommy, I thought libc random is unpredictable...\n'
Good!
Mommy, I thought libc random is unpredictable...

[*] Closed SSH channel with pwnable.kr
```