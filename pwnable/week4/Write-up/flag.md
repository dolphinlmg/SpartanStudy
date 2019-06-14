# flag
## flag

이 파일은 `gdb`로 디버깅을 해봐도 아무 함수도 나오지 않는다.

```
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, stripped
```

```
strings flag | more
UPX!
@/x8
gX lw_
H/\_@
	Kl$
H9\$(t
[]]y
nIV,Uh
AWAVAUATS
uSL9
>t		.
[A\AA;h
]A^A_*U
A4tV
bl@(y
V</uI
	kzPb
o)Kf
	@TpM"
h'T@r
PHES}<w
\Xt@
Ev%~
-)";o
"+u_K
AG8W$
=P&,
```

스트링 검색을 해보니 맨 처음 `UPX`라는문자열을 발견했다. 

```
C:\Users\Mac\Downloads\upx394w>upx -1 flag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94w       Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
upx: flag: AlreadyPackedException

Packed 0 files.

C:\Users\Mac\Downloads\upx394w>upx -d flag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94w       Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    883745 <-    335288   37.94%   linux/amd64   flag

Unpacked 1 file.

```
언패킹을 하고 다시 `gdb`를 돌렸다.

`malloc()`으로 플래그를 줄테니 가져가라는 말이 나온다.

```asm
=> 0x401184 <main+32>:	mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
```

```
gdb-peda$ x/wx 0x6c2070
0x6c2070 <flag>:	0x00496628
gdb-peda$ x/s 0x00496628
0x496628:	"UPX...? sounds like a delivery service :)"
gdb-peda$ 
```