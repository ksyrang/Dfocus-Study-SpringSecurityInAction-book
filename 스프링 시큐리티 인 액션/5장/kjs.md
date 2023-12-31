# 5장 인증 구현

## 이 단원의 내용

- 맞춤형 AuthenticationProvider로 인증 논리 구현
- HTTP Basic 및 양식 기반 로그인 인증 메서드 이용
- SecurityContext 구성 요소의 이해 및 관리

## 시작에 앞서…

- 인증 놀리를 담당하는 것은 AuthenticationProvider 계층이다.
- AuthenticationProvider 계층에서 요청을 허용할지 결정하는 조건과 명령을 발견할 수 있다.
- AuthenticationManager는 HTTP Filter 계층에서 요청을 수신, 이 책임을 AuthenticationProvider에 위임하는 구성 요소다.
    - 요청 엔티티가 인증 않됨 : App이 요청자를 인식 못해 요청을 거부하고  401 상태를 반환한다.
    - 요청 엔티티 인증 : 요청자가 저장돼 있어 App이 이를 권한 부여에 이용한다. 이 정보를 SecurtiyContext 인터페이스 인스턴스에 저장된다.

![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled.png)

## 5.1 AuthenticationProvider의 이해

- 엔터프라이즈 App은 사용자의 ID,Password 기반의 기본 인증 구현이 적합하지 않을 수 있다.
- 사용자가 다양한 시나리오로 신원은 증명할 수 있기에 이에 맞게 기능을 구현해야 한다.
    - 지문, SMS, 특정 표시 등
- 일반적으로 프레임워크는 가장 대중적인 기능은 지원하지만 모든 시나리오를 지원하진 못한다.
- 스프링 시큐리티에서는 AuthenticationProvider 인터페이스로 맞춤형 인증 논리를 정의할 수 있다.

### 5.1.1 인증 프로세스 중 요청 나타내기

- 이 절에서는 시큐리티가 인증 프로세스 도중 요청을 나타내는 방법을 알아볼 것
- AuthenticationProvider를 잘 구현하려면 인증 이벤트를 잘 이해할 것
- Authentication는 인증 프로세스의 필수 인터페이스이다.
    - AuthenticationProvider = 인증 요청 이벤트, App 접근을 요청한 엔티티의 세부 정보를 담는다.
- 인증 요청 이벤트와 관련한 정보는 인증 프로세스 도중과 이후에도 이용 가능하다.
- App에 접근을 요청하는 사용자를 주체(Principal)이라 한다.
- Java 시큐리티API에서의 Principal도 같은 개념이며 스프링 시큐리티는 이를 확장시킨 것이다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%201.png)
    
- Authentication 인터페이스는 계약 주체 뿐만이 아닌 인증 완료 여부, 권한 컬렉션과 같은 정보를 추가로 가진다.
- 스프링 시큐리티 API가 Java 시큐리티 API에서 확장되어 설계됐음은 다른 프레임워크나 App 구현과의 호환성 측면에서 이점이 된다.
- 위의 이점을 통해 다른 인증 방식의 App의 스프링 시큐리티에 더 쉬운 마이그레이션이 가능하다.
- Authentication 인터페이스 기본 구조
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%202.png)
    

### 5.1.2 맞춤형 인증 논리 구현

- 책임과 연관된 스프링 시큐리티 인터페이스를 분석, 이해하고 맞춤형 인증 논리를 직접 구현
- AuthenticationProvider는 인증 논리를 처리한다.
    - 기본적으로 사용자 분석 및 암호 관리를 UserDetailsService와 PasswordEncoder에 위임한다.
- AuthenticationProvider 인터페이스의 기본 구조
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%203.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%204.png)
    
- AuthenticationProvider 인터페이스는 Authentication 인터페이스와 강하게 결합돼 있다.
- authenticate 메서드는 Authentication을 매개변수로 받고 동일한 인터페이스를 반환한다.
- authenticate 메서드 구현에는 아래 3개의 항목으로 요약할 수 있다.
    1. 인증 실패 시 AuthenticationException을 throw해야 한다.
    2. 메서드가 AuthenticationProvider 구현에서 지원되지 않는 인증 객체일 경우 Null을 반환해야한다.
    3. 메서드는 완전히 인증된 객체를 나타내는 Authentication 인스턴스를 반환해야 한다.
        - isAuthenticated()를 true로 반환, 인증된 엔티티의 모든 필수 정보를 포함, 민감한 데이터(password 등)는 인스턴스에서 제거해야 한다.
