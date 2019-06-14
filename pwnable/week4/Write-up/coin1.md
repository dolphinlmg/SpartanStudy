# coin1

## coin1

```

	---------------------------------------------------
	-              Shall we play a game?              -
	---------------------------------------------------
	
	You have given some gold coins in your hand
	however, there is one counterfeit coin among them
	counterfeit coin looks exactly same as real coin
	however, its weight is different from real one
	real coin weighs 10, counterfeit coin weighes 9
	help me to find the counterfeit coin with a scale
	if you find 100 counterfeit coins, you will get reward :)
	FYI, you have 60 seconds.
	
	- How to play - 
	1. you get a number of coins (N) and number of chances (C)
	2. then you specify a set of index numbers of coins to be weighed
	3. you get the weight information
	4. 2~3 repeats C time, then you give the answer
	
	- Example -
	[Server] N=4 C=2 	# find counterfeit among 4 coins with 2 trial
	[Client] 0 1 		# weigh first and second coin
	[Server] 20			# scale result : 20
	[Client] 3			# weigh fourth coin
	[Server] 10			# scale result : 10
	[Client] 2 			# counterfeit coin is third!
	[Server] Correct!

	- Ready? starting in 3 sec... -
```

`nc pwnable.kr 9007`로 접속하면 위와 같은 안내문이 뜬다. 전체 코인의 수와 시도 횟수를 주고, 그 안에 정답을 맞춰야 한다. 

이 문제는 탐색문제이며 최근에 이산수학 시간에 배운 이진탐색을 이용하기로 했다.

## exploit.py

```python
from pwn import *
from time import sleep

#context.log_level = 'debug'

count = 0

# serch if counterfeit coin is in start~end
# if it is in the range, return True
def searchCoin(start, end, server):
    tmp = ''
    for i in range(start, end+1):
        tmp += str(i) + ' '
    tmp = tmp[:-1]
    server.sendline(tmp)
    data = server.recvuntil('\n')
    if int(data) == (end - start + 1) * 10:
        return False        # Not found
    return True

def binarySearch(coin, server):
    global count
    s = 0
    e = coin
    while True:
        m = (s + e)/2
        if searchCoin(s, m, server): # if found
            e = m
        else:                        # if not found
            s = m + 1
        count -= 1
        if s == e:
            return s


r = remote('pwnable.kr', 9007)
r.recvuntil('3 sec... -\n\t\n')
sleep(3)

for i in range(100):
    
    print i
    prob = r.recvuntil('\n')

    num = int(prob.split('N=')[1].split(' ')[0])
    count = int(prob.split('C=')[1].split('\n')[0])

    #print num, count

    try:
        index = binarySearch(num, r)
        
        #print 'rest count:', count
        #print 'counterfeit coin index:', str(index)
        while count >= 0:
            r.sendline(str(index))
            count -= 1
            if 'Correct!' in r.recvuntil('\n'):
                break
    except:
        pass
r.interactive()
```

화면 출력이 느려서 주석처리 해뒀다.

```bash
[+] Opening connection to pwnable.kr on port 9007: Done
0
1
2
3
4
5
~~~~~~
94
95
96
97
98
99
[*] Switching to interactive mode
Congrats! get your flag
b1NaRy_S34rch1nG_1s_3asy_p3asy
$ 
[*] Interrupted
[*] Closed connection to pwnable.kr port 9007
```