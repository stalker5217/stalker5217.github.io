---
title: '[Spring] 의존성 주입'
toc: true
toc-stick: true
search: true
categories:
  - spring
tags:
  - spring
  - java
  - dependency injection
excerpt: '의존성 주입에 대하여 알아봅시다'
---

## 의존 관계  

의존 관계를 UML에서는 아래와 같이 표기한다.

![dependency](/assets/images/spring/dependency.png)  

그림 그대로 A가 B를 사용, 즉 A가 B에 의존하고 있음을 나타낸다. 
A가 B에 의존하고 있다는 것은 B의 내용이 변하면 그대로 그것이 A에 영향을 미친다는 의미이다. 
대표적으로 A에서 B의 메소드를 호출하고 있다면, 
B의 메소드 호출 형식이나 내부 기능이 변한다면 A의 구현 또는 기능이 변할 수도 있다.

![dependency-interface](/assets/images/spring/dependency-interface.png)  

위와 같이 구성하면 결합도가 낮은 좀 더 유연한 구현을 할 수 있다. 
**A는 인터페이스 B를 사용하고 있으므로 인터페이스 자체가 수정되는 것이 아닌 이상 B의 구현체들의 내용이 변해도 그 영향을 덜 받는다고 할 수 있다.**    

하지만 이는 모델링 관점에서의 의존 관계이고 실제 런타임에서 어떤 오브젝트와 의존 관계를 맺는 것은 다른 이야기이다. 
설계 상에는 드러나지 않지만 런타임에서 A와 실제로 관계를 맺는 대상을 **의존 오브젝트(Dependent Object)**라고 한다.
**의존 관계 주입**은 이처럼 구체적인 의존 오브젝트와 그것을 사용하는 클라이언트 오브젝트를 연결해주는 작업을 말한다.  

정리하자만 의존 관계 주입은 다음 세 가지 조건을 충족한다.
1. 클래스 모델이나 코드에는 런타임 시점의 의존 관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
2. 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3의 존재가 결정한다.
3. 의존 관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.  

## 의존 관계 검색과 주입  

**의존 관계 주입(Dependency Injection)**은 생성자나 기타 메소드의 파라미터를 통해 레퍼런스를 전달하는 방식으로 주로 구현 된다.

``` java
public class UserDao{
	private final ConnectionMaker connectionMaker;

	public UserDao(ConnectionMaker connectionMaker){
		this.connectionMaker = connectionMaker;
	}
}
```

**의존 관계 검색(Dependency Lookup)**은 자신이 필요로 하는 의존 오브젝트를 능동적으로 찾는다. 
실제 오브젝트의 생성은 외부에게 맡기지만, 이를 가져올 때는 컨테이너에게 직접 요청하는 방법을 사용한다.

``` java
class UserDao{
	private final ConnectionMaker connectionMaker;

	public UserDao() {
		// Case 1. Factory를 직접 사용
		DaoFactory daoFactory = new DaoFactory();
		this.connectionMaker = daoFactory.connectionMaker();
	}
}
```

``` java
class UserDao{
	private final ConnectionMaker connectionMaker;

	public UserDao() {
		// Case 2. Spring Bean 사용
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
	}
}
```  

의존 관계 주입의 코드가 확실히 더 깔끔하다. 검색 방식은 Dao 내에서 Factory나 Spring API가 나타나고 이는 확실히 어색하다. 
하지만 검색 형태의 방식도 반드시 필요하다. 서버의 구동이나 사용자의 요청을 처리할 때는 어쨋거나 한 번은 검색을 통해 읽어와야 하기 때문이다.  

의존관계 검색과 의존관계 주입은 또 큰 차이가 하나 있다. 의존관계 검색에서는 검색하는 오브젝트 자신은 스프링 Bean일 필요가 없다는 점이다. 
하지만 의존관계 주입에서는 UserDao와 ConnectionMaker 사이에 DI가 적용되려면 UserDao도 반드시 컨테이너가 생성하는 Bean이여야 한다. 
컨테이너가 ConnectionMaker 오브젝트를 주입하기 위해서는 UserDao에 대한 생성과 초기화 권한을 가지고 있어야하기 때문이다. 
DI를 받는 오브젝트는 자기가 먼저 컨테이너가 관리하는 빈이 돼야하는 사실을 잊지 말자. 

<br/>  

참고
- 이일민, 토비의 스프링 3.1