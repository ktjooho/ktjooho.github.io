---
layout: post
title:  "OAuth2 정리"
date:   2019-01-05 00:18:23 +0700
categories: [oauth2]
---
기존 프로젝트에서 사용하는 Spring 버전을 4 에서 5로 업데이트를 하면서, Spring Security 관련 코드 수정에서 어려움을 겪었다. 
OAuth2 관련된 코드를 [WebFlux(Reactive Web)](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) 에 맞게 변경하는데, 코드의 플로우와 사용하는 용어들이 상당히 낯설었다. 이참에 웹 분야의 인증에 어느 정도 개념을 잡고가야겠다는 생각을 했고, 이제껏 내가 공부한 OAuth2 의 개념에 대해서 첫 번째 포스트로 등록하기로 했다. 
# 배경
## SaaS(Software as a Service) 
 - **Twitter, FaceBook, Google** 과 같은 거대 IT 기업들은 자신의 SNS 플랫폼에서 제공하는 서비스를 다른 소프트웨어에서도 사용할 수 있는 형태로 제공하는 상호협력 가능한 비즈니스 모델을 만들었다. 이 비즈니스 모델은 업계의 사실상 표준(**De Facto**)이 됬다. 우리가 사용하는 대다수의 모바일 앱들을 보면 구글, 페이스북, 혹은 네이버와 같은 서비스 플랫폼을 통해서 **인증**을 하고, **허가**받은 해당 플랫폼이 제공하는 서비스를 활용한다.
몇몇 예를 들자면, 달리기앱에서 오늘 달린 거리를 페이스북이나 트위터에 바로 올려서 친구들한테 나의 기록을 보여준다. 소개팅 앱에서 페이스북에서 설정한 개인정보(학교, 취미, 관심분야)를 이용해, 적절한 상대를 매칭해주거나 친구목록을 보고 매칭 상대를 제한하기도 한다. 

## 표준화의 필요성 및 OAuth2 의 등장.
 - SaaS 에 등장으로 여러 플랫폼사들은 자신의 서비스 이용을 위한 표준화된 방법이 필요하다는 결정을 하게 된다. 표준 기술체를 만듬으로써, SaaS 를 구현함에 발생하는 기술적인 이슈를 빠르게 해결하고, 플랫폼 서비스를 사용하는 클라이언트가 구현에 발생하는 비용을 줄일 수 있다. 
 SNS 플랫폼 등장 초기에 이 **인증 / 허가** 프로세스에 대한 표준화된 프로토콜이 대두되었고, 표준화된 프로토콜이 2007년에 [OAuth 1.0 - RFC 5849](https://tools.ietf.org/html/rfc5849) 로 등장했다. 이후, 스마트폰 시장이 폭발적으로 성장하면서 OAuth 프로토콜의 성능적인 한계점이 드러났다. 
 
 > 가벼운 인증 절차, 테스트하기 쉬운 환경, 분산 환경.
 
 모바일 시장의 폭발적인 수요를 맞추기 위해서는 위 3 가지를 지원할 수 있어야 했다. OAuth2 는 위 3가지 사항을 충족하는 형태로 설계됬고, 2012년에 [OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749) 로 등장했다. 그리고 대다수의 IT 서비스 기업들 OAuth2를 공식 인증 방법으로 채택했다. 


# Oauth2 핵심 용어(ROLE) 정리
 [RFC 6749](https://tools.ietf.org/html/rfc6749) 문서에서 정의된 용어는 좀 더 범용적인 의미로 정의가 됬다. 해당 문서를 참조해서 용어를 정리하겠다. 
- Resource Owner
  - 중요한 자원에 대한 권한을 부여할 수 있는 주체.
  - 해당 주체가 사람일 경우, end-user 라고함.
  - 여기서 의미하는 **중요한 자원**은 보통 Twitter, FaceBook 에서 end-user 가 지닌 개인 정보를 의미한다. 

- Resource Server
  - 중요 자원(개인정보)을 소유하고, 이 자원에 대한 요청에 대해 응답을 할 수 있는 서버이다.
  - 자원에 대한 요청은 반드시 **access token** 으로 요청한다. 
  - access token 으로 자원을 요청하는 것. 즉, 아이디와 비밀번호에 대한 노출을 하지않는 것이 OAuth2 프로토콜의 핵심이다. 
  - 보통 Resource Server 는 FaceBook, Twitter 와 같은 서비스 제공자를 의미한다. 
  
- Client
  - Resource Owner 를 대신해서 중요 자원을 요청하고, 이것에 대한 허가를 받는 Application(서버, 모바일앱, 데스크톱앱 .. 등등) 이다 

- Authorization Server
  - Client가 Resource Owner 에 대한 인증과 권한을 획득한 뒤에, access token 을 Client 에 발급해주는 서버이다. 
  - Authorization Server 와 Resource Server 간의 통신에 대해서는 구현하는 측마다 상이하다.
  - 가능한 형상
    - Authorization Server 와 Resource Server 가 동일 서버
    - 한 대의 Authorization Server 복수대의 Resource Server
    
# OAuth2 Grant Type 에 따른 플로우 - Use Case
* Authorization Code
* Client Credential
* Password
* Implicit 
* Password

# 구현의 관점에서 보는 OAuth2 

# OAuth2 와 JWT 토큰
## 토큰 관리 방법

# 토큰 저장 방법 

## 결론

## Reference
 - [OAuth-Wikipedia]()
 - [OAuth2-IETF(RFC6749)](https://tools.ietf.org/html/rfc6749)
 - 

