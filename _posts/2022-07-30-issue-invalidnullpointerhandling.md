---
title: uftrace 이슈 수정 - clang asan에서 applying zero offset to null pointer 에러 발생건
categories: [Blogging, Uftrace]
tags: [study]
---
#### 이슈 내용

clang으로 ASAN=1 unittest 빌드 후 실행 시 runtime error: applying zero offset to null pointer 에러 발생  
```
/uftrace/utils/dwarf.c:2412:16: runtime error: applying zero offset to null pointer
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior
```

- uftrace에서 ASAN=1로 빌드시 적용되는 flag

 
```
Makefile 
ifeq ($(ASAN), 1)
  ASAN_CFLAGS       := -O0 -g -fsanitize=address,leak,undefined
  UFTRACE_CFLAGS    += $(ASAN_CFLAGS)
...
endif
```
cflag에 -O0 -g -fsanitize=address,leak,undefined가 추가된다.


- UBSAN  

clang에서 UndefinedBehaviorSanitizer에서 체크하는 항목은 다음의 페이지에서 확인할수 있다.
<https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html#ubsan-checks>


위의 검출이 해당되는 항목은 다음과 같다.    
```
-fsanitize=pointer-overflow: Performing pointer arithmetic which overflows, or where either the old or new pointer value is a null pointer (or in C, when they both are).
```


#### 원인

이슈가 발생한 부분은 rb tree의 node를 따라가는데,   
마지막 노드는 null이 되고, 이 값은 rb_entry의 첫번째 argument에 전달된다. 
rb_entry는 container_of로 재정의된다.

- container_of 란?  

```
container_of(ptr, type, member)

ptr – the pointer to the member.
type – the type of container struct this is embedded in.
member – the name of the member within the struct.

It returns the address of the container structure of the member.
```

아래의 페이지의 자세한 설명이 이해를 도와주었다!
<https://www.bhral.com/post/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%BB%A4%EB%84%90%EC%9D%98-container_of-%EB%A7%A4%ED%81%AC%EB%A1%9C>


container_of의 구현은

```
#define container_of(ptr, type, member) ({         \
    const typeof( ((type *)0)->member ) *__mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

와 같음으로 
ptr이 null로 전달되는 경우 null에 대한 offsetof(type,member) (여기서는 0) 의 뺄셈이 되어 ubsan에 의해서
applying zero offset to null pointer 를 발생시켰다.

#### 수정
rb tree의 마지막 모드이면 tree traversal를 중지하고 빠져나올 수 있도록 null 체크를 추가하였다.  
rb_entry에 null을 전달하지 않도록..

<https://github.com/namhyung/uftrace/pull/1494>


#### 참고1) rb tree란
https://ko.wikipedia.org/wiki/%EB%A0%88%EB%93%9C-%EB%B8%94%EB%9E%99_%ED%8A%B8%EB%A6%AC




