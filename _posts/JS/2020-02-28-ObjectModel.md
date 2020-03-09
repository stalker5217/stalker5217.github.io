---
title: '[Javascript] BOM, DOM'
toc: true,
toc-stick: true,
search: true
categories:
  - javascript
tags:
  - javascript
  - js
---
## BOM, DOM

1. **BOM (Browser Object Model)**  
	
	브라우저 객체 모델은 브라우저 탭 혹은 창의 모델을 생성하며, 최상위 객체는 ```window```이다.  
	공식적으로 표준 모델은 존재하지 않지만, 모든 브라우저가 비슷한 속성을 가지며 또한 동작하기에 유용하게 쓰인다.  

	Window : 현재 브라우저 창이나 탭  
	- Document : 현재 로드된 웹 페이지
	- History : 히스토리에 기록된 웹 페이지들
	- location : 현재 페이지의 URL
	- Navigator : 브라우저와 관련된 정보
	- Screen : device Display 정보

2. **DOM (Document Object Model)**  
	
	문서 객체 모델은 현재 웹 페이지의 모델을 생성하며, 최상위 객체는 ```document```이다.  

	![domtree](/assets/images/javascript/DOM_tree.gif)  

	이처럼 ```document``` 객체 아래 HTML 구조가 저장되며, HTML 내의 문자열들은 모두 각자 DOM Node로 표현된다.  




## DOM 객체 조작

1. **document object를 통해 요소 노드에 접근하기**  
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

2. **nodeValue 속성을 이용하여 텍스트 노드에 접근/변경하기**  
	``` html
	<li id="one"><em>신선한</em> 무화과</li>
	```

	``` javascript
	var value = document.getElementById('one').firstChild.nextSibling.nodeValue; // Access Text, value = '무화과'
	document.getElementById('one').firstChild.nextSibling.nodeValue = '수박'
	```  

3. **textContent/innerText 속성을 이용하여 텍스트에 접근/변경하기**  
	``` html
	<li id="one"><em>신선한</em> 무화과</li>
	```

	``` javascript
	// Text Access
	var value = document.getElementById('one').textContent; // Access Text, value = '신선한 무화과'
	var value = document.getElementById('one').innerText; // Access Text, value = '신선한 무화과'
	```

	textContent
	- IE9 이하에서는 지원하지 않음

	innerText(권장하지 않음)
	- 표준이 아니기에 일부 브라우저(Firefox 등)에서 지원하지 않는다.
	- css에 의존적. 만약, 위 예제에서 ```<em>```이 css에 의해 blind 처리 되어있다면 결과에 포함되지 않는다.
	- 위 처럼 css에 의존적으로, 화면에 보여지는지를 결정하기 위한 레이아웃 규칙을 고려하면서 성능 또한 떨어진다.  



4. **HTML 콘텐츠 추가/제거하기**  
	``` html
	<li id="one"><em>신선한</em> 무화과</li>
	```

	``` javascript
	// HTML 교체
	// '<em>신선한</em> 무화과' return 된다.
	var html = document.getElementById('one').innerHTML;
	```

	``` javascript
	// Dom tree에 요소 추가
	var newEl = document.createElement('li');
	var newText = document.createTextNode('수박');
	newE1.append(newText);

	var pos = document.getElementsByTagName('ul')[0];
	pos.appendChild(newE1);
	```

	``` javascript
	// Dom tree에 요소 삭제
	var delEl = document.getElementsByTagName('li')[3];
	var pos = document.getELementsByTagName('ul')[0];
	pos.removeChild(delEl);
	```

5. **특성 노드**
   ``` javascript
   // class 속성 가져오기
   document.getElementById('one').getAttribute('class');

   // 클래스 속성 수정
   document.getElementById('one').setAttribute('class', 'myClass');

   // 속성 여부 검사
   if(document.getElementById('one').hasAttribute('class')){
	   ...
   }

   // 속성 삭제
   document.getElementById('one').removeAttribute('class');s
   ```