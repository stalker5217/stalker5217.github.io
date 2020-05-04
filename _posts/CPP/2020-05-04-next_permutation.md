---
title: '[STL] next_permutation'
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

## next_permutation

``<algorithm>`` 헤더에 정의되어 있으며 함수 원형은 다음과 같다.  

``` cpp
// default
template <class BidirectionalIterator>
bool next_permutation (BidirectionalIterator first, BidirectionalIterator last);

// custom
template <class BidirectionalIterator, class Compare>
bool next_permutation (BidirectionalIterator first, BidirectionalIterator last, Compare comp);
```

해당 함수는 구성된 데이터의 순열을 만들어서 보여 준다. 현재 기준으로 operator< 연산을 통해, 다음 순열이 존재한다면 true를 반환하고 container의 값을 바꾸어 준다.


``` cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main(){
    vector<int> arr;

    arr.push_back(1);
    arr.push_back(2);
    arr.push_back(3);

    /**
     * 출력
     * 1 2 3
     * 1 3 2
     * 2 1 3
     * 2 3 1
     * 3 1 2
     * 3 2 1
     */
    do{
        for(int cur : arr) cout << cur << " ";
        cout << "\n";
    }while(next_permutation(arr.begin(), arr.end()));

    /**
     * 자매품 prev_permutation도 존재한다.
     * 모든 경우의 수를 표현하기 위해서는 vector가 역순으로 정렬되어 있어야 한다.
     * 출력
     * 3 2 1
     * 3 1 2
     * 2 3 1
     * 2 1 3
     * 1 3 2
     * 1 2 3
     */
     sort(arr.begin(), arr.end(), greater<int>());
     do{
        for(int cur : arr) cout << cur << " ";
        cout << "\n";
    }while(prev_permutation(arr.begin(), arr.end()));
}
```