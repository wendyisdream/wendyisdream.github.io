---
title: uftrace record data unit
categories: [Blogging, Uftrace]
tags: [study]
---


타켓 프로그램의 실행 과정 중 uftrace libmcount.so는 함수의 entry/exit, event 발생 시 정보를 hook하고 uftrace는 데이터를 {tid}.dat에 기록한다.  
argument/return value나 file name/line number등의 추가 debug 정보가 요청되면 .dbg에 관련 정보가 저장된다.  

Test code를 대상으로 uftrace가 저장하는 모습을 살펴보자~  

- Test code  
	``` C
	#include <stdio.h>
	#include <stdlib.h>

	int fib(int n)
	{
			if (n <= 2)
					return 1;
			return fib(n - 1) + fib(n - 2);
	}

	int main(int argc, char *argv[])
	{
			int n = 8;

			if (argc > 1)
					n = atoi(argv[1]);

			fib(n);
			return 0;
	}
	```

- 컴파일  
```gcc -pg -g tests/s-fibonacci.c
```
로 컴파일 후에 uftrace에 argument/return value를 저장하는 옵션으로 record를 해보자~
- uftrace record  
```uftrace record -a  a.out 3
```  

- uftrace replay로 실행과정을 확인해보자~
```
uftrace replay -a 
# DURATION     TID     FUNCTION
   0.354 us [ 30729] | __monstartup();
   0.199 us [ 30729] | __cxa_atexit();
            [ 30729] | main(2, 0x7fff586c0b08) {
  52.400 us [ 30729] |   atoi("3") = 3;
            [ 30729] |   fib(3) {
   0.094 us [ 30729] |     fib(2) = 1;
   0.055 us [ 30729] |     fib(1) = 1;
   0.743 us [ 30729] |   } = 2; /* fib */
  54.631 us [ 30729] | } = 0; /* main */
```

위의 결과를 위한 uftrace에서 저장한 데이터의 단위와 포멧을 확인해보자.  
uftrace.data의 {tid}.dat을 보면 다음과 같다.


![이미지](https://wendyisdream.github.io/assets/img/uftrace_dat.png "uftrace .dat example")

각 entry/exit에서 저장되는 단위는 다음과 같이 16byte 정보가 기본이 된다.
```C
struct uftrace_record {
	uint64_t time;
	uint64_t type : 2;
	uint64_t more : 1;
	uint64_t magic : 3;
	uint64_t depth : 10;
	uint64_t addr : 48; /* child ip or uftrace_event_id */
};
```
__monstartup와 같이 argument/return value등 추가 저장이 없는 경우(more==0)에는 16byte로 저장되고,  
fib와 같이 argument/return value가 있는 경우(more==1)에는 16byte에 argument(entry), return value(exit)값이 이어서 저장된다.


각 함수의 argument/return value의 형식은 .dbg에 저장된다.
s-fibonacci.c의 a.out 실행파일에 .dbg 내용은 다음과 같다.

- a.out.dbg
```
# path name: /home/jia/workspace/uftrace/a.out
# build-id: 3765f5db162cc251641fd169960c21ca15d2a197
F: 77a fib
L: 4 tests/s-fibonacci.c
A: @arg1
R: @retval
F: 7be main
L: 11 tests/s-fibonacci.c
A: @arg1,arg2/p
R: @retval
```