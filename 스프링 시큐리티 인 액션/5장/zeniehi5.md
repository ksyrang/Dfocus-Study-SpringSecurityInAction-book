# 5장. 인증 구현

---

- AuthenticationProvider
    - 인증 논리를 담당
    - 인증 프로세스 결과
        - 인증 실패
            - 애플리케이션이 사용자를 인식하지 못해 요청을 거부
            - 클라이언트에 HTTP 401 권한 없음 응답을 반환
        - 인증 성공
            - 요청자의 세부 정보가 저장 → 애플리케이션이 이 정보를 권한 부여에 이용
            - 인증된 요청에 대한 세부 정보는 SecurityContext 인스턴스에 저장

![IMG_E1B841B9EAFF-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/c7567006-f5d7-48fb-a36b-4a9ed716bef4/IMG_E1B841B9EAFF-1.jpeg)

**5.1 AuthenticationProvider의 이해**

- 하나의 애플리케이션이 여러가지 서로 다른 방식의 인증을 구현할 수 있어야 함
    - 예시) 암호, 지문, SMS 문자 인증 등
- 어떠한 시나리오가 필요하더라도 이를 구현할 수 있게 해주는 것이 프레임워크의 목적!
    - 스프링 시큐리티는 AuthenticationProvider 인터페이스로 모든 맞춤형 인증 논리를 정의 가능!

**5.1.1 인증 프로세스 중 요청 나타내기**

- Authentication
    - 인증 프로세스의 필수 인터페이스
    - 인증 요청 이벤트를 의미!
    - 애플리케이션에 접근을 요청한 엔티티의 세부 정보를 담음
        - 인증 요청 이벤트 정보는 인증 프로세스 도중과 이후에 이용 가능
    - Principal (주체)
        - 애플리케이션에 접근을 요청하는 사용자
        - 자바 시큐리티 API의 Principal 인터페이스와 같은 개념
            - Authentication 인터페이스는 자바 시큐리티의 Principal 인터페이스를 확장
            - 다른 프레임워크나 애플리케이션과의 호환에 유리 → 유연함
    - 인증 프로세스 완료 여부, 권한의 컬렉션 정보도 추가로 가짐
        - isAuthenticated() - 인증 프로세스가 끝났으면 true
        - getCredentials() - 인증 프로세스에 이용된 암호나 비밀을 반환
        - getAuthorities() - 인증된 요청에 허가된 권한의 컬렉션을 반환

**5.1.2 맞춤형 인증 논리 구현**

- AuthenticationProvider
    - 인증 논리를 처리하는 인터페이스
    - 시스템의 사용자를 찾는 처리를 UserDetailsService에 위임
    - PasswordEncoder로 인증 프로세스에서 암호를 관리
    - Authentication 인터페이스와 강하게 결합
        - authenticate() 메서드는 Authentication 객체를 매개 변수로 받아 Authentication 객체를 반환
            - 인증이 실패하면 AuthenticationException 투척
            - 메서드가 지원되지 않는 인증 객체를 받으면 null을 반환 ⇒ HTTP 필터 수준에서 분리된 여러 Authentication 형식을 사용할 가능성
            - 인증이 완료되면 완전히 인증된 Authentication 인스턴스를 반환
                - isAuthenticated() 메서드는 true
                - 인증된 엔티티의 모든 필수 세부정보가 포함
                - 암호화 같은 민감한 데이터는 제거 ⇒ 민감 데이터 유출 방지
        - supports(Class<?> authentication) 메서드
            - 현재 AuthenticaionProvider가 Authentication 객체로 제공된 형식을 지원하면 true
            - 주의점
                - 이 메서드가 true를 반환해도 authenticate()가 null을 반환해 요청 거부 가능
                    - 인증 유형만 아니라 요청의 세부 정보를 기준으로 인증 요청을 거부할 수 있는 구조
        
        ![IMG_A1913902B354-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/0a1a65e9-d0fb-4067-aba8-93258fa0e279/IMG_A1913902B354-1.jpeg)
        

**5.1.3 맞춤형 인증 논리 적용**

