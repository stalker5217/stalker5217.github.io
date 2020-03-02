---
title: '[STL]Generic Programming'
excerpt: "Generic Programming"
search: true
categories:
  - cpp
tags:
  - cpp
  - c++
  - stl
  - standard template library
---

## Generic Programming이란?
> 제네릭(Generic)은 데이터 형식에 의존하지 않고, 하나의 값이 여러 다른 데이터 타입들을 가질 수 있는 기술에 중점을 두어 재사용성을 높일 수 있는 프로그래밍 방식이다.


``` cpp
int max(int a, int b){
	return (a > b) ? a : b;
}

double max(double a, double b){
	return (a > b) ? a : b;
}

__int64 max(__int64 a, __int64 b){
	return (a > b) ? a : b;
}

........
```

위의 예제를 보면 내부 로직이 같은 함수들이 자료형의 차이로 인해 수 많이 Overloading 하고 있다.  
만약, 내부 로직이 변경이 된다면 저 모든 함수들의 로직을 함께 바꾸어 주어야 한다.  

이러한 것을 C++에서는 **template**를 활용하여 해결한다.  

``` cpp
template<typename T>{
T max(T a, T b){
	return (a>b) ? a : b;
}
```
여기서 T는 다양한 자료형으로 mapping될 수 있으며, 이를 generic화 했다고 한다.  



## 개선하기
위의 예제는 3가지 단계를 거쳐서 개선할 수 있다.

``` cpp
// 1단계
// - 참조자를 이용하여 속도를 높인다.
// - const를 사용하여 값의 변경을 막는다.
template <typename T>
const T& max(const T& a, const T& b){
	return (a > b) ? a : b;
}
```

``` cpp
// 2단계
// 함수의 파라미터가 서로 다른 것이 들어 온다면?
template <typename T1, typename T2>
const T1& max(const T1& a, const T2& b>{
	return (a > b) ? a : b;
}
```

``` cpp
// 3단계 - Specialization
// 위의 코드 같은 경우 T2로 들어오는 Data type이 더 크다면
// return type 불일치로 error가 발생한다.
// 이 경우 예외를 처리할 수 있다.
template <typename T1, typename T2>
const T1& max(const T1& a, const T2& b>{
	return (a > b) ? a : b;
}

template <>
const double& max(const int& a, const double& b){
	return (a > b) ? a : b;
}
```


## Class Template
함수만 아니라 클래스의 경우에도 ```tempalte``` 적용이 가능하다.

아래 두 가지 예시를 첨부한다.
``` cpp
// 예시 1 - Stack
template <typename T, int SIZE>
class Stack
{
private:
	T m_aData[SIZE];
	int m_Count;
	
public:
	Stack()
	{
		Clear();
	}

	// 초기화 한다.
	void Clear()
	{
		m_Count = 0;
	}

	// 스택에 저장된 개수
	int Count()
	{
		return m_Count;
	}

	// 저장된 데이터가 없는가?
	bool IsEmpty()
	{
		return (0 == m_Count) ? true : false;
	}

	// 데이터를 저장한다.
	bool push( T data )
	{
		// 저장 할수 있는 개수를 넘는지 조사한다.
		if( m_Count >= MAX_COUNT )
		{
			return false;
		}
		// 저장 후 개수를 하나 늘린다.
		m_aData[ m_Count ] = data;
		++m_Count;
		
		return true;
	}

	T pop()
	{
		// 저장된 것이 없다면 0을 반환한다.
		if( m_Count < 1 )
		{
			return 0;
		}
		// 개수를 하나 감소 후 반환한다.
		--m_Count;
		return m_aData[ m_Count ];
	}
}
```

``` cpp
template <typename T>
class MySingleton{
private:
	static T& _Singleton;
	
	MySingletone() {}
	virtual ~MySingletone() {}

public:
	// 이 멤버를 통해서만 생성이 가능하다.
	static T* GetSingleton()
	{
		// 아직 생성이 되어 있지 않으면 생성한다.
		if( NULL == _Singleton ) {
			_Singleton = new T;
		}
		return ( _Singleton );
	}

	static void Release()
	{
		delete _Singleton;
		_Singleton = NULL;
	}
}


class MyObject : public MySingleton<MyObject>
{
public:
	MyObject() : _nValue(10) {}
	void SetValue( int Value ) { _nValue = Value;}
	int GetValue() { return _nValue; }

private :
	int _nValue;
};

```