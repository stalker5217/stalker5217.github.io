---
title: Javascript - BOM, DOM
excerpt: "Javascript"
search: true
categories:
  - javascript
tags:
  - javascript
  - js
---

1. BOM (Browser Object Model)  
	
	브라우저 객체 모델은 브라우저 탭 혹은 창의 모델을 생성하며, 최상위 객체는 window이다.  
	공식적으로 표준 모델은 존재하지 않지만, 모든 브라우저가 비슷한 속성을 가지며 또한 동작하기에 유용하게 쓰인다.  

	Window : 현재 브라우저 창이나 탭  
	- Document : 현재 로드된 웹 페이지
	- History : 히스토리에 기록된 웹 페이지들
	- location : 현재 페이지의 URL
	- Navigator : 브라우저와 관련된 정보
	- Screen : device Display 정보

2. DOM (Document Object Model)  
	
	문서 객체 모델은 현재 웹 페이지의 모델을 생성하며, 최상위 객체는 document이다.  

	![domtree](/assets/images/javascript/DOM_tree.gif)