![IMG_0A0CC5F3A81B-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/d1479906-7554-44a1-ac92-880ddc926316/IMG_0A0CC5F3A81B-1.jpeg)

1. AuthenticationProvider 인터페이스를 구현하는 클래스 선언
    1. 스프링에서 관리하는 컨텍스트에 포함되도록 @Component 표시
2. AuthenticationProvider가 어떤 종류의 Authentication 객체를 지원할지 결정
    1. 지원하는 인증 유형을 나타내도록 supports(Class<?> c) 메서드 재정의
        1. 만약 아무것도 맞춤 구성하지 않으면 UsernamePasswordAuthenticationToken 클래스가 형식을 정의 (Authentication 구현체로 사용자 이름과 암호를 이용하는 표준 인증 요청)
    2. authenticate(Authentication a) 메서드 재정의 → 인증 논리 구현
        1. UserDetails를 가져오기 위해 UserDetailsService 구현제 이용
        2. loadUserByUsername()은 사용자가 존재하지 않으면 AuthenticationException 투척
            1. 인증 프로세스가 중단
            2. HTTP 필터는 응답 상태를 HTTP 401 권한 없음으로 설정
        3. 사용자가 발견되면 컨텍스트에서 PasswordEncoder의 matches()로 사용자의 암호 확인
            1. 암호가 일치하지 않으면 AuthenticationException 투척
            2. 암호가 맞으면 요청의 세부 정보를 포함(인증됨 표시)하는 Authentication 객체를 반환
3. AuthenticationProvider 구현체의 인스턴스를 스프링 시큐리티에 등록
    1. 프로젝트의 구성 클래스에서 WebSecurityConfigurerAdapter 클래스의 configure(AuthenticationManagerBuilder auth) 메서드 재정의

### 5.2 SecurityContext 이용

![IMG_365DC20CD876-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/beb00fbd-dce5-49de-85b5-164d65967d06/IMG_365DC20CD876-1.jpeg)

- 인증 프로세스가 끝난 후 인증된 엔티티에 대한 세부 정보가 필요하다면?
    - 보안 컨텍스트
        - Authentication 객체를 저장하는 인스턴스
        - AuthenticationManager가 인증 프로세스를 성공적으로 완료한 후 요청이 유지되는 동안 Authentication 인스턴스를 저장
- 보안 컨텍스트의 작동 방식
- 다양한 스레드 관련 시나리오에서 애플리케이션이 데이터를 관리하는 방법 분석
    - SecurityContext → 주 책임은 Authentication 객체를 저장하는 것
    - SecurityContext는 어떻게 관리될까? → SecurityContextHolder 객체(관리자 역할을 하는 객체)
        - 스프링 시큐리티는 SecurityContext를 관리하는 세 가지 전략 제공
            - MODE_THREADLOCAL
                - 각 스레드가 보안 컨텍스트에 각자의 세부 정보를 가짐
                - 각 요청이 개별 스레드를 가지는 요청 당 스레드 방식의 웹 애플리케이션에서 일반적인 접근법
            - MODE_INHERITABLETHREADLOCAL
                - 비동기 메서드의 경우 보안 컨텍스트를 다음 스레드로 복사하도록 스프링 시큐리티에 지시
                - @Async 메서드를 실행하는 새 스레드가 보안 컨텍스트를 상속
            - MODE_GLOBAL
                - 애플리케이션의 모든 스레드가 같은 보안 컨텍스트 인스턴스를 바라보게 됨

**5.2.1 보안 컨텍스트를 위한 보유 전략 이용**

- `MODE_THREADLOCAL`
    - 스프링 시큐리티가 보안 컨텍스트를 관리하는 기본 전략
    - JDK 인터페이스의 ThreadLocal 이용
        - 애플리케이션의 각 스레드가 자신의 컬렉션에 저장된 데이터만 볼 수 있도록 보장
        - 새 스레드가 생성되면(예: 비동기 메서드 호출) 새 스레드도 자체 보안 컨텍스트를 가짐
            - 상위 스레드의 세부 정보는 새 스레드의 보안 컨텍스트로 복사되지 않음
                
                ![IMG_A9E5B3B9B9F1-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/b66e0c11-48ed-48c2-90f6-6b99ce46dac3/IMG_A9E5B3B9B9F1-1.jpeg)
                
            - 명시적으로 구성할 필요 X
                - 필요할 때마다 getContext() 메서드로 보안 컨텍스트를 요청하기만 하면 됨

