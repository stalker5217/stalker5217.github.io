---
title: '[STL] 연산자 오버로딩'
toc: true
toc-stick: true
search: true
categories:
  - cpp
tags:
  - cpp
  - c++
excerpt: 'C++ 연산자 오버로딩에 대해서 알아봅시다'
---

## 연산자 오버로딩의 이해  

C++에서는 연산자를 오버로딩하여 기존에 존재하던 연산자의 기능 이외에 다른 기능을 추가할 수 있다.

``` cpp
#include <iostream>

using namespace std;

class Point{
// 좌표 값
private:
  int x, y;

public:
  Point(int x, int y){
    this->x = x;
    this->y = y;
  }

  void showInfo(){
    cout << "(" << this->x << ", " << this->y << ")";
  }

  // 더하기 연산자 오버로딩
  Point operator+(const Point & ref){
    Point pos(x + ref.x, y + ref.y);
    return pos;
  }
};

int main(){
  Point p1(1, 1);
  Point p2(3, 4);

  /**
   *  p1 + p2는
   *  컴파일러가 p1.operator+(p2) 로 인식한다.
   */
  Point p3 = p1 + p2;
  p3.showInfo();// (4,5) 출력

  return 0;
}
```

``` cpp
#include <iostream>

using namespace std;

class Point{
// 좌표 값
private:
  int x, y;

public:
  Point(int x, int y){
    this->x = x;
    this->y = y;
  }

  void showInfo(){
    cout << "(" << this->x << ", " << this->y << ")";
  }

  // friend 선언으로 private 값에 접근 가능
  friend Point operator+(const Point & p1, const Point & p2);  
}

Point operator+(const Point & p1, const Point & p2){
  Point pos(x + ref.xpos, y + ref.ypos);
  return pos;
}
```

연산자 오버로딩의 주의사항  
- 본래의 의도를 벗어난 형태의 오버로딩은 좋지 않다.
- 연산자의 우선순위와 결합성은 바뀌지 않는다.
- 매개변수의 디폴트 값 설정이 불가능하다.
- 연산자의 순수 기능까지 빼앗을 수는 없다. (오버라이딩 불가)


## 증감연산자 오버로딩

``` cpp
class Point{
// 좌표 값
private:
  int x, y;

public:
  Point(int x, int y){
    this->x = x;
    this->y = y;
  }

  /**
   *  prefix 증가 연산자
   *  lvalue를 return. ++(++num) 같은 연산이 가능
   */
  Point& operator++(){
	  x += 1;
	  y += 1;
	  
	  return *this;
  }

  /**
   *  postfix 증가 연산자
   *  parameter에 int는 전위연산과 구분하기 위함이다. 다른 의미는 없다.
   *  rvalue를 return. (num++)++ 같은 연산은 컴파일 에러
   */
  const Point operator++(int){
	  const Point ret(x, y);
	  x += 1;
	  y += 1;
	  return ret;
  }
}
```

## 교환 법칙 문제

``` cpp
class Point{
// 좌표 값
private:
  int x, y;

public:
  Point(int x, int y){
    this->x = x;
    this->y = y;
  }

  Point operator*(int times){
	  Point pos(x * times, y * times);
	  return pos;
  }

  /**
   *  위의 함수만 정의하면 
   *  Point p2 = 2 * p1;
   *  해당 식을 실행할 수 없다.
   *  교환 법칙을 위해서는 별도로 외부 선언이 필요하다.
   */
   friend Point operator*(int times, Point & ref);
};

friend Point operator*(int times, Point & ref){
	Point pos(ref.x * times, ref.y * times);
	return pos;
}
```

## cout, cin, endl  

입출력에 사용되는 ```<<``` 또한 연산자이며 구조는 대략 다음과 같다.

``` cpp
namespace mystd{
	using namespace std;

	class ostream:{
	public:
		Ostream& operator<< (char * str){
			printf("%s", str);
			return *this;
		}
		
		Ostream& operator<< (char str){
			printf("%c", str);
			return *this;
		}
		
		Ostream& operator<< (int num){
			printf("%d", num);
			return *this;
		}
		
		Ostream& operator<< (double e){
			printf("%f", e);
			return *this;
		}
		
		Ostream& operator<< (ostream& (*fp)(Ostream &ostm)){ // 함수 포인터
			return fp(*this); // 해당 함수 실행
		}

		Ostream& endl(ostream &ostm){
			ostm<<'\n';
			flush(stdout);
			return ostm;
		}
	}

	ostream cout;
}

int main(){
	using mystd::cout;
	using mystd::endl;
	
	cout<<33<<endl;
	
	return 0;
}
```

이를 이용해서 객체의 출력도 cout를 통해서 할 수 있다.

```cpp
class Point{
private:
	..
public:
	..
	Friend ostream& operator<<(ostream&, const Point&);
}

Ostream& operator<<(ostream& os, const Point pos){
	os << '['  << pos.xpos << ", " << pos.ypos << "]";
	return os;
}
```


## 대입 연산자 오버로딩  

대입연산자도 오버로딩이 가능하며, 이는 Copy constructor의 특성과 비슷하다.  
- 정의하지 않으면 디폴트 대입 연산자가 삽입된다. 
- 디폴트 대입 연산자는 shallow copy이다.
- deep copy 필요시 직접 정의해야한다.


``` cpp
int main(){
	Point pos1(5, 7);
	Point pos2 = pos1; // Copy constructor에 의한 객체 생성

	Point pos3(9, 10); 
	pos3 = pos1; // 두 객체 모두 초기화가 진행된 상태이며 대입 연산자 실행
}
```

> **상속 구조에서의 대입 연산자 호출**
- 생성자 : 자식 클래스의 생성자는 아무런 명시를 하지 않아도, 부모 클래스의 생성자를 호출한다.
- 대입 : 자식 클래스에서 명시를 하지 않으면, 부모 클래스의 대입 연산자가 호출되지 않고 멤버 대 멤버 복사를 진행할 수 없다,


## 배열의 인덱스 연산자 오버로딩

``` cpp
#include <iostream>

using namespace std;

class BoundaryCheckIntArray{
private:
	int * arr;
	int arrlen;
	
	BoundaryCheckIntArray(const BoundaryCheckIntArray& arr);
	BoundaryCheckIntArray& operator=(const BoundaryCheckIntArray& arr);
	// BoundaryCheckIntArray cpy1 = orgin, cpy2 = orgin 같은 문장을 막기 위해 private로 선언

public:
	BoundaryCheckIntArray(int len) 
	: arrlen(len)
	{
		arr = new int[len];
	}
	
	int& operator[](int idx){
		if(idx < 0 || idx >= arrlen){
			cout << "out of index" << endl;
			exit(1);
		}
		
		return arr[idx];
	}
	
	// 단순 값 참조를 위한 const 함수 오버로딩
	// 함수의 const 선언 유무도 오버로딩의 조건에 해당한다.
	int& operator[](int idx) const{ 
		if(idx < 0 || idx >= arrlen){
			cout << "out of index" << endl;
			exit(1);
		}
		
		return arr[idx];
	}
	
	~BoundaryCheckIntArray(){
		delete[] arr;
	}
};

int main(){
	BoundaryCheckIntArray mArr(10);
	
  mArr[0] = 1;
  mArr[1] = 2;
  cout << mArr[1] << endl;
}
```