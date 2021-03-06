---
layout: post
title:  "OAuth2 기본 개념정리"
date:   2019-01-05 00:18:23 +0700
categories: [oauth2]
---
기존 프로젝트에서 사용하는 Spring 버전을 4 에서 5로 업데이트를 하면서, Spring Security 관련 코드 수정에서 어려움을 겪었다. 
OAuth2 관련된 코드를 [WebFlux(Reactive Web)](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) 에 맞게 변경하는데, 코드의 플로우와 사용하는 용어들이 상당히 낯설었다. 이참에 웹 분야의 인증에 어느 정도 개념을 잡고가야겠다는 생각을 했고, 이제껏 내가 공부한 OAuth2 의 개념에 대해서 첫 번째 포스트로 등록하기로 했다. (#some-markdown-heading)
[create an anchor](#anchors-in-markdown)
# 배경

## SaaS(Software as a Service) 
 - **Twitter, FaceBook, Google** 과 같은 거대 IT 기업들은 자신의 SNS 플랫폼에서 제공하는 서비스를 다른 소프트웨어에서도 사용할 수 있는 형태로 제공하는 상호협력 가능한 비즈니스 모델을 만들었다. 이 비즈니스 모델은 업계의 사실상 표준(**De Facto**)이 됬다. 우리가 사용하는 대다수의 모바일 앱들을 보면 구글, 페이스북, 혹은 네이버와 같은 서비스 플랫폼을 통해서 **인증(Authentication)** 을 하고, **허가(Authorization)** 받은 해당 플랫폼이 제공하는 서비스를 활용한다.
몇몇 예를 들자면, 달리기앱에서 오늘 달린 거리를 페이스북이나 트위터에 바로 올려서 친구들한테 나의 기록을 보여준다. 소개팅 앱에서 페이스북에서 설정한 개인정보(학교, 취미, 관심분야)를 이용해, 적절한 상대를 매칭해주거나 친구목록을 보고 매칭 상대를 제한하기도 한다. 

## 표준화의 필요성 및 OAuth2 의 등장.
 - SaaS 에 등장으로 여러 플랫폼사들은 자신의 서비스 이용을 위한 표준화된 방법이 필요하다는 결정을 하게 된다. 표준 기술체를 만듬으로써, SaaS 를 구현함에 발생하는 기술 이슈를 빠르게 해결하고, 플랫폼 서비스를 사용하는 클라이언트가 구현에 발생하는 비용을 줄일 수 있다. 
 SNS 플랫폼 등장 초기에 이 **인증 / 허가** 프로세스에 대한 표준화된 프로토콜이 대두되었고, 표준화된 프로토콜이 2007년에 [OAuth 1.0 - RFC 5849](https://tools.ietf.org/html/rfc5849) 로 등장했다. 이후, 스마트폰 시장이 폭발적으로 성장하면서 OAuth 프로토콜의 성능의 한계점이 드러났다. 
 
 > 가벼운 인증 절차, 테스트하기 쉬운 환경, 분산 환경.
 
 - 모바일 시장의 폭발적인 수요를 맞추기 위해서는 위 3 가지를 지원할 수 있어야 했다. OAuth2 는 위 3가지 사항을 충족하는 형태로 설계됬고, 2012년에 [OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749) 로 등장했다. 그리고 대다수의 IT 서비스 기업들 OAuth2를 공식 인증 방법으로 채택했다. 


# Oauth2 핵심 용어(ROLE) 정리
- **Resource Owner**
  - 중요한 자원에 대한 권한을 부여할 수 있는 주체.
  - 해당 주체가 사람일 경우, end-user 라고함.
  - 여기서 의미하는 **중요한 자원**은 보통 Twitter, FaceBook 와 같은 서비스 제공자가 지닌 end-user 의 개인 정보를 의미한다. 
- **Resource Server**
  - 중요 자원(개인정보)을 소유하고, 이 자원에 대한 요청에 대해 응답을 할 수 있는 서버이다.
  - 자원에 대한 요청은 반드시 **access token** 으로 요청한다. 
  - access token 으로 자원을 요청하는 것. 즉, 아이디와 비밀번호에 대한 노출을 하지않는 것이 OAuth2 프로토콜의 핵심이다. 
  - 보통 Resource Server 는 FaceBook, Twitter 와 같은 서비스 제공자를 의미한다. 
- **Client**
  - Resource Owner 의 자원을 Resource Server 에게 요청하고 이 자원을 사용하는 주체. 어플리케이션(서버, 모바일앱, 데스크톱앱 .. 등등) 이다 
  - 공식 문서에서는 안정성에 따라 두 가지 타입으로 구분한다.
     - **Confidential**  : 자격성(client_secret value)을 안전하게 보관할 수 있음.
     - **Public** :  자격성을 안전하게 보관하기 힘듬. 
  - 타입에 따른 Client(어플리케이션) 종류
     - 웹 어플리케이션(Confidential)
       - Resource Owner 들은 HTML 유저인터페이스(웹브라우저)를 통해서 Client 에 접근하는 형태.
     - 유저-에이전트 기반 어프리케이션(Public)
       - 웹 서버로 부터 클라이언트 코드를 다운받아서, 어플리케이션 로직이 유저-에이전트(브라우저) 위에서 실행되는 형태이다. 
       - 브라우저로 코드가 노출되다보니 소유, 프로토콜 데이터 및 자격 증명에 대해 쉽게 접근이 가능하다. 
     - 네이티브 어플리케이션(Public or Confidential)
       - Resource Owner에 의해 디바이스에 설치되고, 실행되는 형태의 어플리케이션이다. 예를 들면 스마트폰 앱을 들 수 있다. 
       - 클라이언트 자격 증명이 어플리케이션에 포함될 수 있고, 추출해내기도 쉬울 수 있지만, 상황에 따라서 자격 증명 내용을 안전하게 보호받을 수 있음. ( 예 : 트위터 앱, 페이스북 앱)  
- **Authorization Server**
  - Client가 Resource Owner 에 대한 인증과 권한을 획득한 뒤에, Access Token 을 Client 에 발급해주는 서버이다. 
  - Authorization Server 와 Resource Server 간의 통신에 대해서는 구현하는 측마다 상이하다.
  - 가능한 형상
    - Authorization Server 와 Resource Server 가 동일 서버
    - 한 대의 Authorization Server 복수대의 Resource Server
    
# OAuth2 의 대략적인 Flow
 1. Client 가 Resource Owner에게 권한을 요청함. 권한 요청은 직접 Resource Owner 를 통하거나 혹은 Authorization Server 를 매개체로 비간접적으로 만들어진다. 후자가 좀 더 안전하다.
    * 보통은 Authroization Server 에서 제공하는 권한 요청 페이지 서비스를 활용함. 
 2. Client 는 Authorization Server 혹은 Resource Owner 로부터 Resource Owner 가 허가한 권한 수여(Authorization Grant)를 받는다. 권한 수여는 뒤에 나올 4가지의 OAuth2 의 Grant Type 방법 혹은 확장된 방법으로 표현된다. Authorization Grant Type 은 Client가 권한 요청하는 방법 그리고 Authorization Server 가 지원하는 방법에 의해 결정이 된다.   
 3. Client 는 Access Token 을 Authorization Server 에 인증(Authentication)과 허가받은 권한(Authorization Grant)과 함께 요청한다.    
 4. Authorization Server 는 Client 증명을 하고 권한에 대한 유효성을 확인해서, 유효하면 AccessToken 을 발급해준다. 
    
# OAuth2 Grant Type 에 따른 구체적인 플로우 - Use Case
 OAuth2 구현은 Grant Type (권한 수여 방법) 에 따라서 달라진다. 
* Authorization Code
   - 제 3자 인증 방식에서 쓰는 방법이다. 우리가 흔히보는 Facebook, Twitter, Naver, Kakao 인증을 하는 앱들이 이 방식을 쓴다.
   - Client 가 Confidential 할 때 쓰는 방법이다.  
   - Flow
     1. 사전에 Authorization Server 에 Client 를 등록한다. Client 의 id, scope, secret(검증 수단), redirection uri(Client가 정함) 을 발급한다. 
     2. Client 가 Resource Owner 의 User-Agent 와 Authorization Endpoint(보통 인증 폼 화면 - 카카오, 트위터, 페이스북 인증창을 띄움) 를 연결함으로써, flow 를 시작한다.
     3. User-Agent 를 통해 Authorizaiton Server 는 Resource Owner 를 인증하고, User-Agent로 하여금 Client 요청에 대한 권한 수여 혹은 거절 절차를 진행한다. 
     4. 권한이 수여된다면, Authorization Server 는 1번에서 발급한 redirection uri 를 User-Agent 에게 내려주어서 Client 와 다시 남은 작업을 진행하도록 context를 넘긴다. redirection uri 에는 Authorization Code 와 Local State 가 포함되어있다. 
     5. User-Agent 는 redirection uri 를 통해 Client 에게 Authorization Code 를 전달한다. Client 는 Authorization Code 를 받아서 Authroization Server 에게 redirection uri를 포함시켜, AccessToken 을 요청한다. 
     6. Authorization Server 는 redirection uri 가 1번에서 만든 redirection uri 와의 동일성 검사와 Authroization Code 의 유효성을 검증해서, 합격하면 Access Token 과 경우에 따라서는 Refresh Token 을 발급해준다. 
* Implicit
  - Authorization Code 와 비슷하지만 두 가지 차이점이 있다. 첫째는 User-Agent 기반 클라이언트(e.g. 싱글페이지웹 어플리케이션) 에 주로 사용되는 점이다. 이것은 Client secret 보관에 취약하다. 둘째는 Authorization Server 가 Authorization Code 대신에, 직접 AccessToken 을 User-Agent 에게 반환한다. 
  - Flow
    1. 사전에 Authorization Server 에 Client 를 등록한다. Client 의 id, scope, secret(검증 수단), redirection uri(Client가 정함) 을 발급한다. 
    2. Client 가 Resource Owner 의 User-Agent 와 Authorization Endpoint(보통 인증 폼 화면 - 카카오, 트위터, 페이스북 인증창을 띄움) 를 연결함으로써, flow 를 시작한다.
    3. User-Agent 를 통해 Authorizaiton Server 는 Resource Owner 를 인증하고, User-Agent로 하여금 Client 요청에 대한 권한 수여 혹은 거절 절차를 진행한다. 
    4. 권한이 수여된다면, Authorization 서버는 발급한 redirection uri 에 access token을 encoding하고 uri fragment에 포함시켜, User-Agent 에 내려준다. 
    5. User-Agent 는 redirection flow에 따라 웹호스트 Client 리소스에 요청을 만든다. 이때, uri fragment(인코딩된 Access Token)을 따로 떼어내서 내부적으로 보관하고 있다. 
    6. 웹호스트 client 리소스는 웹페이지(HTML Document 와 내장된 스크립트)를 반환한다. 이 웹페이지는 full redirection URI 에 접근 가능하고, AccessToken 을 추출할 수 있다.
    7. User-Agent 는 웹호스팅된 Client 가 제공한 스크립트를 내부적으로 실행해서, Access Token 을 추출한다. 
    8. User-Agent 는 Access Token 을 Client 에 넘긴다. 
* Client Credential
  - 가장 심플한 권한 수여 방식이다. 보통 machine to machine 인증에 적합하다.
  - Client 가 자신의 자격 증명 정보를 가지고 Authorization Server Access Token 을 요청하고 받는다. 
  - Flow
    1. Client 가 Authorization Server 에 인증을 하고, Access Token 을 요청한다.
    2. Authorization Server는 Client 인증을 하고, 유효하면 Access Token 을 발급한다. 
* Password
  - Confidential 한 Client 가 resource Owner 의 자격증명 정보(Credential, e.g. "id", "password") 를 가지고 있어서, Resource Server 랑 직접 통신하는 방식이다. 보통 한 회사가 Client 와 Resource Server, Authorization Server 를 제공하는 경우에 이같은 형상을 가짐. 혹은 서버간 api 요청에서도 쓴다.
  - Flow
    1. Resource Owner 가 Client 에 id 와 password 를 제공한다.
    2. Client 가 Authorization Server 에 Resource Owner 의 자격증명 정보를 포함해서 AccessToken을 요청한다.
    3. Authorization Server 는 Client 를 인증하고, Resource Owner 의 자격증명정보의 유효성을 검증한다. 조건이 충족되면 AccessToken 을 발급한다. 
    
# OAuth2 Grant Type 을 정하는 의사결정방법.
  1. 먼저 Access Token Owner 가 누구인지 확인한다. 
    - Machine 일 경우 (User Resource 를 사용하지 않음.) Grant Type은 *Client Credential* 이 된다. 
    - 그 외에는 아래 2번의 결정 방법에 따라 진행한다. 
  2. Client 의 타입과 각 Client 의 보안레벨로 결정한다. 
     - Web App
       - 보안이 안전하므로, *Authorization Code* Grant Type 을 사용한다.
     - Native App
       - First Party 일 경우.
         - *Implicit* Grant Type 을 사용한다.
       - Third Party 일 경우.
         - *Authorization Code*  Grant Type 을 사용한다.
     - User-Agent Based App
       - First Party 일 경우.
         - *Password* Grant Type 을 사용한다. 
       - Thrid Party 일 경우.
         - *Implicit*  Grant Type 을 사용한다. 
     - 여기서 First Party, Third Party 는 Resource 를 제공하는 개발사에서 만든 App 인 경우, Third Party 를 아니면, First Party 를 의미한다.
       - 예) Twitter, Facebook native app 은 Third Party App 이다. 
# 결론
  - OAuth2는 Resource Owner 의 자격증명(Credential) 정보 없이 Client 가 Resource 를 Resource Server 로부터 받도록 했다.
  - 이를 가능케한 핵심은 Resource 제공 측에서 Access Token 을 발급해서, 그 Token 으로 자격성을 검증하는 것이다.
  - Access Token 을 발급하는 과정은 Client 의 보안 정도에 따라서 달라진다. 
  - 다음 포스트에서는 OAuth2 의 Access Token 에 대해서 좀 더 자세하게 다루겠다. 

## Reference
  - [A Guide To OAuth 2.0 Grants](https://alexbilbie.com/guide-to-oauth-2-grants/)
  - [OAuth2-IETF(RFC6749)](https://tools.ietf.org/html/rfc6749)

