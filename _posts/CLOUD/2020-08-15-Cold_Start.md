---
title: '[GCP] Cloud Functions Cold Start'
toc: true
toc-stick: true
search: true
categories:
  - Cloud
tags:
  - Cloud
  - Serverless Function
  - Cold Start
  - Cold Boot Time
  - Cloud Functions
  - GCP
  - Google Cloud Platform
  - AWS Lambda
excerpt: 'Cold Start에 대해 알아봅시다.'
last_modified_at: '2020-07-13'
---

Dialogflow의 비즈니스 로직을 처리하기 위해 Webhook을 사용하였고, GCP의 Cloud functions을 사용하여 구성하였다.  

하지만 때때로, Webhook Request가 지정된 5000ms를 넘어 타임아웃이 발생하는 경우가 있었고
이는 함수를 Deploy한 직후에 자주 관찰되었다.


## Cold Start  

Serverless function의 특징은 Auto Provisioning과 Auto Scaling으로 트래픽에 맞게 
함수 제공 인스턴스 수를 알아서 조정해준다.

> GCP Cloud Functions의 인스턴스가 시작되는 조건은 다음과 같다.
> 1. 함수를 배포하는 경우
> 2. 트래픽을 처리하기 위해 인스턴스가 확장되거나, 기존 인스턴스를 대체하는 경우
 

여기서 발생한는 문제가 **콜드 스타트** 개념이다. 
새로운 인스턴스가 시작될 때는 함수가 실행될 수 있게 소스 코드 로드 등 여러 동작 환경을 구성하는 작업이 포함된다. 
그래서 인스턴스를 처음 호출할 때에는 꽤나 느리게 동작한다.  

콜드 스타트 시간을 줄이기 위한 비교를 간단히 정리해 봤는데, 소요 시간의 대소 관계는 아래와 같다.

-----------

- **Cloud Vendor**   
AWS Lambda < Cloud Functions < Azure Functions

- **Language**  
Node < Python < Go < Java

- **Instance Memory**  
함수 인스턴스의 메모리 설정이 높을수록 짧음

-----------

## Functions Optimization  

콜드 스타트 시간 외에도 함수 동작 시간을 조금이라도 줄이기 위한, 최적화 방법을 알아본다.

1. **불필요한 dependency 제거**    

	사용하지 않거나 불필요한 dependency라면 제거하여 배포 시간을 줄인다.

2. **전역 변수를 사용한 재사용**  

	Cloud functions은 기본적으로 Stateless이다. 
	하지만 같은 인스턴스가 호출된다면 종종 이전의 실행 환경을 재사용한다.  
	이런 특성을 살려 서로 다른 호출간에 공통적으로 사용하는 값들,
	특히, 네트워크 연결, 라이브러리 참조나 API Client 객체 등은 전역 변수로 올려 캐싱하면 성능이 크게 향상된다.

	``` js
	console.log('Global scope');
	const perInstance = heavyComputation();
	const functions = require('firebase-functions');

	exports.function = functions.https.onRequest((req, res) => {
		console.log('Function invocation');
		const perFunction = lightweightComputation();

		res.send(`Per instance: ${perInstance}, per function: ${perFunction}`);
	});
	```

3. **Lazy Initialization**  

	전역 변수를 사용하되 초기화도 전역 범위에서 하는 것은 최초 실행 시에는 latency를 줄여주지 못한다. 
	일부 객체가 모든 코드에서 사용하지 않는 경우에는, 실제 해당 객체가 사용될 때 초기화를 시켜준다.

	``` js
	// Always initialized (at cold-start)
	const nonLazyGlobal = fileWideComputation();

	// Declared at cold-start, but only initialized if/when the function executes
	let lazyGlobal;

	/**
	* HTTP function that uses lazy-initialized globals
	*
	* @param {Object} req request context.
	* @param {Object} res response context.
	*/
	exports.lazyGlobals = (req, res) => {
		// This value is initialized only if (and when) the function is called
		lazyGlobal = lazyGlobal || functionSpecificComputation();

		res.send(`Lazy global: ${lazyGlobal}, non-lazy global: ${nonLazyGlobal}`);
	};
	```

4. **Keep Alive**

	HTTP 연결은 Stateless이기 때문에 모든 호출마다 커넥션을 생성한다. 
	이런 네트워크에 드는 비용을 줄이기 위해 keep-alive 설정을 활용한다.

	``` js
	const axios = require('axios');

	const http = require('http');
	const https = require('https');

	const httpAgent = new http.Agent({keepAlive: true});
	const httpsAgent = new https.Agent({keepAlive: true});

	/**
	* HTTP Cloud Function that caches an HTTP agent to pool HTTP connections.
	*
	* @param {Object} req Cloud Function request context.
	* @param {Object} res Cloud Function response context.
	*/
	exports.connectionPooling = async (req, res) => {
		try {
			const {data} = await axios.get('/', {httpAgent, httpsAgent});
			res.status(200).send(`Data: ${data}`);
		} catch (err) {
			res.status(500).send(`Error: ${err.message}`);
		}
	};
	```


<br/>

참고
- https://cloud.google.com/functions/docs