---
title: '[STL] lower_bound, upper_bound'
toc: true
toc-stick: true
search: true
categories:
  - cpp
tags:
  - cpp
  - c++
  - stl
  - standard template library
---

## lower_bound와 #upper_bound

<algorhtm> 헤더에 정의되어 있으며 함수 원형은 다음과 같다.  

``` cpp

template< class ForwardIt, class T >
ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value );

template< class ForwardIt, class T, class Compare >
ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value, Compare comp );

```


- lower_bound : 주어진 value 값 이상인 값을 찾아 iterator를 return 한다.  
- upper_bound : 주어진 value 값을 초과하는 값을 찾아 iterator를 return 한다.  

> 해당 함수들은 binary search를 기반으로 동작하므로 연산 대상이 되는 container는 정렬이 되어 있어야 한다.  


``` cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main(){
    vector<int> arr;

    arr.push_back(13);
    arr.push_back(21);
    arr.push_back(35);
    arr.push_back(40);
    arr.push_back(50);

    cout << *lower_bound(arr.begin(), arr.end(), 21) << endl;
    // index 반환
    cout << lower_bound(arr.begin(), arr.end(), 21) - arr.begin() << endl;

    cout << *upper_bound(arr.begin(), arr.end(), 21) << endl;
    // index 반환
    cout << upper_bound(arr.begin(), arr.end(), 21) - arr.begin() << endl;

    return 0;
}
```