---
title: '[Javascript] Block Scope Variable'
toc: true
toc-stick: true
search: true
categories:
  - javascript
tags:
  - javascript
  - js
  - ES6
  - ECMA Script 6
excerpt: 'ES6의 block scope 변수에 대해서 알아봅시다'
---

## Block Scope Variable

ES5까지는 ```var```가 유일한 변수 선언으로 이는 함수 단위의 스코프를 가졌다.  

``` js
function test(){
	if(true){
		var myVar = 10;
	}
	
	console.log(myVar);
}
test();
/*
 *	var 변수는 함수 단위로 스코프를 가진다.
 *	변수 myVar가 if block 내에서 생성 되어도 console은 10을 출력한다.
 */
```  

<br/>

**let, const**  
ES6부터는 ```let```, ```const```라는 유형의 변수가 새롭게 등장했고, 이는 블록 단위의 스코프를 가진다.

``` js
function test(){
	if(true){
		let myLet  = 10;
	}
	
	console.log(myLet);
}
test();
/*
 *	let 변수는 블록 단위로 스코프를 가진다.
 *	변수 myLet은 자신을 감싸고 있는 블록 내부에서만 접근 가능하며
 *	console 출력 결과는 'Uncaught ReferenceError: myLet is not defined'가 된다.
 */
```

<br/>

**호이스팅 여부**

``` js
function test(){
	console.log(myVar);

	var myVar = 10;
}
test();
/*
 *	var 변수는 호이스팅이 된다.
 *	console의 출력이 에러가 발생하는 것이 아니라 undefined가 출력된다.
 *	즉, 선언은 되었으나 값의 할당이 이루어지지 않았음을 의미한다.
 */
```

``` js
function test(){
	console.log(myLet);

	let myLet = 10;
}
test();
/*
 *	Uncaught ReferenceError: Cannot access 'myLet' before initialization
 */
```

위 코드는 myLet이라는 변수가 초기화 되기전에는 접근할 수 없다는 에러가 난다.
언뜻 보면 호이스팅이 되지 않는 것처럼 보인다.

``` js
function test(){
	let myLet = 1;
	if(true){
		console.log(myLet);

		let myLet = 2;
	}
}
test();
/*
 *	위 코드의 실행 결과는 어떻게 될까?  
 *	console이 실행될 시점에 myLet에 2를 할당하는 선언은 아래에 있고,
 *	myLet에 1을 할당된 변수는 이미 존재하기 때문에 1이 출력될 것이라고 생각을 하겠지만 에러가 발생한다.
 */
```

![let_hoisting](/assets/images/javascript/let_hoisting.png)

```let```, ```const```도 호이스팅이 일어난다.
호이스팅에 의해서 실제 내부는 다음과 같음을 추측할 수 있다.  

``` js
function test(){
	let myLet;
	myLet = 1;
	if(true){
		let myLet;
		
		console.log(myLet);

		myLet = 2;
	}
}
test();
```  

<br/>

그렇다면 호이스팅이 일어나는데 왜 var 변수와 달리 에러가 날까?  
변수의 생성은 다음과 같은 절차가 있다.

1. 선언
	실행 컨텍스트에 변수가 등록된다.
2. 초기화
	메모리를 할당한다.
3. 할당
	실제 값을 할당한다.

```var```로 선언을 하게 되면 1번과 2번이 동시에 이루어지며 이 때 undefined가 된다.
하지만 ```let```, ```const```는 1번만이 이루어지기에 접근을 아예 하지 못하는 것이다.  


**Temporal Dead Zone (임시 사각 지대)**

정리하자면 ```let```, ```const```도 호이스팅은 일어나서 선언 자체는 스코프 최상단으로 올려진다.
하지만 ```var```과 달리 메모리를 할당하는 작업이 이루어지지 않기 때문에 실제 선언문을 만나기 전까지는 접근을 할 수 가 없는데,
이러한 구간을 TDZ(Temporal Dead Zone)이라고 한다.