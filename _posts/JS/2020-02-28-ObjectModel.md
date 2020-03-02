---
title: '[Javascript] BOM, DOM'
excerpt: "Brower Object Model, Document Object Model"
search: true
categories:
  - javascript
tags:
  - javascript
  - js
---

1. BOM (Browser Object Model)  
	
	브라우저 객체 모델은 브라우저 탭 혹은 창의 모델을 생성하며, 최상위 객체는 ```window```이다.  
	공식적으로 표준 모델은 존재하지 않지만, 모든 브라우저가 비슷한 속성을 가지며 또한 동작하기에 유용하게 쓰인다.  

	Window : 현재 브라우저 창이나 탭  
	- Document : 현재 로드된 웹 페이지
	- History : 히스토리에 기록된 웹 페이지들
	- location : 현재 페이지의 URL
	- Navigator : 브라우저와 관련된 정보
	- Screen : device Display 정보

2. DOM (Document Object Model)  
	
	문서 객체 모델은 현재 웹 페이지의 모델을 생성하며, 최상위 객체는 ```document```이다.  

	![domtree](/assets/images/javascript/DOM_tree.gif)  

	이처럼 ```document``` 객체 아래 HTML 구조가 저장되며, HTML 내의 문자열들은 모두 각자 DOM Nodefh 표현된다.  

	document object를 통해 요소 노드에 접근하기
	``` javascript
	// id 값이 일치하는 요소를 선택한다.
	getElementById(id);

	// css 선택자와 일치하는 요소를 선택한다. 
	// 가장 먼저 찾아지는 하나를 리턴한다.
	querySelector(css_selector);

	// class 값이 일치하는 모든 요소를 선택한다.
	getElementByClassName(class_name);

	// tag name 값이 일치하는 모든 요소를 선택한다.
	getElementByTagName(tag_name);

	// css 선택자와 일치하는 모든 요소를 선택한다.
	querySelectorAll(css_selector);
	```

	노드 속성을 통해 DOM 탐색하기
	``` javascript
	// 부모 요소를 선택한다
	curNode.parentNode

	// 형제 노드를 선택한다
	curNode.previousSibling
	curNode.nextSibling

	// 자식 노드를 선택한다
	curNode.firstChild
	curNode.lastChild
	```