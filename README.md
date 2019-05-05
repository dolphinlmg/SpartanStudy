# SpartanStudy
This is assignment repository of Spartan study.
PDF and Markdown documents are in [SpartanStudy/pwnable/](/pwnable/)

## First Week
1. Overthewire Bandit All Clear
2. Lazenca.net TechNote 요약
    - Exploit Tech -> Shellcode -> The Basics Technic of Shellcode
    - Exploit Tech -> Return to Shellcode
3. 달고나 문서 27페이지까지 요약

Assignment files are in [here](/pwnable/week1/)

## Second Week
1. WARGAME lv5 까지 모두 풀기.
2. 푼 문제는 꼭 Write-Up 작성.
3. Wargame VMware 이미지 : *
4. lv0 암호는 lv0

Write-Ups are in [here](/pwnable/week2/Write-Up/)

## Third Week
1. Wargame 다 풀고 롸업 작성.
2. Lazenca ROP 읽고 정리해오기.

※Tip
1. 현재 워게임 바이너리에 버그가 있어 패치가 필요합니다. 패치 방법은 아래와 같습니다.
- lv0 로 로그인
- sudo bash < (wget -qO- https://github.com/junheah/sparta-patch/raw/master/patch.sh)
2. 워게임 문제를 풀기위해서는 Lazenca->TechNote->Exploit tech -> 03~05문서와 달고나 문서가 도움이 될겁니다.
3. 각 문제들 힌트를 드리자면 다음과 같습니다.
LEVEL1 simple bof
LEVEL2 small buffer
LEVEL3 small buffer + stdin
LEVEL4 egghunter
LEVEL5 egghunter + bufferhunter
LEVEL6 check length of argv[1] + egghunter + bufferhunter
LEVEL7 check argv[0]
LEVEL8 check argc
LEVEL9 check 0xbfff
LEVEL10 argv hunter
LEVEL11 stack destroyer
LEVEL12 sfp 
LEVEL13 RTL1
LEVEL14 RTL2, only execve
LEVEL15 no stack, no RTL
LEVEL16 fake ebp
LEVEL17 function calls
LEVEL18 plt
LEVEL19 fgets + destroyers

Write-Ups are in [here](/pwnable/week3/Write-Up/)
