# 1장. 오늘날의 보안

### 1.1 스프링 시큐리티: 개념과 장점

- 스프링 시큐리티
    - 스프링 애플리케이션에 보안을 적용하는 과정을 크게 간소화하는 프레임워크
    - 스프링 애플리케이션에 애플리케이션 수준의 보안을 구현하는 사실상의 표준
        - **인증**, **권한** 부여 및 일반적인 공격에 대한 **방어**를 구현하는 세부적인 맞춤 구성 방법을 제공
        - 아파치 2.0 라이선스에 따라 릴리즈되는 오픈 소스 소프트웨어
        - ‘스프링’의 방식(어노테이션, 빈, SpEL(Spring Expression Language))으로 애플리케이션 수준 보안을 적용 가능
        - 스프링 시큐리티를 이해하고 올바르게 이용하는 것은 개발자의 책임
    - 대안
        - 아파치 시로

### 1.2 소프트웨어 보안이란?

- 보안
    - 오늘날의 소프트웨어 시스템은 상당 부분이 민감한 정보일 수 있는 대량의 데이터를 관리중
        - 사용자가 개인적이라고 생각하는 모든 정보는 민감 정보(전화번호, 이메일, 신용 카드 정보)
    - 애플리케이션은 이러한 민감 정보에 접근, 변경 또는 가로챌 기회가 없게 해야 하며 의도된 사용자 이외의 대상은 어떤 식으로든 데이터와 상호 작용할 수 없게 해야 함
    - 보안은 계층별로 적용해야 하며, 계층마다 다른 접근 방식이 필요함
        - 스프링 시큐리티는 애플리케이션 수준 보안에 속하는 프레임워크
        - 한 계층의 보안 문제를 해결할 때는 되도록 위 계층이 존재하지 않는다고 가정하는 것이 바람직
- 애플리케이션 수준 보안
    - 애플리케이션이 실행되는 환경과 애플리케이션이 처리하고 저장하는 데이터를 보호하기 위해 해야 하는 모든 것
- 인증
    - 애플리케이션이 사용자(사람 또는 애플리케이션)를 식별하는 방법 → 누가 요청하는 거지?
- 권한 부여
    - 사용자가 무엇을 하도록 허용해야 하는지 결정하는 것
        - 대부분의 애플리케이션은 사용자가 특정 기능에 대한 접근 권한을 얻는데 제한이 있음

### 1.3 보안이 중요한 이유는 무엇인가?

- 사용자의 관점에서 보라
    - 우리 모두 데이터에 접근하는 애플리케이션을 이용 중(데이터를 변경, 이용, 노출)
    - 사람마다 데이터에 대한 민감도는 제각각(이메일 유출은 그리 심각 X, 은행 계좌의 경우 보다 민감)
        - 애플리케이션은 모든 데이터를 사용자가 원하는 접근 수준까지 보호해야 한다
- 객관적인 관점
    - 보안의 취약으로 인해 금전적 비용을 지불하거나 수익성 손실을 초래할 수 있음
        - 애플리케이션에 대한 사용자의 신뢰는 소중한 자사 중 하나
        - 만약 브랜드와 회사의 이미지가 손상된다면 이를 회복하는 비용이 시스템의 취약성 때문에 발생한 손해보다 더 클 수도 있음

### 1.4 웹 애플리케이션의 일반적인 보안 취약성

- 공격자는 공격을 시작하기 전에 애플리케이션의 취약성(vulnerability) 파악
    - 취약성
        - 악의적 의도를 가지고 이용할 수 있는 애플리케이션의 약점