- supports 메서드는 현재 AuthenticationProvider가 Authentication 객체로 제공된 형식을 지원하면 true를 반환한다.
- 주의사항으로 true를 반환해요 authenticate() 메서드가 null을 반환해 요청을 거부할 수 있고 Support() 메서드를 이용해 AuthenticationProvider에 유요한 Authentication인지를 확인 할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%205.png)
    
- 즉 인증 유형만이 아닌 요청의 세부 정보를 기준으로 인증 요청 거부를 할 수 있게 설계됐다.

### 5.1.3 맞춤형 인증 논리 적용

- 5.1.1과 5.1.2에서 배운 내용을 적용한다.
- 맞춤형 AuthenticationProvider를 구현하는 과정을 단게별로 진행한다.
    1. AuthenticationProvider 구현 클래스를 선언
    2. 새 AuthenticationProvider가 어떤 종류의 Authentication객체를 지원할지 결정
        - 정의하는 AuthenticationProvider가 지원하는 인증 유형을 나타내도록 uspports() 메서드를 재정의 한다.
        - authenticate(Authentication  a) 메서드를 재정의해 인증 논리를 구현
    3. 새 AuthenticationProvider 구현의 인스턴스를 스프링 시큐리티에 등록
- 인증 필터 수준에서 아무것도 맞춤 구성하지 않으면 UsernamePasswordAuthenticationToken 클래스로 정의한다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%206.png)
    
- 구조 시각화와 간단한 구현
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%207.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%208.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%209.png)
    
- 암호가 맞으면 AuthenticationProvider는 요청의 세부 정보를 포함하는 Authentication을 ‘인증됨’으로 표시하고 반환한다.
- 새로 구현한 AuthenticationProvider를 프로젝트의 구성 클래스에 재정의해야 한다.
    - WebSecurityConfigurerAdapter 클래스에서 configure(AuthenticationManagerBuilder auth)메서드를 재정의 한다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2010.png)
    

## 5.2 SecurityContext 이용

- 보안 컨테스트의 작동 방식과 데이터 접근 방법을 알아보기
- 다양한 스레드 관련 시나리오에서 애플리케이션 데이터를 관리하는 방법을 분석
- 이 절을 통해 다양한 상황에 대한 보안 컨텍스트를 구성하는 방법을 숙지할 수 있다.
- 인증 프로세스가 끝난 후 인증된 엔티티에 대한 세부 정보가 필요할 가능성이 크다.
- 인증 프로세스를 성공적으로 완료 후 AuthenticationManager는 요청이 유지되는 동안 Authentication 인스턴스를 저장한다. Authentication 객체를 저장하는 인스턴스를 보안 컨스트럭트라고 한다.
- 보안 컨텍스트 저장 순서
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2011.png)
    
- SecurityContext 인터페이스로 보안 컨테그스트는 기술된다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2012.png)
    
- 스프링 시큐리티는 관리자 역활로 SecurityContext를 관리하며 3가지의 전략을 제공한다.
- 전략들을 정의한 객체를 SecurityContextHolder라고 한다.
    - MODE_THREADLOCAL : 각 스레드가 보안 컨텍스트에 각자의 세부 정보를 저장할 수 있게 해준다. 스레드 방식의 웹 애플리케이션에서는 요청마다 개별 스레드를 가지므로 일반적인 접근법
    - MODE_INHERITABLETHREADLOCAL : MODE_THREADLOCAL과 비슷하지만 비동기 메서드의 경우 보안 컨텍스트를 다음 스레드로 복사하도록 스프링 시큐리티에 지시.
        - @Async 메서드를 실행하는 새 스레드가 보안 컨텍스트를 상속받게 된다.
    - MODE_GLOBAL : 애플리케이션의 모든 스레드가 같은 보안 컨텍스트 인스턴스를 보게 된다.

### 5.2.1 보안 컨텍스트를 위한 보유 전략 이용

