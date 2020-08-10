---
title: '[Algorithm] Binary Search Tree'
toc: true
toc-stick: true
search: true
categories:
  - algorithm
tags:
  - algorithm
  - tree
  - binary search tree
excerpt: '이진 검색 트리에 대해서 알아봅시다'
---

## 이진 검색 트리

트리는 계층 구조 표현 외에도 검색(Search)를 위한 용도로 사용할 수 있다.
그 중 대표적인 것이, 이진 검색 트리(Binary Search Tree)이다.

> 이진 트리는 각 노드가 왼쪽과 오른쪽, 최대 두 개의 자식 노드만을 가질 수 있는 트리를 의미한다.  

![basic_binary_tree](/assets/images/algorithm/basic_binary_tree.png)


**순회**  
이진 검색 트리에서 왼쪽 자식은 현재 노드의 우선 순위보다 항상 낮고, 오른쪽 자식은 우선 순위가 더 높다.  
이러한 특징으로 아래 특징까지 도출해낼 수 있다.

- 루트 노드에서 왼쪽 아래로 쭉 내려가서 만나는 노드가 우선 순위가 가장 낮다.
- 루트 노드에서 오른쪽 아래로 내려가서 만나는 노드가 우선 순위가 가장 높다.
- 중위 순회하면 정렬된 원소의 목록을 얻을 수 있다.


**검색**  
Binary Search와 유사하게 한 번 탐색으로 후보군을 절반씩 삭제해가며 O(logN)에 값을 찾는다.  


## 이진 검색 트리의 동작  

위에서 정리한 특징은 정렬된 배열에 비해서 나을게 없다. 
이진 검색 트리의 장점은 노드의 삽입과 삭제에 있어 배열보다 효율적으로 동작할 수 있다.  

``` cpp
template <typename T>
class Tree;

template <typename T>
class TreeNode{
  friend class Tree<T>;
  private:
    T value;
    TreeNode * left;
    TreeNode * right;
  
  public:
    TreeNode(T value)
    :value(value), left(nullptr), right(nullptr)
    {}
};

template <typename T>
class Tree{
  private:
    TreeNode<T> * root;
  
  public:
    Tree():root(nullptr)
    {}

    // 노드 탐색
    TreeNode<T> * find(T value){
      TreeNode<T> * cur = getRoot();

      if(cur == nullptr) return nullptr;
      while(cur != nullptr){
        if(cur->value == value) return cur;
        else if(cur->value > value) cur = cur->left;
        else if(cur->value < value) cur = cur->right;
      }

      return cur;
    }

    // 데이터 삽입(중복 고려X)
    void insert(T value){
      TreeNode<T> * newNode = new TreeNode<T>(value);

      if(root == nullptr){
        root = newNode;
        return;
      }

      TreeNode<T> * curNode = getRoot();
      while(true){
        if(curNode->value > newNode->value){
          if(curNode->left == nullptr){
            curNode->left = newNode;
            break;
          }
          else curNode = curNode->left;
        }
        else{
          if(curNode->value < newNode->value){
            if(curNode->right == nullptr){
              curNode->right = newNode;
              break;
            }
            else curNode = curNode->right;
          }
        }        
      }
    }

    // 데이터 삭제
    void remove(T value){
      // 삭제 대상 찾기
      TreeNode<T> * targetNode = getRoot();
      TreeNode<T> * parentNode = nullptr;

      bool findTarget = false;
      while(targetNode != nullptr){
        if(targetNode->value == value) break;
        
        parentNode = targetNode;
        if(targetNode->value > value) targetNode = targetNode->left;
        else targetNode = targetNode->right;
      }

      // 못 찾으면 종료
      if(targetNode == nullptr) return;

      // Case 1. 삭제 노드의 자식이 없을 때
      // 그냥 삭제!
      if(targetNode->left == nullptr && targetNode->right == nullptr){
        if(parentNode->left == targetNode) parentNode->left = nullptr;
        else parentNode->right = nullptr;
      }
      // Case 2. 삭제 노드의 자식이 하나 일 때
      // 삭제 노드의 자식을 부모와 연결시켜 줌
      else if(targetNode->left == nullptr){
        if(parentNode->left == targetNode) parentNode->left = targetNode->right;
        else parentNode->right = targetNode->right;
      }
      else if(targetNode->right == nullptr){
        if(parentNode->left == targetNode) parentNode->left = targetNode->left;
        else parentNode->right = targetNode->left;
      }
      // Case 3. 삭제 노드의 자식이 둘 일 때
      // 오른쪽 자식에서 가장 작은 값을 가지는 노드를 올리고 삭제한다.
      else{
        TreeNode<T> * swapNode = targetNode->right;
        while(swapNode->left != nullptr) swapNode = swapNode->left;
        
        T newValue = swapNode->value;
        remove(swapNode->value);

        if(parentNode->left == targetNode){
          parentNode->left = new TreeNode<T>(newValue);
          parentNode->left->left = targetNode->left;
          parentNode->left->right = targetNode->right;
        }
        else{
          parentNode->right = new TreeNode<T>(newValue);
          parentNode->right->left = targetNode->left;
          parentNode->right->right = targetNode->right;
        }

        
      }

      delete targetNode;
    }
    
    TreeNode<T>* getRoot(){
      return root;
    }

    void inOrder(TreeNode<T> * cur){
      if(cur != nullptr){
        inOrder(cur->left);
        cout << cur->value << " ";
        inOrder(cur->right);
      }
    }
};

int main(){
  Tree<int> tree;
  tree.insert(27);
  tree.insert(13);
  tree.insert(10);
  tree.insert(15);
  tree.insert(30);
  tree.insert(28);
  tree.insert(32);
  tree.insert(31);
  tree.insert(40);
  tree.insert(35);

  tree.remove(32);

  tree.inOrder(tree.getRoot());
}
```  


## 균형 잡힌 이진 검색 트리

만약, 숫자의 크기를 우선 순위로 갖는 트리에서 삽입이 1부터 N까지 순차적으로 삽입됐다고 가정하자.
루트 노드는 1이고, 오른쪽 아래로만 노드가 추가되며 결국에서는 높이가 N-1인 트리가 만들어진다.

이러한 트리는 ***기울어졌다(skewed)***라고 한다. 
이렇게 되면 트리 구조를 사용하는 의미가 없고,
단순 배열보다 못한 퍼포먼스를 가진다.

이 점을 해결하기 위해서, 노드들이 한 쪽으로 몰리지 않게 조정을 해주는 자료구조인 
AVL Tree, Red-Black Tree 등의 **Balanced Tree**가 존재하며, 
대부분 표준 라이브러리의 이진 검색 트리는 레드 블랙 트리를 사용해서 구현되어 있다.


## 트립  

교과서적인 자료구조인 AVL Tree, Red-Black Tree 등은 구현이 상당히 힘들다.
PS를 진행하며 구현하기에는 공수가 너무 많아 적합하지 않고 다른 대안이 사용되는데, 
많이 사용하는 것 중 대표적인 것이 **Treap**이다.