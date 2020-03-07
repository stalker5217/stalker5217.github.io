---
title: '[Javascript] HTML 콘텐츠 수정'
excerpt: "document.wirte(), element.innerHTML, DOM 조작"
search: true
categories:
  - javascript
tags:
  - javascript
  - js
---

## HTML Contents 수정 기법

1. ```element.innerHTML```
 - DOM 조작 방식에 비해 더 빠르게 동작한다
 - 이벤트 핸들러가 의도한 대로 동작하지 않을 수 있다
 - **보안 이슈가 있다**
2. ```DOM 조작 방식```
 - 이벤트 핸들러에 아무 영향을 미치지 않는다
 - innerHTML 방식에 비해 느리게 동작한다

 ``` html
 <!-- 수정 대상-->
 <div id='myDiv'>
	<p id='myP'>
		라이언
	</p>
</div>

<!-- 수정 후 포맷-->
<div id='myDiv'>
	어피치
</div>
```

``` javascript
// 1. element.innerHTML
document.getElementById('myDiv').innerHTML = '어피치';

// 2. DOM 조작 방식
document.getElementById('myP').remove();
document.getElementById('myDiv').append('어피치');
```

## XSS(Cross-Site Scripting Attacks) 