**5.2.2 비동기 호출을 위한 보유 전략 이용**

- 요청 당 여러 스레드가 사용될 때
    - 엔드포인트가 비동기가 되면(?) 메서드를 실행하는 스레드와 요청을 수행하는 스레드가 다른 스레드가 됨
- @Async 어노테이션을 활성화 → 메서드가 별도의 스레드에서 실행
    - 컨텍스트에서 정보를 얻을 때 NullPointerException이 투척되는 상황 발생
        - 메서드가 보안 컨텍스트를 상속하지 않는 다른 스레드에서 실행되기 때문
- SecurityContextHolder.setStategyName() 메서드를 호출하거나 spring.security.strategy 시스템 속성을 이용
    - 이 전략을 설정하면 프레임워크는 요청의 원래 스레드에 있는 세부 정보를 비동기 메서드의 새로 생성된 스레드로 복사

![IMG_2F158E59631B-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/1c21463d-5d82-4047-af71-08c3c967d78f/IMG_2F158E59631B-1.jpeg)

**5.2.3 독립형 애플리케이션을 위한 보유 전략 이용**

- 보안 컨텍스트가 애플리케이션의 모든 스레드에 공유되는 전략
- 웹 서버에는 이용되지 않음
    - 백엔드 웹 애플리케이션은 수신하는 요청을 독립적으로 관리
    - 요청별로 보안 컨텍스트를 분리하는 것이 합리적
    - SecurityContextHolder.setStategyName() 메서드 또는 spring.security.strategy 설정을 통해 전략 변경 가능
    - 유의점
        - securityContext는 스레드 안전을 지원하지 X
        - 동시 접근 문제는 개발자가 직접 해결해야 함

![IMG_26472106F378-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/a1abf5f8-fa6c-4986-ba4a-5a398f1895bf/IMG_26472106F378-1.jpeg)

**5.2.4 DelegatingSecurityContextRunnable로 보안 컨텍스트 전달**

- 프레임워크가 모르는 방법으로 코드가 새 스레드를 시작한다면?
    - 이러한 스레드는 프레임워크가 관리해주지 않아 개발자가 관리해야 함 → **자체 관리 스레드**
- SecurityContextHolder 전략으로는 자체 관리 스레드를 위한 해결책을 얻을 수 없음
    - 개발자가 직접 보안 컨텍스트 전파를 해결해야!
    - 해결책 중 한 가지가 별도의 스레드에서 실행하고 싶은 작업을 DelegatingSecurityContexRunnable로 장식하는 것!
- DelegatingSecurityContexRunnable
    - Runnable을 확장
    - 반환 값이 없는 작업 실행 후 이용 가능
    - 반환 값이 있는 작업에는 DelegatingSecurityContexCallable<T>에 해당하는 Callable<T> 대안 이용
    - 두 클래스 모두 Runnable 또는 Callable과 마찬가지로 비동기적으로 실행되는 작업을 나타냄
    - 작업을 실행하는 스레드를 위해 현재 보안 컨텍스트를 복사해 줌!

**5.2.5 DelegatingSecurityContextExecutorService로 보안 컨텍스트 전달**

- 작업에서 보안 컨텍스트 전파를 처리하지 않고 스레드 풀에서 전파를 관리하는 방법
- 작업을 장식하는 대신 특정 유형의 Executor를 이용
    - 작업이 단순한 Callable<T>로 남아있지만, 여전히 스레드가 보안 컨텍스트를 관리
    - 보안 컨텍스트가 전파되는 이유는 DelegatingSecurityContextExecutorServcie 구현체가 ExecutorService를 장식하기 때문