- OWASP(https://www.owasp.org)
    - 애플리케이션을 개발할 때 피해야 하는 가장 일반적인 취약성에 대한 풍부한 자료가 있는 오픈 웹 애플리케이션 보안 프로젝트

**1.4.1 인증과 권한 부여의 취약성**

- 인증(Authentication)
    - 애플리케이션이 이를 이용하려는 사람을 식별하는 프로세스
- 권한 부여(Authorization)
    - 인증된 호출자가 특정 기능과 데이터에 대한 이용 권리가 있는지 확인하는 프로세스
- 인증 취약성
    - 사용자가 악의를 가지고 다른 사람의 기능이나 데이터에 접근 가능
- 예시
    - 권한을 가진 사용자만 특정 엔드포인트에 접근하도록 정의
    - 데이터 수준에 제한이 없으면 다른 사용자의 데이터를 이용할 수 있다는 허점

**1.4.2 세션 고정이란?**

- 세션 고정(Session Fixation)
    - 애플리케이션이 인증 프로세스 중에 고유한 세션 ID를 할당하지 않아 기존 세션 ID가 재사용될 가능성 존재
    - 이미 생성된 세션 ID를 재이용해 유효한 사용자를 가장

**1.4.3 XSS(교차 사이트 스트립팅)란?**

- XSS(Cross Site Scripting)
    - 서버에 노출된 웹 서비스로 클라이언트 쪽 스크립트를 주입해 다른 사용자가 이를 실행하도록 하는 공격
    - 원치 않는 외래 스크립트의 실행 방지를 위해 이용 전이나 저장 전에 요청을 적절하게 ‘소독’하는 과정 필요

**1.4.4 CSRF(사이트 간 요청 위조)란?**

- CSRF(사이트 간 요청 위조)
    - 특정 서버에서 작업을 호출하는 URL을 추출, 애플리케이션 외부에서 재사용
    - 서버가 요청의 출처를 확인하지 않고 무턱대고 실행하면 다른 모든 곳에서 요청이 실행 가능
        - 일반적으로 공격자는 CSRF를 이용해 시스템의 데이터를 변경하는 동작을 실행

**1.4.5 웹 애플리케이션의 주입 취약성 이해**

- 주입(Injection) 공격
    - 시스템에 특정 데이터를 유입하는 취약성 공격 → 피해 시스템의 데이터 변경, 삭제, 무단 이용을 유발
        - XSS도 주입 취약성의 하나로 볼 수 있음
    - SQL 주입
        - SQL 쿼리를 변경하거나 실행해서 시스템의 데이터를 변경, 삭제 또는 추출
        - OS 명령을 실행해서 전체 시스템을 손상시킬 수도 있음

**1.4.6 민감한 데이터의 노출 처리하기**

- 스프링 시큐리티는 자격 증명과 개인 키를 다룸
    - 중요한 데이터는 구성 파일이 아닌 볼트에 넣어야 한다
        - 실제 개발된 시스템에서는 모든 환경에서 민감한 키 값을 볼 수 없어야 함
        - 운영 환경에서는 소수의 사람만 개인 데이터에 접근할 수 있어야 함
            - [application.properties](http://application.properties) 또는 application.yaml에 설정하면 소스 코드를 볼 수 있는 모든 사람이 값에 접근할 수 있다
            - 버전 관리 시스템에도 값의 변경 기록이 저장되는 것들 볼 수 있다
    - 콘솔에 기록하거나 스플렁크나(Splunk)나 엘라스틱서치(EleasticSearch)와 같이 데이터베이스에 저장하는 로그 정보도 민감한 데이터의 노출과 관련이 있다
        - 공개 정보가 아닌 것은 절대 로그에 기록하지 말아야 한다!
    - 애플리케이션에 예외가 발생했을 때 서버가 클라이언트에게 반환하는 정보에 주의할 필요가 있다
        - 잘못된 요청을 처리하는 과정에서 애플리케이션이 너무 많은 세부 정보를 반환해서 구현을 노출하는 경우가 있음

**1.4.7 메서드 접근 제어 부족이란?**

- 애플리케이션 수준에도 한 계층에만 권한 부여를 적용하면 안됨
    - 개발자가 컨트롤러 계층에만 권한 부여를 적용하면 리포지토리는 사용자에 대해 모르고 데이터 검색을 제한하지 X
        - 인증되지 않은 계정으로 요청해도 리포지토리는 이를 반환
    - 만약 동일한 리포지토리를 이용하는 다른 기능을 추가한다면 추가된 컨트롤러에 다시 권한 부여 구성을 적용해야 한다
        - 이럴 경우네에는 권한 부여 구성을 리포지토리가 있는 곳으로 옮기는 것이 더 좋다

**1.4.8 알려진 취약성이 있는 종속성 이용**

- 때로는 개발하는 애플리케이션이 아니라 기능을 만들기 위해 이용하는 라이브러리나 프레임워크 같은 종속성에 취약성이 있을 수 있다
    - 항상 종속성을 주의깊게 살펴보고 취약성이 있는 버전은 제거해야!
    - 메이븐 또는 그레이들 구성에 플러그인을 추가하면 신속하게 정적 분석을 수행 가능

### 1.5 다양한 아키텍처에 적용된 보안

- 시스템의 디자인에 맞는 보안 관행을 적용하는 방법
    - 소프트웨어 아키텍처가 서로 다르면 가능한 유출과 취약성도 서로 다름

**1.5.1 일체형 웹 애플리케이션 설계**

- 백엔드와 프런트엔드 개발 간의 직접적인 분리 X
- 애플리케이션이 HTTP 요청을 수신하고 HTTP 응답을 클라이언트에게 보내는 일반 서블릿 흐름
    - 세션 고정 취약성과 앞서 언급한 CSRF 가능성을 고려해야 하고 HTTP 세션에 저장하는 정보도 고려해야 함
        - CSRF 방지 토큰을 이용해야
    - 서버 쪽 세션은 준 영구적이며 데이터의 상태를 저장하므로 수명이 더 길다
    - 힙 덤프에 접근할 수 있는 사람은 앱의 내부 메모리에 있는 정보를 읽을 수 있음
        - 스프링 액추에이터를 사용해 엔드포인트 호출만으로도 힙 덤프를 반환할 수 있음

**1.5.2 백엔드/프론트엔드 분리를 위한 보안 설계**

- 앵귤러나 리액트, 또는 Vue.js 같은 프레임워크를 이용해 프론트엔드를 개발
- REST 엔드포인트를 통해 백엔드와 통신
- 서버 쪽 세션을 줄이고 클라이언트 쪽 세션으로 대체하는 것이 좋음

**1.5.3 OAuth2 흐름 이해**

- OAuth2 프레임워크
    - 권한 부여 서버와 리소스 서버라는 두 가지 별도의 엔티티를 정의
    - 권한 부여 서버
        - 사용자에게 권한을 부여하고 이용 권리 집합을 지정하는 토큰을 제공
    - 리소스 서버
        - 권한 부여를 수행한 후 획득한 토큰에 따라 호출이 허용되거나 거부
        - 호출할 수 있는 엔드포인트는 보호된 리소스

**1.5.4 API 키, 암호화 서명, IP 검증을 이용해 요청 보안**

- 백엔드를 서비스 그룹으로 배포하거나 시스템 외부의 다른 백엔드를 이용하는 경우
- 구성 요소 간의 메시지를 어떤 방식으로든 검증
    - 요청 및 응답 헤더에 정적 키 이용
        - 요청과 응답의 헤더에 키를 이용
        - 헤더 값이 잘못되면 요청과 응답이 수락되지 않는다
        - 트래픽이 데이터센터 외부로 이동하면 가로채기 쉽다
        - 키 값을 얻으면 엔드포인트에 대한 호출을 재현할 수 있다
        - 이 접근법을 이용할 때는 IP 주소 허용 목록을 함께 결합한다
    - 암호화 서명으로 요청 및 응답 서명
        - 키로 요청과 응답에 서명 → 키를 보낼 필요가 없다
        - 상대는 자신의 키로 서명을 검증
        - 두 개의 비대칭 키 쌍을 이용 → 개인 키를 교환하지 않는다
        - 서명을 계산하는데 더 많은 리소스가 소비된다
    - IP 주소에 검증 적용

### 1.6 이 책에서 배울 내용

- 스프링 시큐리티의 아키텍처와 기본 구성 요소 및 이를 이용해 애플리케이션을 보호하는 방법
- OAuth2 및 OpenID Connect 흐름을 비롯해 시프링 시큐리티로 인증과 권한을 부여하는 방법
- 운영 준비 단계의 애플리케이션에 이를 적용하는 방법
- 애플리케이션의 다양한 계층에서 스프링 시큐리티로 보안을 구현하는 방법
- 다양한 구성 스타일과 프로젝트에 맞는 모범 사례
- ~~리액티브 애플리케이션에 스프링 시큐리티 이용~~
- 보안 구현 테스트

### 요약

- 스프링 애플리케이션을 보호하기 위한 사실상의 표준
    - 다양한 스타일과 아키텍처에 적용할 수 있는 갖가지 대안을 제공
- 시스템의 계층별로 보안 적용
    - 계층별로 다른 관행을 이용해야 함
- 보안은 소프트웨어 프로젝트를 시작할 때부터 고려해야 하는 공통 관심사
- 취약성을 예방하기 위한 투자 비용보다 공격의 대가가 훨씬 크다
- 오픈 웹 애플리케이션 보안 프로젝트(OWASP)는 취약성과 보안 관련 사항을 참고할 수 있는 훌륭한 장소
- 로그나 오류 메시지에 민감한 데이터를 노출하는 등의 사소한 실수가 애플리케이션의 취약성을 만든다
