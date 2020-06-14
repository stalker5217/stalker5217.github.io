---
title: '[Javascript] HTML 콘텐츠 수정'
toc: true
toc-stick: true
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
XSS Attack은 웹 상에서 간단한 하나의 공격 형태이다.
악의적인 목적으로 스크립트를 삽입하여 의도치 않은 동작을 일으키거나 정보를 탈취할 수 있다.

깊은 내용까지는 지금 다룰 수 없고, 기본적으로 다음과 같은 동작을 준수하도록 한다.

- **사용자 입력 콘텐츠 escape 처리**  
&, <, >, ", / 등의 문자들은 코드로 처리되는 것이 아니라 텍스트로 인식하게 처리한다.  

- **신뢰할 수 없는 출처의 데이터는 자바스크립트단에 담지 않는다**

- **innerHTML, jQuery의 html 속성 사용은 되도록 지양하도록 하나 사용시에는 주의를 한다.**  
반드시 사용자 입력을 이스케이프 처리하고, 텍스트로 인식되도록 한다.  


참고
- 장현희, 자바스크립트 & 제이쿼리 (인터랙티브 프론트엔드 웹 개발 교과서)