- MODE_THREADLOCAL 전략은 스프링 시큐리티가 보안 컨텍스트를 관리하는 기본 전략이다.
- 이 전략은 ThredLocal을 컨텍스트를 이용해 관리하며 JDK에 있는 구현이다.
- ThredLocal은 데이터 컬렉션처럼 작동하지만 애플리케이션의 각 스레드가 컬렉션에 저장된 데이터만 볼 수 있도록 보장한다. 각 요청은 자신의 보안 컨텍스트에 접근하며 스레드는 다른 스레드에 접근 할 수 없다.
- 이는 백엔드 웹 애플리케이션의 일반적인 작동 방식이다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2013.png)
    
- 요청에서 컨텍스트 데이터를 얻는 방법
    - SecurityContextHolder 클래스에서 .getContext() 메서드를 이용해 불러올 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2014.png)
    
    - 스프링은 인증을 매서드 매개 변수에 곧바로 주입할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2015.png)
    

### 5.2.2 비동기 호출을 위한 보유 전략 이용

- 보안 컨텍스트 관리는 기본 전략을 고수하는 것이 더 쉽고 대부분이 기본 전략으로 충분하다.
- MODE_INHERITABLETHREADLOCAL는 각 스레드의 보안 컨텍스트를 격리할 수 있게 해주고 보안 컨텍스트를 더 자연스럽게 이해하기 쉽게 만들어준다. 하지만 모든 상황에 적용 할 수 없다.
- 하나의 요청에 여러 스레드를 사용해야 할 때 복잡하다. 이때 엔드포인트가 비동기가 되면서 메서드를 싱행하는 스레드와 요청을 수행하는 스레드가 다른 스레드가 된다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2016.png)
    
    - @Aysnc 어노테이션을 사용하기 위해서는 @EnableAsync 어노테이션을 지정해야 한다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2017.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2018.png)
    
- 위의 코드 상태에서는 실행시 getName() 메서드에서 NullPointerException이 투척 된다. 이는 이전 보안 컨텍스트를 상속하지 않는 스레드에서 실행되기 때문이다.
- 이를 MODE_INHERITABLETHREADLOCAL에서 SecurityContextHolder.setStrategyname() 메서드나 spring.security.strategy 시스템 속성을 이용하면 해결할 수 있다.
- 위의 메서드들은 원래 스레드의 세부 정보를 새 스레드에 복사한다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2019.png)
    
- 설정 방법
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2020.png)
    

### 5.2.3 독립형 애플리케이션을 위한 보유 전략 이용

- MODE_GLOBAL는 보안 컨텍스트가 모든 스레드에 공유되는 전략으로 일반적인 애플리케이션에는 맞지 않아 웹 서버에서 이용되지 않는다.
- 백엔드 웹 애플리케이션은 수신한 요청을 독립적으로 관리한다. 모든 요청에 대해 하나의 컨텍스트를 이용하기 보단 요청별 보안 컨텍스트를 분리하는 것이 더 합리적이다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2021.png)
    
- 구현 방법
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2022.png)
    
- SecurityContext는 스레드의 안전을 지원하지 않는다! 즉, 이전략에서는 모든 스레드가 SecurityContext에 접근할 수 있으므로 개발자가 동시 접근을 해결해야한다.

### 5.2.4 DelegatingSecurityContextRunnable로 보안 컨텍스트 전달

- 프레임워크는 요청의 스레드에 보안 컨텍스트를 제공하고 접근을 보장하지만 새로 생긴 스레드에 관해서는 별도의 작업을 하지 않는다.
- 따라서 이러한 스레드는 개발자가 관리해줘야 한다.(자체 관리 스레드라고도함)
- 개발자가 이런 스레드를 보안 컨텍스트로 전파하고 싶을 때 DelegatingSecurityContextRunnable로 장식한다.
- Runnable을 확장하며 반환 값이 없는 작업 실행 후 이용할 수 있다.
- 반환 값이 있는 작업에는 DelegatingSecurityContextCallable<T>에 해당하는 Callable<T> 대안을 이용할 수 있다. 두 클래스 모두 비동기적으로 실행되는 작업이며 작업을 실행하는 스레드르 위해 현재 보안 컨텍스트를 복사해 준다.

![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2023.png)

- 기본 사용 예
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2024.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2025.png)
    
- ExecutorService를 이용한 스레드 작업 제출 방법
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2026.png)
    
