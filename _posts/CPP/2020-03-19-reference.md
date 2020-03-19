---
title: '[C++] Reference(참조자)'
toc: true,
toc-stick: true,
search: true
categories:
  - cpp
tags:
  - cpp
  - c++
---

## 참조자  

C++에서 별칭이라고 할 수 있다. 참조자는 하나의 메모리 공간에 둘 이상을 부여할 수 있다.

```&```는 이미 선언된 변수의 앞에 오면 그 변수의 주소 값의 반환을 명령하는 뜻이 되지만,
새로 선언되는 변수의 이름 앞에 오면 **참조자의 선언**을 뜻한다.  

``` cpp
// 변수 num의 주소 값을 ptr에 저장
int * ptr = &num;

// 변수 num에 대한 참조자 num_ref
int & num_ref = num;
```

참조자의 특징  
- 선언됨과 동시에 반드시 누군가를 참조해야 한다. (참조자의 선언만 하면 오류 발생)
- NULL로 초기화가 불가능하다.
- 상수를 대상으로 참조가 불가능하다.
- 포인터 변수도 참조할 수 있다.


## 참조자와 함수  

- Call by value  
	: 값을 인자로 전달하는 함수 호출 방식. 함수 외부 변수에 접근할 수 없다.
- Call by reference
	: 주소 값을 인자로 전달하는 함수 호출 방식. 주소 값을 통해 접근하기 때문에 함수 외부 변수에 접근 가능하다.

``` cpp
// 참조자를 이용한 call by reference 구현
void swap(int & ref1, int & ref2){
	int temp = ref1;
	ref1 = ref2;
	ref2 = temp;
}

// const 선언
void square(const int & ref){
	return ref * ref;
}
```

> 참조자를 매개변수로 하는 함수의 경우 매개변수로 들어가는 변수의 값이 바뀔 수 있다.
함수 내에서, 이러한 값의 변경을 진행하지 않는 경우에는 참조자를 const로 선언하여 함수의 원형만 봐도 값의 변경이 이루어짐을 알 수 있게한다.


## 기타 참조자  

**반환형의 참조형인 경우**

``` cpp
int& func(int & ref){
	ref++;
	return ref;
}

// 위와 같은 경우 두 가지 케이스로 return 받을 수 있다.

// 1. 이 경우 변수 x는 전달된 n과 같은 메모리를 가리키는 n의 참조자가 된다.
int & x = func(n);

// 2. 이 경우 함수가 반환하는 값을 단순히 저장하고 n과는 다른 메모리를 가리킨다.
int y = func(n);
```
	
const 참조자  
``` cpp
const int num = 20;
int & ref = num; // 에러 발생

const int num = 20;
const int & ref = num; // 가능
```

상수 참조자
``` cpp
// 30은 임시 변수로 메모리 공간에 저장된다.
const int & ref = 30;

// 이와 같은 함수에서 상수 파라미터를 처리할 수 있다.
int adder(const int & n1, const int & n2){
	return n1 + n2;
}
```