# Collision

## col.c

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

check_password를 보면 `argv[1]`을 `char*`로 받고, `int*`로 변환해 5번 더한다. `argv[1]`에 리틀 엔디안으로 더해서 `0x21DD09EC`이 되는 값을 넣어주면 된다.

## exploit.py

```python
from pwn import *

context.log_level= 'debug'

shell = ssh('col', 'pwnable.kr', 2222, 'guest')

hashcode = 0x21dd09ec

p = shell.run('./col ' + p32((hashcode - hashcode % 5) / 5) * 4 + p32((hashcode - hashcode % 5) / 5 + hashcode % 5))

p.recv()
```

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;./col \xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xcc\xce\xc5\x06': Done
[DEBUG] Received 0x34 bytes:
    'daddy! I just managed to create a hash collision :)\n'
[*] Closed SSH channel with pwnable.kr

```