- 위의 예제에서는 새로 생성된 스레드에 인증이 더는 존재하지 않아 보안 컨텍스트는 비어 있게 된다. 이를 해결하기 위해 DelegatingSecureityContextCallable로 장식해 현재 컨텍스트를 새 스레드에 제공한다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2027.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2028.png)
    

### 5.2.5 DelegatingSecurityContextExcutorService로 보안 컨텍스트 전달

- 프레임워크에 알리지 않고 코드에서 시작한 스레드를 다룰 때는 보안 컨텍스트에서 다음 스레드로의 전파를 관리해야 한다.
- 앞선 절과는 다른 방법으로 새 스레드로의 보안 컨텍스트 전파를 해결할 수 있는데, 작업을 처리하지 않고 스레드 풀에서 전파를 관리하는 것이다.
- 즉 작업을 장식(전파? 스레드에 복사?)하는 대신 특정 유형의 Executor를 이용할 수 있다.
- Service의 구현이 ExcutorService를 장식하면 스레드가 보안 컨텍스트를 관리한다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2029.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2030.png)
    
- 예약된 작업을 위해 보안 컨텍스트 전파를 구현해야 한다면 스프링 시큐리티에 있는 DelegatingSecurityContextScheduledExcutorService라는 데코레이터를 이용하면 된다.
- DelegatingSecurityContextScheduledExcutorService의 작동 메커니즘은 DelegatingSecurityContextExcutorService와 비슷하지만 ScheduledExcutorService를 표시 하므로 예약된 작업을 대상으로 하는 점이 다르다.
- 또한 시큐리티에는 유연성을 높이기 위한 DelegatingSecurityContextExcutor라는 더 추상적 데코테이터 버전이 있다.
    - 이 클래스는 스레드 풀 계층 구조의 가장 추상적인 인터페이스 Excutor를 직접 장식한다.
    - 애플리케이션을 디자인할 때 스레드 풀의 구현을 언어가 제공하는 선택 사항으로 대체할 수 있기를 원하면 이를 선택할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2031.png)
    

## 5.3 HTTP Basic 인증과 양식 기반 로그인 인증 이해하기

- HTTP Basic은 인증이 간단하기에 데모 또는 개념 증명에 좋지만 실제 시나리오에는 적합하지 않을 수 있다.
- HTTP Basic과 관련한 추가 구성을 알아보고 formLogin이라는 새 인증 방식을 소개한다.
- 또한 다양한 아키텍처와 어울리는 인증 방식들과 모범 사례 안티 패턴을 이해할 수 있도록 비교해 볼것이다.

### 5.3.1 HTTP Basic 이용 및 구성

- 이 절에서는 HTTP Basic 인증 방식의 구성에 관해 알아볼 것
- 기본적으로는 HTTP Basic로 충분하지만 더 복잡한 애플리케이션에서는 일부 설정을 맞춤 구성해야 할 수 있다.
    - 예로 인증 실패 시 특정 논리 구현, 클라이언트로 반환되는 응답 설정 등
- Customizer형식의 매개 변수를 지정하고HttpSecurity 인스턴스의 httpBasic() 메서드르 호출할 수도 있다. 예시처럼 인증 방식과 관련된 일부 구성을 설정할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2032.png)
    
- 또한 Customizer로 인증 실패 때의 응답을 맞춤 구성할 수 있다.
    - 클라이언트가 응답에서 특정 항목을 기대하는 경우에 맞춤 구성이 필요하다.
- 인증 실패 떄의 응답을 맞춤 구성하려면 AuthenticationEntryPoint를 구현하면 된다.
    - AuthenticationEntryPoint의 commence() 메서드는 httpServletRequest, httpServletRespoose, AuthenticationException을 받는다.
    - 위의 객체들을 이용해 AuthenticationEntryPoint에서 구현하면 된다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2033.png)
    
- 하지만 AuthenticationEntryPoint 인터페이스는 시큐리티 아키텍처에서 ExceptionTranslationManager라는 구성 요소에서 직접 사용되며 필터 체인에서 투척된 모든 AccessDeniedException 및 AuthenticationException을 처리한다. 즉 자바 예외와 HTTp 응답을 연결하는 다리라고 볼 수 있다.

### 5.3.2 양식 기반 로그인으로 인증 구현