![IMG_ABE49ACFDE0C-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/a7b5c063-0002-4dcb-ba18-7eaf9bd0d524/IMG_ABE49ACFDE0C-1.jpeg)

- 스프링에는 스레드를 생성할 때 보안 컨텍스트를 관리하기 위한 다양한 유틸리티 클래스가 있음
    - DelegatingSecurityContextScheduledExecutorService
        - 예약된 작업을 위한 보안 컨텍스트 전파 구현
    - DelegatingSecurityContextExecutor
        - Executor 인터페이스 구현체
        - 보안 컨텍스트를 해당 풀에 의해 생성된 스레드로 전달하는 기능을 제공

### 5.3 HTTP Basic 인증과 양식 기반 로그인 인증 이해하기

**5.3.1 HTTP Basic 이용 및 구성**

- HTTP Basic
    - 기본 인증 방식
    - 일부 설정을 맞춤 구성할 수 있음
        - 인증 프로세스가 실패할 때를 위한 특정한 논리를 구현
        - 클라이언트로 반환되는 응답의 일부 값을 설정
        - Customizer 형식의 매개 변수를 지정하고 httpBasic() 메서드 호출
            - 인증 방식과 관련한 일부 구성 설정이 가능
            - 예시) 영역(realm) 설정 → 특정 인증 방식을 이용하는 보호 공간
        - Customizer로 인증이 실패했을 때의 응답을 맞춤 구성 가능
            - AuthenticationEntryPoint 구현
                - commence() 메서드는 HttpServletRequest, HttpServletResponse, AuthenticationException을 매개변수로 받음
                
                ![IMG_5501671CD3E8-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/4d6e8d11-30c2-43b5-9d4e-2ef59b922c23/IMG_5501671CD3E8-1.jpeg)
                

**5.3.2 양식 기반 로그인으로 인증 구현**

- 작은 웹 애플리케이션에서는 양식 기반 인증 방식 활용 가능
    - 왜 작은 웹 애플리케이션인가?
        - 보안 컨텍스트를 관리하는데 서버 쪽 세션을 이용하기 때문
        - 수평 확장성이 필요한 대형 애플리케이션의 경우에는 보안 컨텍스트를 서버 쪽 세션에서 관리하는 것은 좋지 않음
- 구성 방법
    - 구성 클래스의 configure(HttpSecurity http) 메서드에서 HttpSecurity 매개 변수의 httpBasic() 대신 formLogin() 메서드 호출
        - 스프링 시큐리티는 로그인 양식과 로그아웃 페이지를 제공
    - UserDetailsService를 등록하지 않으면 기본 제공된 자격 증명으로 로그인
        - 사용자 이름 ‘user’와 애플리케이션의 콘솔에 표시되는 UUID 암호
    - 이전 예제와 동일한 인증 아키텍처에 의존하지만 컨트롤러에서 간단한 JSON 형식의 응답 대신 브라우저가 **웹 페이지로 해석할 수 있는 HTML을 반환**하는 엔드포인트를 원한다는 차이점
        - RestController가 아닌 Controller 어노테이션 이용
    - formLogin() 메서드는  FormLoginConfigurer<HttpSecurity> 형식의 객체를 반환
        - 이를 이용해 맞춤 구성이 가능
            - 예를 들어, defaultSuccessUrl() 메서드 호출이 가능
    - 보다 세부적인 맞춤 구성
        - AuthenticationSuccessHandler
            - 인증이 성공했을 때의 논리를 구현
            - onAuthenticationSuccess() 메서드는 매개변수로 서블릿 요청, 서블릿 응답, 그리고 Authentication 객체를 받는다
        - AuthenticationFailureHandler
            - 인증이 실패했을 때의 논리를 구현
                - 예를 들어 애플리케이션이 요청 식별자를 보내야 하는 경우가 있음
                    - 여러  시스템 사이에서 요청을 역추적하기 위한 고유한 값
                - onAuthenticationFailure() 메서드는 서블릿 요청, 서블릿 응답, 그리고 Authentication 객체를 매개변수로 받는다
