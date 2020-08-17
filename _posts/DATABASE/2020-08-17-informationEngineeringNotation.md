---
title: '[Database] 데이터 모델링 - I/E 표기법'
toc: true
toc-stick: true
search: true
categories:
  - database
tags:
  - database
  - data modeling
  - I/E
  - Information Engineering Notation
excerpt: 'I/E 표기법을 정리합시다'
---

## Entity

Entity는 업무 정의에 필요한 객체의 최소 표현 정도로 볼 수 있다.  

- Identifier : PK. Entity를 유일하게 식별할 수 있는 속성
- Attribute : 그 외 Entity를 설명하기 위한 속성

<br/>

**I/E 표현**

![entity](/assets/images/database/entity.png)


## Relation

Entity 간의 관계를 정의한다.

**M:N 관계**

![ie_relation](/assets/images/database/ie_relation.png)

<br/>

**Identifying 관계**

![ie_relation2](/assets/images/database/ie_relation2.png)


<br/>

참고
- [DBGuide.Net - 데이터 모델링 표기법 이해](http://www.dbguide.net/db.db?cmd=view&boardUid=12845&boardConfigUid=9&boardIdx=31&boardStep=1)