- 웹 애플리케이션을 개발할 때는 사용자 친화적인 로그인 양식을 제공, 인증된 사용자가 로그인 후 웹 페이지를 탐색하고 로그아웃하는 기능을 구현하길 원할 것이다.
- 이 절은 인증 방식을 구성하는 방법을 배운다.
- 양식 기반의 로그인 과정
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2034.png)
    
- 인증 후 이동할 페이지를 위한 컨트롤러를 구현해야한다. 간단한 JSON 형식의 응답 대신 브라우저가 웹 페이지로 해석할 수 있는 HTML을 반환하는 엔드포인트를 원한다.
- 이 것이 스프링 MVC이며  스프링 MVC 흐름의 간소하게 표현해 봤다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2035.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2036.png)
    
- formLogin() 메서드는 FormLoginConigurer〈 httpSecurity〉 형식의 객체를 반환하며 이를 이용해 login 성공 시 url를 설정할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2037.png)
    
- 더 세부적인 맞춤 구성이 필요하면 AuthenticationSuccessHandler 및 AuthenticationFailureHandler 객체를 이용할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2038.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2039.png)
    
- 인증을 실패 했을 때 401 이외의 HTTP 상태 코드나 응답 본문에 추가 정보가 필요할 수 있다.
    - 가장 일반적 사례는 요청 식별자를 보내는 경우가 있다.
        - 요청의 역추적을 위함(고유 값)과 응답 본문에 그 값을 보낼 수 있기 때문이다.
    - 민감 데이터를 외부에 유출하지 않기 위한 응답 소거가 있다.
- 인증 실패 때의 실행할 논리를 구성하려면 AuthenticationFailureHandler 구현을 이용하면 된다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2040.png)
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2041.png)
    
- 이 두 객체를 이용하려면 formLogin() 메서드가 반환한 FormLoginConfigurer 객체의 configure() 메서드에서 객체를 등록해야 한다
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2042.png)
    
- 올바른 사용자 정보로 HTTP Basic 방식으로 /home 경로에 접근하려고 하면 ‘302 임시 이동’이 포함된 응답이 반환 된다.
- 302 코드는 애플리케이션이 리디렉션을 시도하고 있음을 알려주는 것으로 올바른 정보를 제공했어도 이를 고려치 않고 formLogin 메서드의 요청에 따라 사용자 로그인 양식으로 보내려고 시도한다. 이를 HTTP Basic과 양식 기반 로그인 방식 모드를 지원토록 변경 할 수 있다.
    
    ![Untitled](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%20f4bd83b65e454d17bef0e098c39bfa93/Untitled%2043.png)
    

### 요약

- AuthenticationProvider 구성요소를 이용하면맞츰형인증논리를 구현할수 있다.
- 맞춤형 인증 논리를 구현할 때는 책임을 분리하는 것이 좋다. AuthenticationProvider는 사용자 관리는 UserDetailsService에 위임하고 암호 검증 책임은 PasswordEncoder에 위임한다.
- SecurityContext는 인증이 성공한 후 인증된 엔티티에 대한 세부 정보를 유지한다.
- 보안 컨텍스트 관리하는 데는 3가지 전략을 이용할 수 있으며, 선택한 전략에 따라 다른 스레드에서 보안 컨텍스트 세부 정보에 접근하는 방법이 달라진다.
    - MODE_THREADLOCAL, MODE_INHERITABLETHREADLOCAL, MODE_GLOBAL
- 공유 스레드 로컬 전략을 사용할 때는 스프링이 관리하는 스레드에만 전략이 적용된다는 것을 기억하자. 프레임워크는 자신이 관리하지 않는 스레드에는 보안 컨택스트를 복사하지 않는다.
- 스프링 시큐리티는 코드에서 생성했지만 프레임워크가 인식한 스레드를 관리할 수 있는 우수한 유틸리티 클래스를 제공한다.
    - DelegatingSecurityContextRunnable, DelegatingSecurityContextCallable, DelegatingSecurityContextExcutor
- 스프링 시큐리티는 양식 기반 로그인 인증 메서드인 formLogin()으로 로그인 양식과 로그아웃하는 옵션을 자동으로 구성한다. 작은 웹 애플리케이션의 경우 직관적으로 이용할 수 있다.
- formLogin 인증 메서드는 세부적으로 맞춤 구성이 가능하며 HTTP Basic 방식과 함께 이용해 두 인증 유형을 모두 지원할 수도 있다
