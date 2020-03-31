---
title: '[STL] Priority Queue(우선순위 큐)'
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

## 우선순위 큐  

우선순위 큐는 큐에서 데이터를 꺼낼때 미리 정의된 우선순위 중 가장 높은 데이터가 빠져나오는 자료구조이다.
우선순위 큐(PQ)는 binary tree인 힙구조를 사용하여 많이 구현되며 데이터의 삽입, 삭제, 탐색이 O(nlogn)으로 이루어진다.  


예시
``` cpp
  #include <iostream>
  #include <vector>
  #include <queue>
  #include <functional>

  using namespace std;

  // compare 구조체의 구현
  struct compare{
    bool operator()(const int & a, const int & b){
      if(a >= b){
        return true;
      }
      else{
        return false;
      }
    }
  };

  int main(){
    /*
    * template parameter는 
    * 1. T
    * 2. Container
    * 3. Compare
    */ 
    priority_queue<int, vector<int>, less<int>> pq1;
    priority_queue<int, vector<int>, greater<int>> pq2;
    priority_queue<int, vector<int>, compare> pq3;

    pq1.push(1); pq2.push(1); pq3.push(1);
    pq1.push(9); pq2.push(9); pq3.push(9);
    pq1.push(4); pq2.push(4); pq3.push(4);
    pq1.push(6); pq2.push(6); pq3.push(6);
    pq1.push(7); pq2.push(7); pq3.push(7);
    pq1.push(2); pq2.push(2); pq3.push(2);
    pq1.push(8); pq2.push(8); pq3.push(8);
    pq1.push(3); pq2.push(3); pq3.push(3);
    pq1.push(5); pq2.push(5); pq3.push(5);

    // 987654321
    while(!pq1.empty()){
      cout << pq1.top();
      pq1.pop();
    }
    cout << endl;
    
    // 123456789
    while(!pq2.empty()){
      cout << pq2.top();
      pq2.pop();
    }
    cout << endl;

    // 123456789
    while(!pq3.empty()){
      cout << pq3.top();
      pq3.pop();
    }
    cout << endl;
  }
```

