---
title: '[Javascript] 프로토타입'
toc: true,
toc-stick: true,
search: true
categories:
  - javascript
tags:
  - javascript
  - js
---

## ProtoType
Prototype은 디자인패턴 중 하나이다.
객체의 생성이 이미 존재하는 객체를 clone하여 생성하는 방식이다.
<br>
자바스크립트는 일반적인 객체지향언어와 달리 class 개념이 존재하지 않으며 prototype을 통해 객체를 생성한다.
물론, ECMAScript6 부터는 ```class``` 문법을 지원하고 있지만, 이는 엄밀히 말해서 prototype을 기반으로하는 문법일 뿐이다.  

![proto_type](/assets/images/javascript/prototype.png)

``` js
var myArr = new Array();
myArr.push(1);
myArr.push(2);
myArr.push(3);
```

위 코드에서 ```new``` 키워드로 통해 배열을 생성하면 ```myArr```은 ```Array```의 ```prototype```을 가리키게 되며
이를 통해 push와 같은 메소드를 사용할 수 있는 것이다.









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
- concat
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