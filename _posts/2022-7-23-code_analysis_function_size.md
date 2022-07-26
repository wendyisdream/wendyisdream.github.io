---
title: uftrace 코드 분석- function size field 
categories: [Blogging, Uftrace]
tags: [study]
---

### uftrace 코드 분석 및 size filed 추가 
uftrace에서 report/graph/tui 에서 사용되는 data structure 파악을 위해 doxygen의 data structure 결과에 uftrace에서 record 결과로부터 내부 data structure에 메모리를 할당하고 값을 업데이트하는 부분을 따라가보며 구조체 옆에 함수 이름을 표기해 보았다

 

![이미지](https://wendyisdream.github.io/images/data_structures.png "uftrace data structures")



size 는 symbol에 대한 데이터를 업데이트 하는 load_module_symbol_file  함수에서 업데이트 한다.  
아래와 같이 addr로 sorting된 상태이기 때문에 이전 addr와의 차이를 size로 저장한다. 

---
``` c
load_module_symbol_file  
 

      if (symtab->nr_sym > 1 && sym[-1].size == 0)
            sym[-1].size = sym->addr - sym[-1].addr;
```

+ report/graph/tui에 size field 추가 출력을 위해 수정해 본 commit
<https://github.com/namhyung/uftrace/commit/19b561a24709ef1b00acf8e27c94c770b226eb61>


다음은 uftrace에 function size filed에 대한 결과를 확인해본것이다.

### size 필드의 값의 실제와의 비교
 
#### readelf의 결과
```console
    65: 00000000000007ba    42 FUNC    GLOBAL DEFAULT   14 foo  
    70: 00000000000007e4    97 FUNC    GLOBAL DEFAULT   14 main  
```

#### gdb disas의 결과
```console
    (gdb) disas main  
    Dump of assembler code for function main:  
    0x00000000000007e4 <+0>:     push   %rbp  
    0x00000000000007e5 <+1>:     mov    %rsp,%rbp  
    0x00000000000007e8 <+4>:     sub    $0x20,%rsp  
    0x00000000000007ec <+8>:     callq  *0x2007f6(%rip)        # 0x200fe8  
    0x00000000000007f2 <+14>:    mov    %edi,-0x14(%rbp)  
    0x00000000000007f5 <+17>:    mov    %rsi,-0x20(%rbp)  
    0x00000000000007f9 <+21>:    mov    -0x20(%rbp),%rax  
    0x00000000000007fd <+25>:    add    $0x8,%rax  
    0x0000000000000801 <+29>:    mov    (%rax),%rax  
    0x0000000000000804 <+32>:    test   %rax,%rax  
    0x0000000000000807 <+35>:    je     0x83e <main+90>  
    0x0000000000000809 <+37>:    mov    -0x20(%rbp),%rax  
    0x000000000000080d <+41>:    add    $0x8,%rax  
    0x0000000000000811 <+45>:    mov    (%rax),%rax  
    0x0000000000000814 <+48>:    mov    %rax,%rdi  
    0x0000000000000817 <+51>:    callq  0x630 <atoi@plt>  
    0x000000000000081c <+56>:    mov    %eax,-0x4(%rbp)  
    0x000000000000081f <+59>:    movl   $0x0,-0x8(%rbp)  
    0x0000000000000826 <+66>:    jmp    0x836 <main+82>  
    0x0000000000000828 <+68>:    mov    -0x8(%rbp),%eax  
    0x000000000000082b <+71>:    mov    %eax,%edi  
    0x000000000000082d <+73>:    callq  0x7ba <foo>  
    0x0000000000000832 <+78>:    addl   $0x1,-0x8(%rbp)  
    0x0000000000000836 <+82>:    mov    -0x8(%rbp),%eax  
    0x0000000000000839 <+85>:    cmp    -0x4(%rbp),%eax  
    0x000000000000083c <+88>:    jl     0x828 <main+68>  
    0x000000000000083e <+90>:    mov    $0x0,%eax  
    0x0000000000000843 <+95>:    leaveq  
    0x0000000000000844 <+96>:    retq  
    End of assembler dump.  
```

#### uftrace의 symbol size 값
```console
    $ uftrace report -f +size  
    Total time   Self time       Calls        Size  Function  
    ==========  ==========  ==========  ==========  ====================  
        8.408 us    1.545 us           1         108  main  
        6.161 us    1.487 us           3          42  foo  
        4.674 us    4.674 us           3          16  printf  
        0.894 us    0.894 us           1          16  __monstartup  
        0.702 us    0.702 us           1          16  atoi  
        0.393 us    0.393 us           1          16  __cxa_atexit  
```

=> 이값은 test.sym에서의 0x850-0x7e4의 값
```
    00000000000007e4 T main  
    0000000000000850 T __libc_csu_init  
```

uftrace에서는 main의 size가 dummy 공간까지 포함해서 더 큰데,
라이브러리 함수의 시작은 16byte align을 맞추기 때문인것으로 생각된다. (uftrace 빌드 옵션에 따라 실제 크기로 같게 맞출수도 있을까?)

  

### size 값 확인 해보기
#### uftrace자체의 size 확인 큰 순서대로..
```console
    $ uftrace report -f +size -s size  
    Total time   Self time       Calls        Size  Function  
    ==========  ==========  ==========  ==========  ====================  
    379.341 us  375.811 us           2        5488  parse_option  
    654.927 ms   23.988 ms          29        4304  load_module_symtab  
    761.607 ms   55.653 us           1        4288  command_replay  
    20.763 us   15.750 us           5        4176  get_argspec_string  
    104.865 ms   61.194 ms       31640        2704  dd_type  
    180.175 us   22.025 us           3        2672  read_record_mmap  
    37.556 ms    3.028 ms          15        2560  load_elf_dynsymtab  
    444.593 us   49.785 us          13        2400  __read_rstack  
    10.091 us   10.091 us           8        2384  __fstack_consume  
    263.935 ms   44.834 us           1        2224  command_record  
    106.072 ms  102.200 us           1        2208  finish_writers  
    262.807 ms   46.116 us           1        2160  do_main_loop  
    251.592 ms   15.213 ms        6856        1920  dd_encoding  
```

#### gcc 최적화 옵션에 따른 차이

[0] X [1] O3
```console
    $ uftrace report -f +size --diff=uftrace.data.old  
        Total time     Self time         Calls          Size   Function  
    ===========   ===========   ===========   ===========   ====================  
        -3.729 us     -0.667 us            -3           -42   foo  
        -3.062 us     -3.062 us            -3           -16   printf  
        -2.028 us     +0.067 us            +0            +4   main  
        +1.957 us     +1.957 us            +3           +16   __printf_chk  
        -0.654 us     -0.654 us            -1           -16   atoi  
        +0.331 us     +0.331 us            +1           +16   strtol  
        -0.197 us     -0.197 us            +0            +0   __monstartup  
        -0.183 us     -0.183 us            +0            +0   __cxa_atexit  
``` 

#### 참고 
+ function에 isra가 붙는데 이것의 의미는?
<https://stackoverflow.com/questions/13963150/what-does-the-gcc-function-suffix-isra-mean#:~:text=ISRA%20is%20the%20name%20of,created%20by%20IPA%20SRA%20...&text=Perform%20interprocedural%20scalar%20replacement%20of,by%20parameters%20passed%20by%20value.>


+ doxygen 사용법
<https://onecellboy.tistory.com/342>

 
+ stderr 출력을 파일로 저장하기
``` console
    $ uftrace report -vv 2>&1 | tee -a > report_vv.txt
```

#### Todo 

+ session task dlop의 구분하는 과정

vlc player의 task.txt의 결과
```
    SESS timestamp=165175.535895084 pid=19833 sid=a8bc3745995b1ee5 exename="/home/jia/workspace/target_program/vlc/bin/vlc-static" 
    TASK timestamp=165175.535981731 tid=19833 pid=19833
    DLOP timestamp=165175.564566883 tid=19833 sid=a8bc3745995b1ee5 base=7f50f5067000 libname="/home/jia/workspace/target_program/vlc/modules/.libs/libsyslog_plugin.so"
    DLOP timestamp=165175.564805657 tid=19833 sid=a8bc3745995b1ee5 base=7f50f5067000 libname="/home/jia/workspace/target_program/vlc/modules/.libs/libsyslog_plugin.so" 
    ...
    TASK timestamp=165175.565860565 tid=19836 pid=19833  
    TASK timestamp=165175.566957031 tid=19837 pid=19833  
    ...
    DLOP timestamp=165175.567895904 tid=19833 sid=a8bc3745995b1ee5 base=7f50f42f0000 libname="/home/jia/workspace/target_program/vlc/modules/.libs/libalsa_plugin.so"
    DLOP timestamp=165175.567895904 tid=19833 sid=a8bc3745995b1ee5 base=7f50dfcf9000 libname="/usr/lib/x86_64-linux-gnu/libasound.so.2"
```