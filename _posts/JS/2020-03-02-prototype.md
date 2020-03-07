---
title: '[Javascript] Closure'
toc: true,
toc-stick: true,
search: true
categories:
  - javascript
tags:
  - javascript
  - js
---

## ProtoType이란?  
Javasciprt에서는 일반적인 객체지향언어와 달리 class 개념이 존재하지 않는다. 
물론, ECMAScript6 부터는 ```class``` 문법을 지원하고 있지만, 이는 엄밀히 말해서 prototype을 기반으로하는 문법일 뿐이다.




avaScript : 프로토타입(prototype) 이해
 Nextree  Mar 25, 2014  16 Comments
JavaScript는 클래스라는 개념이 없습니다. 그래서 기존의 객체를 복사하여(cloning) 새로운 객체를 생성하는 프로토타입 기반의 언어입니다. 프로토타입 기반 언어는 객체 원형인 프로토타입을 이용하여 새로운 객체를 만들어냅니다. 이렇게 생성된 객체 역시 또 다른 객체의 원형이 될 수 있습니다. 프로토타입은 객체를 확장하고 객체 지향적인 프로그래밍을 할 수 있게 해줍니다. 프로토타입은 크게 두 가지로 해석됩니다. 프로토타입 객체를 참조하는 prototype 속성과 객체 멤버인 proto 속성이 참조하는 숨은 링크가 있습니다. 이 둘의 차이점을 이해하기 위해서는 JavaScript 함수와 객체의 내부적인 구조를 이해 해야합니다. 이번 글에서는 JavaScript의 함수와 객체 내부 구조부터 시작하여 프로토타입에 대해 알아보겠습니다.







대망의 

prototype
constructor
__proto__

constructor -> new instance

생성자의 프로토타입 

생성자 함수의

[1, 2, 3]

Array
- from
- isArray
- of
- arguments
- length
- name

prtotype 아래
-concat
- filter
- foreach
- map
- push
- pop 


메소드 상속 및 동작 원리

prototype도 객체이므로 object를 상속하고,, object.prototype에 정의된 것두 접근,

arr = [1,2,3]
arr.toString = function() {
	return this.join('_');
}

console.log(arr.toString 1_2_3
console.log(arr.__proto__.toString.call(arr)); 1,2,3
console.log(arr.__proto__.__proto__.toString.call(arr)) ; object


arr = [1.2.3]
Array.prototype.toString = function(){
	return this.join ('_;)l;

console.log(arr.toString 1_2_3
console.log(arr.__proto__.toString.call(arr)); 1_2_#
console.log(arr.__proto__.__proto__.toString.call(arr)) ; object